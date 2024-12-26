---
layout: post
title: "How Many Resources Does a Click Consume? React vs. Vanilla"
date: 2024-10-21 10:00:00 -0500
categories: javascript programming react webperf
---

![How Many Resources Does a Click Consume? React vs. Vanilla](/assets/resources-click-react-vanilla/banner.png)

React, like any other JavaScript framework, handles a lot of things behind the scenes that we often don’t even think about.

And that’s okay—our job as developers is to solve problems, and the simpler the implementation, the better. You don’t always need to understand every detail of what a framework does for you.

JavaScript is an interesting language; it’s the king of the browser, and browsers are still used heavily, so I don’t see it disappearing anytime soon.

In fact, many native apps (iOS, Android, Smart TVs) run hybrid solutions using both native and web technologies.

In this post, I want to test a simple counter in React versus its Vanilla JavaScript version.

## Measuring Performance with Chrome DevTools

First, let's talk about a useful tab that Chrome offers called `Performance`. This tab includes a recording feature that measures a web application's performance.

In this post, I'm going to focus on three key metrics: `JS Heap`, `Nodes`, and `Listeners`.

> **JS Heap**: The heap is a region of memory in JavaScript where objects, arrays, and functions are stored. Unlike the stack, which holds primitive values (numbers, strings, booleans) and function calls, the heap manages dynamic memory allocation.

> **DOM Nodes**: A DOM node is an individual element, attribute, or text within a web page's HTML, represented in the Document Object Model (DOM).

> **Event Listeners**: In JavaScript, event listeners are functions that wait for specific events (e.g., clicks, key presses, mouse movements) on HTML elements. When the event occurs, the listener triggers, executing code in response.

## Demo: Building a Basic React Counter

Alright, the UI is a simple counter. The UI is just a button with a click handler. Every time the button is clicked, the counter increments.

- [React Demo](https://demo.garciadiazjaime.com/react-click-handler-performance)

The React code looks like this:

```javascript
"use client";

import { useState } from "react";

export default function Page() {
  const [counter, setCounter] = useState(0);

  const incrementClickHandler = (event: { preventDefault: () => void }) => {
    event.preventDefault();

    setCounter((prevCounter) => prevCounter + 1);
  };

  return (
    <div style={{ maxWidth: 800, margin: "0 auto" }}>
      <a
        href="#"
        style={{
          display: "inline-block",
          padding: "20px 40px",
          fontSize: 28,
          border: "1px solid black",
          width: "100%",
          textAlign: "center",
          marginTop: 40,
        }}
        onClick={incrementClickHandler}
      >
        Increment {counter}
      </a>
    </div>
  );
}
```

The code is pretty self-explanatory. One thing to note is that the demo runs on top of Next.js, which is why we need `"use client"`. Other than that, it's just a basic React component.

### React Counter UI

![Initial React Counter UI](/assets/resources-click-react-vanilla/qporuecnfxwxx3j0jju1.png)

### 20 Seconds and Only One Click

Now, I'm going to open the `Performance` tab in Chrome, click the record icon, and let it run for 20 seconds while clicking the button only once. At the end of the 20 seconds, the performance results look like this:

![20 Seconds and One Click Performance - React Version](/assets/resources-click-react-vanilla/345szgo8eq0jiu661ttx.png)

See how just one click bumps the numbers to:

|             | React |
| ----------- | ----- |
| `JS Heap`   | 3.4MB |
| `Nodes`     | 47    |
| `Listeners` | 287   |

🤯

### 20 Seconds with a Click per Second

Now, I’m going to let it run for another 20 seconds, but this time I’ll click the button once per second. Let’s take a look at the results:

![20 Seconds and 20 Clicks Performance - React Version](/assets/resources-click-react-vanilla/nk4dzxfhmvo3wf4aac20.png)

|             | React |
| ----------- | ----- |
| `JS Heap`   | 4MB   |
| `Nodes`     | 46    |
| `Listeners` | 331   |

Two things to note about React:

a) When a state variable is updated, the component is re-rendered, meaning that in this case, the component was rendered 20 times.

b) Thanks to the virtual DOM, React only updates the nodes that need to be updated.

Now, let’s go back to the chart and see how the blue line (`JS Heap`) and the yellow line (`Listeners`) increment, while the green line (`Nodes`) remains constant.

It's also worth mentioning that the numbers didn't change much compared to the one-click run.

## Demo: Building a Vanilla JavaScript Counter

Now, we have the same UI, but this time it’s built with vanilla HTML and JavaScript—no frameworks involved.

- [Vanilla Demo ](https://demo.garciadiazjaime.com/react-click-handler-performance/vanilla.html).

```html
<html>
  <head>
    <script>
      let increment = 0;

      window.onload = function () {
        document.querySelector("#counter").innerText = increment;

        document.querySelector("a").addEventListener("click", function (event) {
          event.preventDefault();
          increment++;
          document.querySelector("#counter").innerText = increment;
        });
      };
    </script>
  </head>
  <body style="max-width: 800; margin: 0 auto; font-family: monospace;">
    <a
      href="#"
      style="
        display: inline-block;
        padding: 20px 40px;
        font-size: 28px;
        border: 1px solid black;
        width: 100%;
        text-align: center;
        text-decoration: none;
        color: black;
        margin-top: 40;
        box-sizing: border-box;
      "
      >Increment <span id="counter"></span>
    </a>
  </body>
</html>
```

One thing to mention is the necessity of the following element:

```html
<span id="counter"></span>
```

that is manipulated with JavaScript to update its content:

```javascript
document.querySelector("#counter").innerText = increment;
```

### Vanilla Counter UI

![Vanilla Counter UI](/assets/resources-click-react-vanilla/ps03uqcjinnm2m92yp2l.png)

### 20 Seconds and Only One Click

Again, I’m going to click the record icon and let it run for 20 seconds, clicking the button only once.

Take a look at the results:

![20 Seconds and One Click Performance - Vanilla Version](/assets/resources-click-react-vanilla/jpm5ci5e0winepjpbn44.png)

|             | Vanilla |
| ----------- | ------- |
| `JS Heap`   | 1.7MB   |
| `Nodes`     | 16      |
| `Listeners` | 20      |

### 20 Seconds with a Click per Second

Again, I’m going to click the record icon and let it run for another 20 seconds, but this time, I’ll click the button once per second. Check out the results:

![20 Seconds and 20 Clicks Performance - Vanilla Version](/assets/resources-click-react-vanilla/tphizn7z85fx940m843w.png)

|             | Vanilla |
| ----------- | ------- |
| `JS Heap`   | 2.3MB   |
| `Nodes`     | 42      |
| `Listeners` | 77      |

Just like in the React example, the blue line (`JS Heap`) and the yellow line (`Listeners`) increased over time. However, the green line (`Nodes`) is not constant; it increases as the button is clicked.

## A Few Words on Garbage Collection

> **Garbage Collection**: The main concept that garbage collection algorithms rely on is the concept of reference.

JavaScript automatically handles garbage collection for us; we don’t need to trigger it manually. In the previous examples, we saw how resources are consumed, but at some point, JavaScript takes care of releasing some of those resources through its garbage collector.

## Conclusion

One click or twenty clicks isn’t that different in terms of resource consumption. As soon as a click happens, JavaScript allocates resources, and subsequent clicks continue to consume resources. However, the jump isn't as significant as the transition from zero to one click.

Let’s take a look at the end values for 20 clicks in both versions:

|             | Vanilla | React |
| ----------- | ------- | ----- |
| `JS Heap`   | 2.3MB   | 4.0MB |
| `Nodes`     | 42      | 46    |
| `Listeners` | 77      | 331   |

It makes sense that React consumes more resources; that’s the cost of using a framework.

One key difference is that React attaches all the nodes from the beginning, while the vanilla version adds nodes as the clicks happen. However, in the end, both versions ended up with pretty much the same number of nodes.

The demo is quite simple, and at this level, there’s no significant difference in terms of performance. As mentioned earlier, there’s a price to pay for using the framework, but it’s worth it considering all the features and conveniences it provides.
