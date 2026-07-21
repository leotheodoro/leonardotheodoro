---
title: "The Thundering Herd Problem (and How to Actually Fix It)"
date: 2026-07-21
---

If you've ever watched your database CPU spike to 100% right after a popular cache key expired, you've already met the **thundering herd problem**. You just didn't know its name yet.

## What is the thundering herd problem?

It happens when a large number of processes or requests are woken up at the same time to compete for the same limited resource, and only one (or a few) of them actually needed to do the work.

Picture a stadium after the final whistle. Everyone stands up at once and rushes toward the same exit door, even though there are three other doors nobody's using. The door isn't broken, it's just completely overwhelmed by everyone arriving at the exact same moment.

That's a thundering herd. Not too many requests, just too many requests hitting the same door at the same time.

## Where it shows up in real systems

There are a few classic triggers:

- **Cache expiration**: a hot cache key (say, the homepage data) expires, and the next 10,000 requests all miss the cache simultaneously and slam the database trying to rebuild it.
- **Connection pool wakeups**: multiple worker processes are all blocked waiting on the same event (a new connection, a new job in the queue), and the event wakes all of them, even though only one can grab the work.
- **Retry storms**: a service goes down, clients start retrying, it comes back up, and every client retries at the exact same instant, taking it back down again.
- **Cron jobs at round numbers**: everyone's cron job is set to run at `00:00`, so a burst of unrelated jobs hits your infra at the same second.

The common thread: a single triggering event, and a crowd of consumers reacting to it in lockstep.

## Problem #1: cache stampede

This is the most common version you'll run into as a backend engineer. Let's simulate it.

```javascript
async function getHomepageData() {
  const cached = await cache.get('homepage');
  if (cached) return cached;

  // cache miss: go rebuild it
  const data = await db.query('SELECT * FROM expensive_aggregation');
  await cache.set('homepage', data, { ttl: 60 });
  return data;
}
```

This looks fine in isolation. It falls apart under load. When the key expires, every concurrent request sees `cached === null` at the same time and all of them run the expensive query. If that query takes 500ms and you get 5,000 requests per second, you just sent 2,500 identical queries to your database in half a second.

### Solution: request coalescing (a.k.a. singleflight)

The fix is to make sure that when multiple requests want the same missing value, only one of them actually does the work, and the rest wait for that result.

```javascript
const inFlight = new Map();

async function getHomepageData() {
  const cached = await cache.get('homepage');
  if (cached) return cached;

  if (inFlight.has('homepage')) {
    return inFlight.get('homepage'); // piggyback on the ongoing request
  }

  const promise = (async () => {
    const data = await db.query('SELECT * FROM expensive_aggregation');
    await cache.set('homepage', data, { ttl: 60 });
    inFlight.delete('homepage');
    return data;
  })();

  inFlight.set('homepage', promise);
  return promise;
}
```

Now 2,500 concurrent requests become 1 query and 2,499 requests waiting on the same promise. This pattern has a name in Go's ecosystem: **singleflight**. Most languages have a library for it, but as you can see, it's simple enough to write by hand.

### Solution: probabilistic early expiration

Coalescing solves the stampede at the moment of expiration, but there's a subtler fix that avoids the cliff entirely: instead of a hard TTL, let requests probabilistically decide to refresh the cache *before* it expires, with the odds increasing as the expiration gets closer.

```javascript
function shouldRefreshEarly(cachedAt, ttl, beta = 1) {
  const delta = Date.now() - cachedAt;
  const xfetch = delta * beta * Math.log(Math.random());
  return xfetch >= ttl;
}
```

This is known as the **XFetch** algorithm. In practice, it spreads the recomputation over time instead of concentrating it all at the exact expiration instant, so no single request ever gets hit with the full cost of a cold cache.

## Problem #2: the wakeup stampede

The second flavor shows up at a lower level, when many workers are blocked on the same event.

Imagine ten worker processes all doing a blocking wait on a single job queue. A new job arrives, the underlying OS primitive wakes up all ten workers, they all rush to grab the job, and nine of them immediately go back to sleep having wasted a context switch for nothing.

This is the same shape as the stadium exit: one event, many consumers, only one who actually needed to act.

### Solution: exclusive wake, not broadcast wake

The fix here usually lives at the level of the primitive you're using, not your application code, but it's worth knowing what to look for. Some patterns:

- Use a **mutex around a condition variable** so only one waiter is released per signal (`pthread_cond_signal` instead of `pthread_cond_broadcast`).
- If you're queueing jobs, use a queue implementation that supports **exclusive consumers** (a single worker atomically claims a job, like `BRPOPLPUSH` in Redis) rather than a fan-out notification that every worker has to race to react to.
- For load balancer health checks or leader-election style wakeups, add a small random delay (**jitter**) before acting, so that even if everyone got woken up, they don't all act in the same millisecond.

## Problem #3: retry storms

This one's less about a shared cache and more about client behavior after an outage.

```javascript
async function fetchWithRetry(url, attempt = 1) {
  try {
    return await fetch(url);
  } catch (err) {
    if (attempt > 5) throw err;
    await sleep(1000 * attempt); // fixed backoff, same for everyone
    return fetchWithRetry(url, attempt + 1);
  }
}
```

If a thousand clients all started failing at the same second, they'll all retry at the same second, one second later, then two seconds later, and so on. The backoff grows, but the herd stays perfectly synchronized.

### Solution: jitter

Add randomness to the backoff itself, not just a fixed delay.

```javascript
async function fetchWithRetry(url, attempt = 1) {
  try {
    return await fetch(url);
  } catch (err) {
    if (attempt > 5) throw err;
    const base = 1000 * attempt;
    const jitter = Math.random() * base;
    await sleep(base + jitter);
    return fetchWithRetry(url, attempt + 1);
  }
}
```

This is exactly the strategy AWS describes in their well-known "Exponential Backoff and Jitter" post. The base delay still grows, but the randomness breaks the synchronization, so instead of one giant wave hitting the service, you get a smoothed-out trickle.

## Putting it together

None of these fixes are exotic. They all boil down to the same underlying idea: **decouple the trigger from the reaction**, so a single event doesn't cause every consumer to act at the exact same instant.

| Scenario | Symptom | Fix |
|---|---|---|
| Cache expiration | DB spike on key miss | Request coalescing / singleflight |
| Cache expiration | Predictable, recurring spikes | Probabilistic early expiration |
| Worker wakeups | Wasted context switches | Exclusive wake / atomic job claim |
| Retry storms | Repeated synchronized retries | Backoff with jitter |

Short version: if a lot of things are reacting to one thing at the same time, add either **coordination** (only one of them should actually do the work) or **randomness** (spread the reaction out over time). Almost every thundering herd fix is one of those two ideas wearing a different costume.
