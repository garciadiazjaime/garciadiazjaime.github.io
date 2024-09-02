---
layout: post
title: "JavaScript: A Short Story of Decoupling"
date: 2024-08-12 08:00:00 -0500
categories: javascript software-development best-practices
---

![JavaScript: A Short Story of Decoupling](/assets/javascript-decoupling/banner.png)

I was working on a piece of code and submitted a Pull Request for it. During the review, a colleague left this comment:

> Why not decouple this file?

It was a great idea. I'm always in favor of decoupling because it makes things easier to scale and aligns with the Single Responsibility Principle.

The requirement was:

> If condition `A` is `true`, then save this `payload` into a file.

_Note_: For now, the `filename` doesn't matter.

Easy, right?

## First Version

Well, my first iteration looked like this:

File: `controller`

```js
import savePayload from "./utils";

if (CONDITION_A) {
  savePayload(payload);
}
```

File: `utils`

```js
const DIRECTORY = "some_directory";

export const savePayload = (payload) => {
  const filename = "unique_name";
  const filePath = `${DIRECTORY}/${filename}`;
  ...
}
```

The approach works; it's clean, readable, and testable. But what if someone else wants to save to a different directory?

The approach won't work 😔

This is where decoupling comes in handy. If the `DIRECTORY` is decoupled from `utils`, our code becomes scalable. The best part is that the change is quite simple.

## Second Version

File: `controller`

```js
import savePayload from "./utils";
const DIRECTORY = "some_directory"; // <-- small big change

if (CONDITION_A) {
  savePayload(DIRECTORY, payload);
}
```

File: `utils`

```js
export const savePayload = (directory, payload) => {
  const filename = "unique_name";
  const filePath = `${directory}/${filename}`;
  ...
}
```

Sweet! Now whoever calls `savePayload` needs to pass the directory 🙃.

At least now, this method works for any desired directory. Essentially, the coupling has been moved to the consumer. 😄

## Third Version

The last change introduced was moving the directory to a constants file.

File: `constants`

```js
export const DIRECTORY = "some_directory";
```

File: `controller`

```js
import savePayload from "./utils";
import { DIRECTORY } from "./constants";

if (CONDITION_A) {
  savePayload(DIRECTORY, payload);
}
```

File: `utils`

```js
export const savePayload = (directory, payload) => {
  const filename = "unique_name";
  const filePath = `${directory}/${filename}`;
  ...
}
```

## Conclusion

As with any approach, there are always pros and cons. In our case, we knew `savePayload` would be needed for other cases, so it made sense to proceed with decoupling.

Decoupling is definitely beneficial, but be cautious with early optimizations. Most of the time, it's better to cross that bridge when you get there :)
