---
layout: post
title: "React: ReCAPTCHA v3 Client and Server Demo"
date: 2024-11-18 10:00:00 -0500
categories: javascript react recaptcha nextjs
---

![React: ReCAPTCHA v3 Client and Server Demo](/assets/react-recaptcha-v3-nextjs/banner.png)

In this demo, I’ll use Google ReCAPTCHA v3 credentials within a React application built on Next.js. The ReCAPTCHA token will be generated on the client side and validated on the server side.

## Links

- [Demo](https://demo.garciadiazjaime.com/react-recaptcha-v3-nextjs)

![Demo](/assets/react-recaptcha-v3-nextjs/demo.gif)

- [Codebase](https://github.com/garciadiazjaime/demo-reactjs/blob/main/app/react-recaptcha-v3-nextjs/page.tsx)

## Step 1: Generate Your ReCAPTCHA Credentials

Go to Google ReCaptcha V3 and generate your credentials.

![Generate Your ReCAPTCHA Credentials](/assets/react-recaptcha-v3-nextjs/google-recaptcha-credentials.png)

## Step 2: Import the ReCaptcha library

```
<Script src={`https://www.google.com/recaptcha/enterprise.js?render=${process.env.NEXT_PUBLIC_RE_CAPTCHA_SITE_KEY}`} />
```

> Note: There are some packages you could use, but the implementation is simple.

## Step 3: Call the `execute` method in your click handler

```javascript
const loginClickHandler = (event) => {
  event.preventDefault();

  grecaptcha.enterprise.ready(async () => {
    const token = await grecaptcha.enterprise.execute(
      process.env.NEXT_PUBLIC_RE_CAPTCHA_SITE_KEY,
      { action: "LOGIN" }
    );

    await submit(token);
  });
};
```

`grecaptcha` is an object injected by the imported script.

> Note: When using Next.js, ensure all environment variables exposed in the browser are prefixed with `NEXT_PUBLIC`.

When the user clicks `login`, the app automatically generates a `captcha` for them by calling two methods from the `grecaptcha` object:

- `window.grecaptcha.enterprise.ready`: This makes sure the Google reCAPTCHA object is ready to go.
- `window.grecaptcha.enterprise.execute`: This generates the captcha token.

Finally, the data is sent to the backend (in my case, I’m using a Lambda function), along with the generated captcha token.

```javascript
const submit = async (code) => {
  await fetch("`/.netlify/functions/react-recaptcha-v3-nextjs", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
    },
    body: JSON.stringify({ code }),
  });
};
```

> Note: If you are working with a form, you’d also include other field values like username, name, or any additional data your form collects.

## Step 4: Validate the Captcha on the Backend

```javascript
const validateReCaptcha = async (captcha) => {
  const url = `https://www.google.com/recaptcha/api/siteverify?secret=${process.env.RE_CAPTCHA_SECRET_KEY}&response=${captcha}`;
  const response = await fetch(url, {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
    },
    body: JSON.stringify({ captcha }),
  });

  return response.json();
};
```

`validateReCaptcha` is a backend method that calls a Google API endpoint, passing the `SECRET_KEY` (stored as an environment variable) and the `Captcha` token generated on the client.

If the Captcha is valid, the API response will look something like this:

```
{
  "success": true,
  "challenge_ts": "2024-11-24T03:04:34Z",
  "hostname": "localhost",
  "score": 0.9
}
```

## Conclusion

ReCaptcha is crucial for securing forms, especially when you're looking to prevent bots from submitting them. Google offers a free tier that provides up to 10,000 assessments per month (at the time of writing), making it a solid choice for many applications. The integration is made easier with the library that google provides. You'll just need to pass your credentials: `SITE_KEY` on the client side and `SECRET_KEY` on the server side.

A key point to remember is that the `SECRET_KEY` should never be exposed on the client side, as this could compromise the security of your application. Only the `SITE_KEY` is meant for the client.
