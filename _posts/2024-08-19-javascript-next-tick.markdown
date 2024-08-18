---
layout: post
title: "Utilizing `setTimeout` to Maintain Operation Sequence"
date: 2024-08-18 08:00:00 -0500
categories: javascript reactjs nextjs web-development
---

## Using an iframe

In a project I'm currently working on, there's a requirement to:

> Display an external website within an iframe.

This approach is necessary because we are two different teams, each managing our own independent codebase.

Embedding a `child app` is straightforward—this is exactly what iframes are designed to do:

```ts
export default function () {
  return (
    <div>
      <div>Parent app</div>
      <iframe src="/javascript-next-tick/child-app.html" />
    </div>
  );
}
```

Everything works seamlessly, and from the user's perspective, the UX remains consistent. They don't need to know—or care—that there are two separate apps running within the webview.

## Listens for clicks within the iframe

**Next Requirement:**

> The `parent-app` should listen for clicks within the iframe. When a click occurs, it should store a "unique id" in local storage.

This is a fairly straightforward requirement. While React and TypeScript may add some complexity, the change is still simple: it’s about attaching an onclick event to the iframe:

```ts
"use client";

import { useRef, useEffect } from "react";

export default function () {
  const ref = useRef<HTMLIFrameElement | null>(null);

  useEffect(() => {
    if (!ref.current?.contentWindow) {
      return;
    }

    ref.current.contentWindow.document.body.onclick = function () {
      console.log("iframe clicked");
      localStorage.setItem("id", "unique-id");
    };
  }, []);

  return (
    <div>
      <div>Parent app</div>
      <iframe ref={ref} src="/javascript-next-tick/child-app.html" />
    </div>
  );
}
```

This implementation ensures that whenever a user clicks anywhere within the iframe, a "unique id" is saved to local storage.

## Interaction Between Parent and Child Applications

**Next and Final Requirement:**

> The `child-app` should be able to call a method from the `parent app`, which should log the previously saved "unique id."

While this may seem challenging, one easy way to establish communication between the parent and child (assuming they are hosted on the same domain) is by using the `window` object, as shown below:

```js
"use client";

import { useRef, useEffect } from "react";

declare global {
  interface Window {
    parentMethod: () => void;
  }
}

export default function () {
  const ref = useRef<HTMLIFrameElement | null>(null);

  useEffect(() => {
    if (!ref.current?.contentWindow) {
      return;
    }

    ref.current.contentWindow.document.body.onclick = function () {
      console.log("2. parent iframe click listener");
      localStorage.setItem("id", "unique-id");
    };

    window.parentMethod = function () {
      const uniqueID = localStorage.getItem("unique-id");
      console.log(`1. parentMethod called, uniqueID: ${uniqueID}`);
    };
  }, []);

  return (
    <div>
      <div>Parent app</div>
      <iframe
        ref={ref}
        src="/javascript-next-tick/child-app.html"
        style={{ width: "100%", minHeight: 400 }}
      />
    </div>
  );
}

```

Now, assuming the `child-app` contains a clickable element, the iframe can invoke the injected method in the following way:

```js
window.top.parentMethod();
```

This should ideally print:

```shell
unique-id: unique-id-value
```

**However, there’s a catch:** the call to `parentMethod();` is executed first because the clickable element in the iframe triggers it. Since the click listener in the `parent app` hasn't been executed yet, nothing is stored in `localStorage`. Consequently, the log output appears as follows:

```shell
1. parentMethod called, uniqueID: null
```

This indicates that the expected value, `unique-id-value`, is missing.

## Using `setTimeout` with a Delay of 0 (Next Tick)

`nextTick` is a concept primarily used in back end, that sort of translates to `setTimeout(() => {}, 0)`. It allows you to instruct the browser to execute a specific line of code once it has completed its current operations.

```js
"use client";

import { useRef, useEffect } from "react";

declare global {
  interface Window {
    parentMethod: () => void;
  }
}

export default function () {
  const ref = useRef<HTMLIFrameElement | null>(null);

  useEffect(() => {
    localStorage.removeItem("id");

    if (!ref.current?.contentWindow) {
      return;
    }

    ref.current.contentWindow.document.body.onclick = function () {
      console.log("2. iframe click triggered, storage set");
      localStorage.setItem("id", "unique-id");
    };

    window.parentMethod = function () {
      const uniqueID = localStorage.getItem("id");
      console.log(`1. parentMethod called, uniqueID: ${uniqueID}`);

      setTimeout(() => {
        console.log(`3. delay done, uniqueID: ${localStorage.getItem("id")}`);
      }, 0);
    };
  }, []);

  return (
    <div>
      <div>Parent app</div>
      <iframe
        ref={ref}
        src="/javascript-next-tick/child-app.html"
        style={{ width: "100%", minHeight: 400 }}
      />
    </div>
  );
}
```

With this change, clicks on the clickable element within the iframe will invoke `parentMethod`, but execution will be deferred until the "next tick." This allows the iframe's click listener to be triggered in time, setting the value for `unique-id`. Only after this will the execution return to `parentMethod` to retrieve and log the value from local storage.

Now, the log should display:

```shell
3. delay done, uniqueID: unique-id
```

## Diagram of how setTimeout helps

![Next Tick: setTimeout sequence of operations](/assets/javascript-next-tick/settimeout-sequence-operations.jpeg)

## Conclusion

The challenge is that there are **two click listeners**: one in the `child` (iframe) and one in the `parent`. The child’s listener runs first but depends on the parent’s processing, which hasn’t completed yet. Using `setTimeout(0)` helps delay the child’s execution just long enough for the parent’s logic to finish.

> setTimeout(0) solves an order of operations issue.

JavaScript is fun, but understanding the event loop is key. Sometimes, you need to wait for the "next tick" to ensure the correct sequence of operations. While `setTimeout 0` can help, it depends on execution timing. For more robust solutions, consider observability but that’s a topic for another day.

### Links

- [Demo](https://demo.garciadiazjaime.com/javascript-next-tick)
- [Codebase](https://github.com/garciadiazjaime/demo-reactjs/tree/main/app/javascript-next-tick)
