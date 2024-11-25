---
layout: post
title: "React: How Often Does a Cleanup Function Run?"
date: 2024-10-14 10:00:00 -0500
categories: javascript programming react webdev
---

![React: How Often Does a Cleanup Function Run?](/assets/react-cleanup-function-run/banner.png)

React components with state variables trigger a re-render when those variables are updated. This is expected and is one of React's core features. Additionally, components offer a cleanup function, which is triggered every time the component is "unmounted." But how often does this cleanup function actually run?

Well, as we’ll explore in this demo, the cleanup function can have multiple triggers, but a common one is updates to a state variable when it’s linked to a side effect.

In short, if your React component is using `useEffect` and follows a specific state variable, let's look at the following example:

```javascript
useEffect(() => {
  console.log("Page mounted");

  return () => {
    // cleanup function
    console.log("Page unmounted");
  };
}, [counter]);
```

The cleanup function will be called every time `counter` is updated.

When a state variable is updated, React re-renders the component, which means it first needs to "remove" it and then "render" it again. While the virtual DOM optimizes which nodes need updating, at the logic level, the cleanup function still gets called.

In most cases, this is fine and expected. However, if your cleanup function is doing things like reporting events, removing listeners, etc., make sure that’s intentional, since the cleanup function will be called multiple times based on the state variable.

## Demo

Let's see a [demo](https://www.garciadiazjaime.com/react-cleanup-function).

- In the developer tools, open the console.
- You will notice two logs:

```shell
Child mounted
Page mounted
```

![Initial UI - Components mounted](/assets/react-cleanup-function-run/nrnth8ipo1u9a4pt4p1e.png)

That means both components, Page and Child, were rendered.

- If you remember the snippet shared above, `useEffect` is tied to `counter`, which is incremented when clicking the first button. Right now, it should say: `Increment 0`. Let's go ahead and click it.

- Keep an eye on the Console and notice how four logs were added:

```
Child unmounted
Page unmounted
Child mounted
Page mounted
```

![Components unmounted and mounted after a click](/assets/react-cleanup-function-run/dpbnwj8k4qq8ihxg6ity.png)

The state variable `counter` got updated, and since there are two `useEffect` hooks tied to `counter`, it means their cleanup functions were executed. Notice how, for the Page component, `useEffect` is tied to the state variable, while for the Child component, the side effect is tied to a prop variable, where the source is still the same `counter` state variable.

Additionally, you can see the button `Flag Off`, which updates another state variable that is not linked to `useEffect`. This means that clicks on this button won't trigger the cleanup function.

## Conclusion

Cleanup functions are heavily used in React, especially to report things once the component is "done." However, be cautious about tying the `useEffect` to a state variable. As seen in the demo, this causes the cleanup function to be called every time the state variable is updated, which might not be expected.

Most of the time, the cleanup function is housed in a `useEffect` without any state variables.

```javascript
useEffect(() => {
  console.log("Page mounted");

  return () => {
    console.log("Page unmounted");
  };
}, []);
```

It's fine to have multiple `useEffect` hooks in one component. In this case, you could have one that reacts to a state variable and another to set the cleanup function.

```javascript
useEffect(() => {
  // side effects for counter
}, [counter]);

useEffect(() => {
  console.log("Page mounted");

  return () => {
    console.log("Page unmounted");
  };
}, []);
```

To answer the question, the cleanup function will be called when the component is unmounted. This could happen when the user navigates to another section of the app, or if `useEffect` depends on a state variable, then it will be called N times whenever that state variable is updated.
