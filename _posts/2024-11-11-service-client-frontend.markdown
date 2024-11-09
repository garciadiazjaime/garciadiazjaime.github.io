---
layout: post
title: "React + AWS Cognito: Email Authentication Setup Guide (Second Part)"
date: 2024-11-11 10:00:00 -0500
categories: javascript programming react aws-cognito
---

![React + AWS Cognito: Email Authentication Setup Guide (Second Part)](/assets/aws-cognito-user-password-authentication/banner_2.png)

In the [last post](https://www.garciadiazjaime.com/posts/aws-cognito-user-password-authentication-setup), we handled everything on the AWS side; now let's dive into React to set up our code.

AWS provides the npm package `@aws-sdk/client-cognito-identity-provider`, which includes functions for:

- Creating an account using an email and password
- Verifying the email via a code sent by AWS
- Logging in with the email and password

![React + AWS Cognito: Email Authentication Setup Guide (Second Part)](/assets/aws-cognito-user-password-authentication/aws-actions.png)

> Check out the [demo page](https://demo.garciadiazjaime.com/aws-cognito-user-password-authentication) to try it yourself, and feel free to look at the code in the [GitHub repository](https://github.com/garciadiazjaime/demo-reactjs/blob/main/app/aws-cognito-user-password-authentication/page.tsx).

## Sign up

The first step is signing up

```typescript
import { SignUpCommand } from "@aws-sdk/client-cognito-identity-provider";

const AWS_CLIENT_ID = "REPLACE_WITH_YOUR_AWS_CLIENT_ID";

export const signUp = async (email: string, password: string) => {
  const params = {
    ClientId: AWS_CLIENT_ID,
    Username: email,
    Password: password,
    UserAttributes: [
      {
        Name: "email",
        Value: email,
      },
    ],
  };

  const command = new SignUpCommand(params);

  try {
    await cognitoClient.send(command);
  } catch (error) {
    throw error;
  }
};
```

Note how `AWS_CLIENT_ID` is required, and this helper function takes in `email` and `password`. In the demo, both values are input by the user in a form, and the UI then calls this method, passing those values.

If there’s an error, an exception is thrown, and the UI handles it accordingly.

## Confirmation

> **Note:** During testing, any email used in the form must first be verified. This won’t be necessary in production.

When `SignUpCommand` runs, AWS registers the account and sends a verification code by email, so the next step is to check the inbox and copy the code.

```typescript
import { ConfirmSignUpCommand } from "@aws-sdk/client-cognito-identity-provider";

const AWS_CLIENT_ID = "REPLACE_WITH_YOUR_AWS_CLIENT_ID";

export const confirmSignUp = async (username: string, code: string) => {
  const params = {
    ClientId: AWS_CLIENT_ID,
    Username: username,
    ConfirmationCode: code,
  };

  const command = new ConfirmSignUpCommand(params);
  try {
    await cognitoClient.send(command);
  } catch (error) {
    throw error;
  }
};
```

Notice that `ConfirmSignUpCommand` requires your AWS `ClientId`, `username` (email), and the confirmation code that was sent to the email.

## Sign In

If `ConfirmSignUpCommand` completes successfully, the account should be all set for logging in.

```typescript
import { SignUpCommand } from "@aws-sdk/client-cognito-identity-provider";

const AWS_CLIENT_ID = "REPLACE_WITH_YOUR_AWS_CLIENT_ID";

export const signIn = async (username: string, password: string) => {
  const params = {
    AuthFlow: AuthFlowType.USER_PASSWORD_AUTH,
    ClientId: AWS_CLIENT_ID,
    AuthParameters: {
      USERNAME: username,
      PASSWORD: password,
    },
  };
  const command = new InitiateAuthCommand(params);

  let AuthenticationResult;
  try {
    const response = await cognitoClient.send(command);
    AuthenticationResult = response.AuthenticationResult;
  } catch (error) {
    throw error;
  }

  if (!AuthenticationResult) {
    return;
  }

  sessionStorage.setItem("idToken", AuthenticationResult.IdToken || "");
  sessionStorage.setItem("accessToken", AuthenticationResult.AccessToken || "");
  sessionStorage.setItem(
    "refreshToken",
    AuthenticationResult.RefreshToken || ""
  );

  return AuthenticationResult;
};
```

In the `InitiateAuthCommand`, AWS requires the `ClientId`, `username` (email), and `password` provided by the user through the form. If the email has already been verified, the login will succeed.

Additionally, some values are stored in `sessionStorage` for potential future use.

## Conclusion

> Check out the [demo](https://demo.garciadiazjaime.com/aws-cognito-user-password-authentication) and explore the [code base](https://github.com/garciadiazjaime/demo-reactjs/blob/main/app/aws-cognito-user-password-authentication/page.tsx).

Cognito is relatively easy to set up yet powerful. It handles essentials like creating, verifying, and authenticating accounts. While building your own service for this is possible, it demands significant effort for proper implementation and maintenance.

When starting a project, cloud services offer the advantage of offloading these responsibilities so you can focus on your core business logic, even if it introduces some dependency.
