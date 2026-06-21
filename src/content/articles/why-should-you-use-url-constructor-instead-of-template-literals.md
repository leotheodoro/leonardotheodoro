---
title: "Why you should use URL Constructor instead of template literals"
date: 2024-07-24
---

Hey everyone! Today, I'm sharing a quick tip that improved the semantics of my code significantly.

Often, whether working in frontend or backend development, we need to construct URLs with parameters, right?

I used to write the URLs of my requests like this:
```javascript
const url = `http://localhost:3000/endpoint?param1=${var1}&param2=${var2}&param3=${var3}`
```

We agree that this URL is hard to read and maintain; we always need to identify which parts are parameters, which are variables, and which are just Javascript syntax.

To address this semantic issue, I discovered the **URL Constructor**, which accomplishes the same task but in more efficient and elegant way.

Now, we can rewrite the same code like this:
```javascript
const url = new URL('http://localhost:3000/endpoint')

url.searchParams.set('param1', var1)
url.searchParams.set('param2', var2)
url.searchParams.set('param3', var3)
```

The code clearly indicates what it's doing. In the first line, we create the base URL and in the subsequent lines, we add the necessary search parameters.

Done. Now, the variable `url` contains the same search parameters as before, but now we are using the `URL` class, making the code much simpler and easier to maintain.

What about you? Have you used the `URL` class before? Maybe for another purpose? Feel free to share your experiences with me.
