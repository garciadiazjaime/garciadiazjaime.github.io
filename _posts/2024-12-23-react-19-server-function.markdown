---
layout: post
title: "React 19: Server Functions"
date: 2024-12-23 10:00:00 -0500
categories: react javascript server-functions programming
---

![React 19: Server Functions](/assets/react-19-use-actions-state/banner.png)

Server Functions are functions referenced on the client but executed on the server.

Here’s an example:

```javascript
'use client'

import { useActionState } from "react";

import { updateName } from "@/app/react-19-server-function/actions";


export default function Page() {

    const [error, submitAction, isPending] = useActionState(
        async (_previousState, formData) => {
            const error = await updateName(formData.get("name") as string);
            if (error) {
                return error;
            }

            return ""
        },
        "",
    );

    return <div>
        <h1>React 19: Server Functions</h1>
        <fieldset>
            <div>Name</div>

            <form action={submitAction}>
                <input type="text" name="name" />

                <button type="submit" disabled={isPending}>Save</button>
            </form>

            <div>
                {error && <p>{error}</p>}
            </div>
        </fieldset>
    </div>
}
```

> Check my earlier post for more details on [useActionState](https://www.garciadiazjaime.com/react-19-use-actions-state)

Notice how `updateName` is imported.

```javascript
import { updateName } from "@/app/react-19-server-function/actions";
```

and passed into `useActionState`.

This means that whenever the form is submitted, it runs `submitAction`, which then calls `updateName`.

Now, let’s check out `updateName`:

```javascript
"use server";

export async function updateName(name) {
  if (name?.length < 2) {
    return "Name must be at least 2 characters.";
  }

  return "";
}
```

It’s a very simple function that checks the length of `name`. If it has less than 2 characters, it returns an error; otherwise, it returns an empty string, meaning no error.

Another thing to note is the directive: `"use server"`. This tells React the function will be executed on the server, so it creates a reference the client can use.

The UI is super straightforward and looks like this:

![React 19: Server Functions UI](/assets/react-19-server-function/ui.png)

If the form is submitted without any value, you’ll notice a POST network request with a few interesting details:

- `content-type:` is `text/x-component`.

![React 19: Server Functions Content Type](/assets/react-19-server-function/content-type.png)

- The payload is passed automatically, even though it’s empty.

![React 19: Server Functions Payload](/assets/react-19-server-function/payload.png)

- The response is kind of like JSON.

![React 19: Server Functions Response](/assets/react-19-server-function/response.png)

## Links

- [Codebase](https://github.com/garciadiazjaime/demo-reactjs/blob/main/app/react-19-server-function/page.tsx)

## Conclusion

Server Functions are functions that run on the server. The alternative is to manually use `fetch` to make a request to the backend and handle things like reading the status code and parsing the payload.

With Server Functions, you don’t need to worry about the communication part. Just create a function with the `"use server"` directive and import it into a client file, the framework takes care of the rest.
