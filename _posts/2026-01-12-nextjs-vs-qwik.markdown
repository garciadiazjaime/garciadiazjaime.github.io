---
layout: post
title: "Moving from Nextjs to Qwik"
date: 2026-01-12 10:00:00 -0500
categories: nextjs qwik javascript-frameworks lighthouse
---

![Moving from Nextjs to Qwik](/assets/2026-01-12-nextjs-vs-qwik/banner.png)

Whenever I’m working on a web application, Next.js is my go-to framework. I like it and I’m used to it, but I’ve been hearing a lot about Qwik and decided to give it a shot.

In my previous post, [Exploring the Art Institute API and Gemini](https://www.garciadiazjaime.com/posts/artic-gemini-quiz), I built the app using Next.js. Since it’s a new project, I thought it would be a good exercise to redo it using Qwik instead.

You can still access the previous version, and to make it a bit more interesting, I’m using Lighthouse to compare the two versions:

- [With Qwik](https://artic.mintitmedia.com/)
- [With Next.js](https://artic-old.mintitmedia.com/)

> Both codebases render a static site, which is generated daily using a cron job and a deploy hook.

## Network

Let’s start by looking at the Network tab:

> Left is Next.js and right is Qwik

![Network tab](/assets/2026-01-12-nextjs-vs-qwik/network-tab.png)

This is very interesting because, in summary, Qwik sends fewer bytes to the browser, and fewer bytes translate into faster rendering as well, even though both apps show basically the same UI.

|                  | Next.js | Qwik   |
| ---------------- | ------- | ------ |
| Transferred      | 285 kB  | 118 kB |
| Resources        | 649 kB  | 172 kB |
| DomContentLoaded | 92 ms   | 96 ms  |
| Load             | 368 ms  | 124 ms |

- **kB transferred** is the data sent from the server (CDN) to the browser, which is zipped.
- **kB resources** is the total size once unzipped.

As we can see, the Qwik project sends less data. **This is a big win for Qwik.**

- **DomContentLoaded** is the event fired when the HTML is parsed and the DOM is built. At this point, JavaScript is executed and images and stylesheets start loading.
- **Load** is the event fired when everything is ready. The browser doesn’t need to download anything else (images, styles, etc.), and the page is fully rendered.

Notice that while **DomContentLoaded** happens at almost the same time, **Load** happens much faster for Qwik. This is just how the framework works. In the case of Next.js, since the app is larger, it takes a bit longer to completely render the page.

> Note: I checked the “Disable cache” checkbox to always download files from the origin (CDN).

Also keep in mind that these values change on every load. Things like browser cache, internet connectivity, and server response times are different every time. That’s why, when testing these values, a load test is a better approach. Loading the page N times gives you a better average. A single run can be influenced by chance. Still, these numbers are a great benchmark for any website.

## Lighthouse

### Performance

![Lighthouse performance results](/assets/2026-01-12-nextjs-vs-qwik/lighthouse-performance-results.png)

Right from the start, Qwik gets a perfect score of 100 across the board, while Next.js gets a slightly lower score. This is interesting because it’s not about the application code, but about the framework itself.

Sure, a Next.js application can also reach 100, but that usually requires extra work, which isn’t necessary when using Qwik.

### Metrics

![Lighthouse metrics results](/assets/2026-01-12-nextjs-vs-qwik/lighthouse-metrics-results.png)

Based on the results from the **Network tab**, it makes sense that these values are lower for Qwik. In simple terms: same UI, fewer bytes, faster rendering.

### Diagnostics

![Lighthouse diagnostics results](/assets/2026-01-12-nextjs-vs-qwik/lighthouse-diagnostics-results.png)

Here we can see Lighthouse pointing out that Next.js has some unused JavaScript. This doesn’t happen with Qwik, confirming that Qwik generates a leaner build.

## Coding

This is my first time using Qwik, and I didn’t want to use AI agents to convert the code. I spent some time reading the [Qwik documentation](https://qwik.dev/docs/), and that was enough to build this simple app. Here are some findings:

### Components

In Next.js, a function is a component, with no extra syntax needed:

```javascript
export default function Home() {
  return <div>Home Page Component</div>;
}
```

With Qwik, you need to use the `component$` function:

```javascript
export default component$(() => {
  return <div>Home Page Component</div>;
});
```

Not a big deal. The syntax changes a bit, but it’s easy enough to follow.

### State variables and clicks

In Next.js, we use `useState` along with `"use client";` because state lives on the client. Similarly, `onClick` handlers only run on the client.

```javascript
"use client";

import { useState } from "react";

export default function Home() {
  const [greeting, setGreeting] = useState("hello world");

  return <div onClick={() => setGreeting("hello from click")}>{greeting}</div>;
}
```

Qwik uses `useSignal` for "signal variables" and `onClick$` for click handlers:

```javascript
import { component$, useSignal } from "@builder.io/qwik";

export default component$(() => {
  const greetingSignal = useSignal("hello world");

  return (
    <div onClick$={() => (greetingSignal.value = "hello from click")}>
      {greetingSignal}
    </div>
  );
});
```

Assigning a new value to `greetingSignal.value = ...` is equivalent to calling `setGreeting(...)`, and it triggers a re-render.

### Fetching data

In both cases, data is fetched on the server. Remember, the page is generated daily, meaning the server generates the HTML, which is then cached in the CDN and served to users. Once per day, the server fetches a JSON file.

**Next.js:**

```javascript
export default async function Home() {
  const quiz = await fetch(`https://domain.com/name.json`).then((res) =>
    res.json()
  );

  return <Quiz quiz={quiz} />;
}
```

Since there’s no `"use client";` directive, this code runs only on the server.

**Qwik:**

```javascript
import { routeLoader$ } from "@builder.io/qwik-city";

export const useQuiz = routeLoader$(async () => {
  const response = await fetch(`https://domain.com/name.json`).then((res) =>
    res.json()
  );
  return response;
});

export default component$(() => {
  const quiz = useQuiz();

  return <Quiz quiz={quiz} />;
});
```

With Qwik, `routeLoader$` defines server-side logic, and the component consumes it as a hook.

## Summary

**Qwik is clearly a leaner framework than Next.js**, and the syntax differences are minimal. Both frameworks share the same core concepts: components, state/signals, data fetching, and more.

One big advantage of Next.js is the size of its community.

At the time of writing:

|                      | Next.js      | Qwik          |
| -------------------- | ------------ | ------------- |
| GitHub stars         | 137k         | 21.9k         |
| GitHub watchers      | 1.5k         | 135           |
| GitHub pull requests | 1.2k         | 43            |
| NPM weekly downloads | 15.5 million | 16.3 thousand |

- [Next.js GitHub repository](https://github.com/vercel/next.js)
- [Next.js NPM page](https://www.npmjs.com/package/next)
- [Qwik GitHub repository](https://github.com/QwikDev/qwik)
- [Qwik NPM page](https://www.npmjs.com/package/@builder.io/qwik)

It’s easy to see that the Next.js community is much larger, which means better ecosystem support, more documentation, and more tooling.

For my personal projects, I’ll definitely keep using Qwik to get more familiar with it. In a professional environment, I’ll stick with Next.js. Just because a framework does something better doesn’t justify the cost of refactoring existing code. Even with AI agents helping, refactoring still involves testing, monitoring, and maintenance. Shaving off a fraction of a second usually isn’t worth the investment—unless you’re in the business of speed, like e-commerce, streaming, or other high-traffic services. Even then, there are often alternatives besides switching frameworks.

Finally, now that a lot of code is written by AI agents, I don’t think the framework choice matters. What really matters is keeping things like reusability, scalability, performance, and design patterns in mind.

It’s not about the framework, it’s about the solution.

That’s it—happy coding.
