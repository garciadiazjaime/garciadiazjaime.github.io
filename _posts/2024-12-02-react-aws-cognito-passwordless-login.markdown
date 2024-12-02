---
layout: post
title: "React: Implementing Passwordless Login with AWS Cognito"
date: 2024-12-02 10:00:00 -0500
categories: javascript react passwordless cognito
---

![React: Implementing Passwordless Login with AWS Cognito](/assets/react-aws-cognito-passwordless-login/banner.png)

Passwords are easy to forget, and users often reuse the same password for everything. On the other hand, almost everyone has access to email, making passwordless login a more user-friendly and secure option. In this post, I'll demo how to implement passwordless login using AWS Cognito.

## Links

- [Demo](https://demo.garciadiazjaime.com/react-aws-cognito-passwordless-login/sign-up)

- [Codebase](https://github.com/garciadiazjaime/demo-reactjs/blob/main/app/react-aws-cognito-passwordless-login/sign-up/page.tsx)

## Requirements

First, you’ll need to [set up AWS SES and Cognito](https://www.garciadiazjaime.com/posts/aws-cognito-user-password-authentication-setup)

Then, do the following:

1. Enable the `ALLOW_USER_AUTH` permission
   - Go to the app client, click **Edit**, and select: `Choice-based sign-in: ALLOW_USER_AUTH`.

![Enable the `ALLOW_USER_AUTH` permission](/assets/react-aws-cognito-passwordless-login/ALLOW_USER_AUTH.png)

2. Enable the `Email message one-time password` sign-in option
   - Navigate to your Cognito user pool, edit the **Sign-in** options, and select `Email message one-time password`.

![Enable the `Email message one-time password` sign-in option](/assets/react-aws-cognito-passwordless-login/one-time-password-option.png)

## Pool ID and Client ID

Your React application will require the AWS Cognito Pool ID and Cognito App ID:

- **Pool ID**:

![AWS Cognito Pool ID](/assets/react-aws-cognito-passwordless-login/aws-cognito-pool-id.png)

- **Client ID**

![AWS Cognito Client ID](/assets/react-aws-cognito-passwordless-login/aws-cognito-client-id.png)

## Amplify Configuration

I’ll be using the npm package `aws-amplify`, which provides handy helpers. First, let’s configure `Amplify`:

```javascript
import { Amplify } from "aws-amplify";

Amplify.configure({
  Auth: {
    Cognito: {
      userPoolId: "REPLACE_WITH_YOUR_COGNITO_POOL_ID",
      userPoolClientId: "REPLACE_WITH_YOUR_COGNITO_CLIENT_ID",
    },
  },
});
```

## Create Account

Now that `Amplify` is configured, let’s use the `signUp` method to create an account.

```javascript
import { signUp } from "aws-amplify/auth";

await signUp({
  username: "REPLACE_THIS_WITH_AN_EMAIL",
  password: "REPLACE_THIS_WITH_A_RANDOM_PASSWORD",
  options: {
    userAttributes: {
      email: "REPLACE_THIS_WITH_AN_EMAIL",
    },
  },
});
```

Notice that the `email` is set as both the `username` and the `email`.

> The user will log in with a one-time password, but to create the account, a password needs to be defined. In this case, I’m passing a random password that will never actually be used.

## Confirm Account

The `signUp` method will send an email to the user, which looks like this:

![Email with Confirmation Code](/assets/react-aws-cognito-passwordless-login/account-creation-code.png)

```javascript
import { confirmSignUp } from "aws-amplify/auth";

await confirmSignUp({
  username: "REPLACE_THIS_WITH_AN_EMAIL",
  confirmationCode: "608809",
});
```

Copy the code you received from the email and pass it to `confirmSignUp`, along with the same email you used for `signUp`.

## Request One-time Password

```javascript
import { signOut, signIn, confirmSignIn } from "aws-amplify/auth";

await signIn({
  username: "REPLACE_THIS_WITH_AN_EMAIL",
  options: { authFlowType: "USER_AUTH" },
});

await confirmSignIn({ challengeResponse: "EMAIL_OTP" });
```

Notice that two calls need to be made:

- The first call is to initiate a session with `signIn`, passing the email and the `USER_AUTH` parameter.
- The second call requests AWS to send the OTP via email with the `EMAIL_OTP` parameter.

The user will then receive an email with an OTP that looks like this:

![Email with OTP](/assets/react-aws-cognito-passwordless-login/login-otp.png)

## Passwordless Login with an OTP

```javascript
import { confirmSignIn } from "aws-amplify/auth";

await confirmSignIn({ challengeResponse: "30079914" });
```

Finally, `confirmSignIn` is called again, but this time, you pass the OTP received in the email.

If everything goes smoothly, `confirmSignIn` will authenticate the user, and your application can now display the logged-in elements.

## Conclusion

From the user’s perspective, passwordless login is a much easier way to log in. As long as the user has access to an email, they can log into any passwordless application. In this demo, I’m using Cognito, but the same flow applies to any provider, even a custom one. The user provides their email, the system sends an email with a one-time password, and the user logs in using this OTP.
