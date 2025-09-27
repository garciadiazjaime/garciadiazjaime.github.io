---
layout: post
title: "NextAuth.js: An easy way to add authentication to your Next.js project"
date: 2025-06-09 10:00:00 -0500
categories: nextjs react nextAuth.js authentication
---

![NextAuth.js: An easy way to add authentication to your Next.js project](/assets/nextjs-auth-session-login/banner.png)

Recently, I needed to add session management to a Next.js project and came across [NextAuth.js](https://next-auth.js.org/). It’s a pretty solid library that simplifies login, logout, and session handling.

Here are the steps I followed:

## 1. API Auth Route Handler

First, create the following file:

```bash
api/auth/[...nextauth]/route.ts
```

And add this content: (you can customize it based on your provider and config needs)

```javascript
import NextAuth from "next-auth";
import CredentialsProvider from "next-auth/providers/credentials";

type UserToken = { id: string; name: string; email: string };

const handler = NextAuth({
  providers: [
    CredentialsProvider({
      name: "Credentials",
      credentials: {
        username: { label: "Username", type: "text", placeholder: "admin" },
        password: {
          label: "Password",
          type: "password",
          placeholder: "1234",
        },
      },
      async authorize(credentials, req) {
        return { id: "1", name: "Admin", email: "admin@example.com" };
      },
    }),
  ],
  secret: process.env.NEXTAUTH_SECRET,
  callbacks: {
    async jwt({ token, user }) {
      if (user) {
        token.user = user;
      }
      return token;
    },
    async session({ session, token }) {
      session.user = token.user as UserToken;
      return session;
    },
  },
});

export { handler as GET, handler as POST };
```

This is the main configuration file, and it defines a few key things:

- Provider

  `NextAuth.js` supports multiple providers. For this simple demo, I’m using Credentials, which is super handy when you already have a login system in place. It basically lets you plug directly into your existing auth service.

- authorize

  This function is where you hook into your internal login service. If it returns an object, `NextAuth.js` treats it as a successful login and creates a session. If you return null, it means login failed.

> Note: providers is an array—so you can support multiple login methods, like `Credentials` (custom login), `Google`, `GitHub`, etc.

- JWT

  This callback attaches the user data returned by `authorize` to the token. You can include whatever you want inside the token here. That token will later be used to build the session.

- session

  This is where the session is created using the token. You’ll see how the user info gets added to the session object. This is what your app can access using the `useSession()` hook on the client side.

For this demo, I’m not calling any real auth service. Just passing a user and password is enough to simulate a session. But in a real-world scenario, you’d call your actual login API inside `authorize`, and then return the user object (or `null` on failure).

## 2. UI Updates

Now that the provider setup is done, the next step is to implement the UI components. In this case, I only have two components:

- Page

  A very simple page that uses `'use client'` (required for client-side hooks like `useSession`) and wraps everything in `SessionProvider`:

```javascript
"use client";

import { SessionProvider } from "next-auth/react";

import LoginButton from "./login-btn";

export default function Page() {
  return (
    <SessionProvider>
      <LoginButton />
    </SessionProvider>
  );
}
```

> Note: The `LoginButton` component uses `useSession`, so it needs to be inside a `SessionProvider`.

- LoginButton

  A very simple LoginButton component that uses `useSession` to either show a `Sign In` or `Sign Out` button depending on the session state:

```javascript
import { useSession, signIn, signOut } from "next-auth/react";

export default function Component() {
  const { data: session } = useSession();

  if (session) {
    return (
      <div>
        <p>
          Signed in as <strong>{session.user.name}</strong>
        </p>
        <button onClick={() => signOut()}>Sign out</button>
      </div>
    );
  }

  return (
    <div>
      <p>Not signed in</p>
      <button onClick={() => signIn()}>Sign in</button>
    </div>
  );
}
```

And that’s it—you’ve got a simple login system in place using NextAuth.js

![Sign In Using NextAuth.js](/assets/nextjs-auth-session-login/sign_in.png)

![Default Login Form](/assets/nextjs-auth-session-login/login_form.png)

![Sign Out Using NextAuth.js](/assets/nextjs-auth-session-login/sign_out.png)

## Demo

Make sure to check out the [Website](https://demo.garciadiazjaime.com/nextjs-next-auth-login) and take a look at the [Codebase](https://github.com/garciadiazjaime/demo-reactjs/blob/main/app/api/auth/%5B...nextauth%5D/route.ts).

Thanks for reading!
