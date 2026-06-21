---
title: "What Are Async Generators and What Are They Used For?"
date: 2025-02-19
---

As a developer, you've probably dealt with data streams that aren’t immediately available – think about data coming from an API, chunked downloads, or even reading large files. In these scenarios, it’s crucial to have an elegant and efficient approach to processing data as it arrives. This is where **async iterators** come in. But before diving into this concept, let’s first understand how synchronous iterators work.

## What Are Iterators?
An **iterator** is an object that defines a sequence and knows how to access its elements one at a time through a method called `next()`. In JavaScript, to make an object iterable – meaning it can be traversed using a `for..of` loop – we define a special method called `Symbol.iterator`.

Imagine we have a `range` object that represents a range of numbers:
```javascript
const range = {
    from: 1,
    to: 5,

    [Symbol.iterator]() {
        return {
            current: this.from,
            last: this.to,
            next() {
                if (this.current <= this.last) {
                    return { done: false, value: this.current++ };
                } else {
                    return { done: true };
                }
            }
        };
    }
};

for (let value of range) {
    console.log(value); // Prints: 1, 2, 3, 4, 5
}
```
In this example, each call to `next()` returns an object with the structure `{ value, done }`. As long as `done` is `false`, the loop continues, allowing `for..of` to iterate over all values in the range.

## Limitations of Iterators
While synchronous iterators work well for sequences of immediately available data, they fail when values are only known or available asynchronously – for instance, when each value depends on a network request or a delay (`setTimeout`).

Imagine you need to iterate over a sequence that depends on an operation that takes time to complete (such as an API call). Using a synchronous iterator won’t work in this case because the `next()` method cannot wait for a Promise to resolve or a delay to pass.

## Generators: Simplifying Iterator Creation
Manually implementing an iterator can be verbose. **Generators** simplify this process. They are special functions that can "pause" execution using the `yield` keyword, returning values on demand and maintaining internal state between calls.

A generator is defined using `function*` and can be used to create iterators concisely:
```javascript
function* generateSequence(start, end) {
    for (let i = start; i <= end; i++) {
        yield i;
    }
}

for (let value of generateSequence(1, 5)) {
    console.log(value); // Prints: 1, 2, 3, 4, 5
}
```

## Introducing Async Iterators
**Async iterators** were created specifically to handle scenarios where values are obtained asynchronously. They work similarly to synchronous iterators but with some key differences:

1. **Iteration method:** Instead of implementing `Symbol.iterator`, we use `Symbol.asyncIterator`.
2. `**next()**`** method:** In an async iterator, `next()` returns a Promise that resolves to an object `{ value, done }`.
3. **Iteration loop:** To consume values, we use `for await...of`, which waits for each Promise to resolve before proceeding.

Let’s refactor the `range` example so that values are returned with a 1-second delay:
```javascript
const asyncRange = {
    from: 1,
    to: 5,

    [Symbol.asyncIterator]() {
        return {
            current: this.from,
            last: this.to,
            async next() {
                // Simulating a delay (as if waiting for a network response)
                await new Promise(resolve => setTimeout(resolve, 1000));
                
                if (this.current <= this.last) {
                    return { done: false, value: this.current++ };
                } else {
                    return { done: true };
                }
            }
        };
    }
};

(async () => {
    for await (let value of asyncRange) {
        console.log(value); // Prints: 1, 2, 3, 4, 5 (with a 1-second interval)
    }
})();
```
In this code, each iteration waits for the delay to resolve before proceeding, allowing data to be processed as it arrives.

## Why Use Async Iterators?
Now that we understand the difference between synchronous iterators and async iterators, let’s explore the advantages of using async iterators in everyday coding:

### 1. On-Demand Processing
In many cases, data arrives in fragments. Instead of waiting for all data to be available, an async iterator allows processing each value as soon as it arrives. This is especially useful for:
- **Paginated requests:** Processing API results page by page instead of loading everything at once.
- **Data streams:** Such as downloading or uploading files where data is received in chunks.

### 2. Integration with Asynchronous APIs
Many modern APIs (such as Fetch API or Streams API) handle data asynchronously. Async iterators allow integrating these data flows naturally and consistently, avoiding "hacks" or excessive code coupling.

### 3. Performance and Efficiency Improvements
By processing data on demand, we avoid loading large volumes of data into memory all at once. This can lead to significant performance improvements, especially when dealing with large data streams.

## Practical Example: Paginating Data with Async Iterators
A classic example of using async iterators is paginating data. Suppose we need to fetch commits from a GitHub repository, where each request returns a page with 30 commits and provides a link to the next page.

Using an async iterator, we can create a `fetchCommits(repo)` function that handles all pagination logic and allows us to iterate over the commits seamlessly:
```javascript
async function* fetchCommits(repo) {
    let url = `https://api.github.com/repos/${repo}/commits`;

    while (url) {
        const response = await fetch(url, {
            headers: { 'User-Agent': 'Our App' }
        });
        const commits = await response.json();

        // Extracts the link to the next page, if available
        let nextPageMatch = response.headers.get('Link')?.match(/<([^>]+)>;\s*rel="next"/);
        url = nextPageMatch ? nextPageMatch[1] : null;

        for (let commit of commits) {
            yield commit;
        }
    }
}

(async () => {
    for await (let commit of fetchCommits("username/repository")) {
        // Process each commit as it arrives
        console.log(commit);
    }
})();
```
This pattern allows you to process commits continuously and efficiently, without worrying about pagination logic in each call.

## Final Considerations
**Async iterators** are a powerful tool for working with asynchronously arriving data. By allowing on-demand iteration, they simplify integration with modern APIs, make code cleaner and more expressive, and help avoid performance issues when handling large data streams.

In summary, opting for async iterators provides:
- **Control:** Processing data as it becomes available.
- **Readability:** Using `for await...of` for more intuitive code.
- **Efficiency:** Avoiding unnecessary large memory loads.
- **Integration:** Easy handling of asynchronous APIs and real-time data streams.

See you next time!