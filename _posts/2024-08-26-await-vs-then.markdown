---
layout: post
title: "Try/Catch vs .then().catch(): Which Error Handling Approach is Better?"
date: 2024-08-26 08:00:00 -0500
categories: javascript reactjs web-development best-practices
---

## Handling Promises

Both `.then` and `await` help manage the resolution of a promise.

For example, when working with a helper method that makes an HTTP request, we can use `.then` to structure the code like this:

```js
function getList() {
  const url = "/.netlify/functions/chicagomusiccompass";
  const results = fetch(url).then((response) => response.json());

  return results;
}
```

Alternatively, we can use `await` to structure the code like this:

```js
async function getList() {
  const url = "/.netlify/functions/chicagomusiccompass";
  const response = await fetch(url);
  const results = await response.json();

  return results;
}
```

In both cases, the output remains the same:

```js
[
    {
        "name": "name one"
    },
    {
        "name": "name two"
    },
    ...
]
```

**Note**: Both return a promise, so whoever calls the method will need to wait for the promise to be resolved using either `.then()` or `await` 🤓.

- `.then()`

```js
getList().then((list) => setList(list));
```

- `await`

```js
const list = await getList();
setList(list);
```

## Handling Errors

The real difference comes when handling errors. Up to this point, the implementation looks similar. But what happens if an error is thrown?

With the `.then` approach, you can handle it like this:

```js
function getList() {
  const url = "/invalid-url";

  const results = fetch(url)
    .then((response) => response.json())
    .catch(() => []);

  return results;
}
```

With `await`, the error handling would look like this:

```js
async function getList() {
  const url = "/invalid-url";

  try {
    const response = await fetch(url);
    const results = await response.json();

    return results;
  } catch (error) {
    return [];
  }
}
```

Once again, both approaches produce the same result:

```js
[];
```

It's worth noting that the exception is actually triggered by `.json()`. The `fetch()` function always resolves its promise, even for HTTP errors, and communicates issues via the status code. In this case, the `/invalid-url` response body happens to be invalid JSON, which is why `.json()` throws an exception.

Also, notice how a default value of `[]` is provided in both cases. This is recommended because it allows the consumer to avoid handling exceptions and simply respond to the **returned value**.

## Conclusion

It doesn’t really matter which approach you choose, both produce the same output, and the syntax differences are minimal. As with many other things in development, the choice often comes down to preference.

In the teams I’ve worked with, there’s usually a preferred style, and for consistency, everyone tries to stick with it. However, during Pull Requests reviews, there’s rarely any strict policy on whether to use `try/catch` or `.then().catch()`.

At some point, tools like ESLint or Prettier might enforce a rule or even convert code to a consistent style. _AI agents_ can already do this, but since there’s no functional difference, it’s more about aligning with team consensus. So next time you need to resolve a promise, check if your team has a preference, then go with the flow.
