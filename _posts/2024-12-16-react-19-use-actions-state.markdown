---
layout: post
title: "React 19: New hook useActionState"
date: 2024-12-16 10:00:00 -0500
categories: react javascript server-actions programming
---

![React 19: New hook useActionState](/assets/react-19-use-actions-state/banner.png)

Usually, when working with a form, you’d want to:

a) Display a loader

b) Show validation errors

This often means managing a couple of state variables. But with the new `useActionState` hook introduced in React 19, there’s now a simpler way to handle it.

## Links

- [Demo](https://demo.garciadiazjaime.com/react-19-use-action-state)

- [Codebase](https://github.com/garciadiazjaime/demo-reactjs/blob/main/app/react-19-use-action-state/page.tsx)

## React Hook: `useActionState`

In the following snippet, notice how `useActionState` is being used:

```javascript
import { useActionState } from "react";
import Loader from "@/components/loader";

function Form() {
  const [error, submitAction, isPending] = useActionState(
    async (previousState, formData) => {
      const error = await updateName(formData.get("name"));

      if (error) {
        return error;
      }

      return "";
    },
    ""
  );

  return (
    <form action={submitAction}>
      Name: <input type="text" name="name" />
      <button type="submit" disabled={isPending}>
        Save
      </button>
      {isPending && <Loader />}
      {error && <p>{error}</p>}
    </form>
  );
}
```

A couple of things to mention:

- `useActionState` returns three things:

1. The first variable, `error`, is for errors. In this case, it’s a string, but it could really be any type.

2. The second variable, `submitAction`, is a function that triggers the form submission.

3. The third variable, `isPending`, is a boolean that indicates the `pending` state. It’s useful for disabling elements or showing a spinner when the form is being submitted.

- `useActionState` expects two parameters:

1. The first parameter is the function that gets triggered by `submitAction` when the form is submitted.

2. The second parameter is the initial value for the `error`. In this case, it’s an empty string, but you could use something like "fill in all the fields" instead.

- `useActionState` provides `formData`, which is an instance of `FormData`, basically, an object that represents the fields in the form.

Input field in the form:

```html
<input type="text" name="name" />
```

Can be read like this:

```javascript
formData.get("name");
```

> `name` is the name of the input field.

- `error` will display any error from the server. If there’s none, it will be empty.

- `isPending` is automatically updated by the hook to `true` when the form is submitted and set back to `false` once the backend response is received.

> if there’s no error, `useActionState` also handles resetting the form.

## Request to the Backend

For this demo, the `updateName` function is pretty basic, it just makes a `POST` request to the backend, passing the `name`. If the backend returns an error, the function returns it; otherwise, it returns an empty string, meaning there was no error from the server.

```javascript
async function updateName(name) {
  const response = await fetch("/lambda-function", {
    method: "POST",
    body: JSON.stringify({ name }),
  });

  if (!response.ok) {
    const { message } = await response.json();
    return message;
  }

  return "";
}
```

## Backend POST request handler

I'm using a simple lambda function that just checks if the name is a string with at least 2 characters. There’s also a 2 second delay, just to give the UI a bit of time to show the spinner.

```javascript
const sleep = (ms) => {
  return new Promise((resolve) => setTimeout(resolve, ms));
};

const handler = async (event) => {
  const { name } = JSON.parse(event.body || "{}");

  await sleep(2_000);

  if (name?.length < 2) {
    return {
      statusCode: 400,
      body: JSON.stringify({ message: "Name must be at least 2 characters." }),
    };
  }

  return {
    statusCode: 200,
  };
};
```

## Conclusion

React 19's `useActionState` hook is helpful when dealing with two things you should always have in a form: `a pending state and validation errors`. The hook takes care of updating those "state variables" and even provides a reference to `previousState` if you want to compare values.

What happens after the form is submitted and no error is returned is up to the application. In most cases, this means redirecting the user to another page or showing a success message.
