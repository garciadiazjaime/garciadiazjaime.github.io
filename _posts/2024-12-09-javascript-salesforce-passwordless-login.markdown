---
layout: post
title: "Javascript: Implementing Passwordless Login with Salesforce"
date: 2024-12-09 10:00:00 -0500
categories: javascript programming passwordless salesforce
---

![React: Implementing Passwordless Login with AWS Cognito](/assets/react-aws-cognito-passwordless-login/banner.png)

Salesforce offers a Headless Passwordless Login Flow that allows registered users to access an application seamlessly. Passwordless login is very user friendly, all it requires is an active email address. In this post, I’ll share some code snippets for implementing the Passwordless login flow with Salesforce.

## Requirements

Before we start, ensure the following:  
a) You have access to a Salesforce environment.  
b) You’ve registered a user and enabled the Passwordless login option.

## Step One: Send Username

```javascript
export async function passwordlessLogin(username, captcha) {
  const payload = {
    username,
    recaptcha: captcha,
    verificationmethod: "email",
  };
  const config = {
    headers: {
      "Content-Type": "application/json",
    },
    method: "POST",
    body: JSON.stringify(payload),
  };
  const url = `${REPLACE_WITH_YOUR_SALESFORCE_CLOUD_URL}/services/auth/headless/init/passwordless/login`;

  const response = await fetch(url, config);

  const result = await response.json();

  return result;
}
```

This is the first call to Salesforce. Take note of the following:

a) You’ll need to pass a `captcha`. For assistance, check out [ReCAPTCHA v3](https://www.garciadiazjaime.com/posts/react-recaptcha-v3-nextjs).

b) The `verificationmethod` is set to `email`, which tells Salesforce to send a one-time password (OTP) via email.

c) Along with `captcha` and `verificationmethod`, the only other parameter required is `username`, which corresponds to the email registered for the user.

If the request is successful, Salesforce will send an email to the `username` provided and return a response like this:

```json
{
  "identifier": "MFF0RWswMDAwMDIxdVRk",
  "status": "success"
}
```

## Step two: Capture OTP

```javascript
export async function passwordlessAuthorize(identifier, code) {
  const Authorization = btoa(identifier + ":" + code);

  const config = {
    headers: {
      "Auth-Request-Type": "passwordless-login",
      "Auth-Verification-Type": "email",
      Authorization: "Basic " + Authorization,
      "Content-Type": "application/x-www-form-urlencoded",
    },
    method: "POST",
    body: new URLSearchParams({
      response_type: "code_credentials",
      client_id: "REPLACE_WITH_YOUR_CLIENT_ID",
      redirect_uri: "REPLACE_WITH_YOUR_REDIRECT_URI",
    }),
  };
  const response = await fetch(
    `${REPLACE_WITH_YOUR_SALESFORCE_CLOUD_URL}/services/oauth2/authorize`,
    config
  );

  const result = await response.json();

  return result;
}
```

This is the second call to Salesforce. Here are a few key points:

a) The `identifier` is the value returned from step one.  
b) The `code` is the OTP that Salesforce sent via email.  
c) Pay attention to how the `Auth` and `Authorization` headers are defined.  
d) The `Content-Type` is `application/x-www-form-urlencoded`. Notice how the body uses `URLSearchParams` for formatting.

If everything goes smoothly, Salesforce will return a response like this:

```json
{
  "code": "aPrxOPPU1bwu2d3SbsSBKLUbZop4sxhra2Tb.p3LApgVIexVmwyIGVaF6vTebI7ottVto18uuQ==",
  "sfdc_community_url": "https://site.com/application",
  "sfdc_community_id": "xxxxxxxx"
}
```

## Step Three: Exchange Code for Access Token

The final step is to exchange the `code` from the previous step for an `Access Token`. The `Access Token` is crucial because it allows you to make requests on behalf of the user. The presence of the `Access Token` enables a user's session.

```javascript
export async function getAccessToken(code) {
  const config = {
    headers: {
      "Content-Type": "application/x-www-form-urlencoded",
    },
    method: "POST",
    body: new URLSearchParams({
      code,
      grant_type: "authorization_code",
      client_id: "REPLACE_WITH_YOUR_CLIENT_ID",
      redirect_uri: "REPLACE_WITH_YOUR_REDIRECT_URI",
    }),
  };

  const response = await fetch(
    `${REPLACE_WITH_YOUR_SALESFORCE_CLOUD_URL}/services/oauth2/token`,
    config
  );

  const result = await response.json();

  return result;
}
```

The response should look something like this:

```json
{
  "access_token": "00DEj000006DHsR!AQEAQGpj5XvnBl1QQ8PI4XjygHmXAJiG7CA4Ci0mIxZcg7hO_YYZanyXPX9uelAez2905VFnE6VzhmavmnDoBOks.wzhlZHc",
  "sfdc_community_url": "https://site.com/application",
  "sfdc_community_id": "xxxxxxxx",
  "signature": "jwnfZY2G3phxCl3fJrfJu5X2AyxW7Ozsfg2BZ6bBB74=",
  "scope": "refresh_token openid user_registration_api api",
  "id_token": "...",
  "instance_url": "https://site.com/",
  "id": "https://test.salesforce.com/id/00000/11111",
  "token_type": "Bearer",
  "issued_at": "1733700157803"
}
```

Make sure to securely store the `access_token`, and from here, you can create a session for your user. For security reasons, it’s best to execute these methods on the server.

> The `id_token` is a JWT token. If you decode it, it will look something like this:

```json
{
  "at_hash": "HTa4VEmQhCYi59WLhiL6DQ",
  "sub": "https://test.salesforce.com/id/00000/11111",
  "aud": "3MXG9j6uMOMC1DNjcltNj9xPoUi7xNbiSwPqOjmDSLfCW54f_Qf6EG3EKqUAGT6xyGPc7jqAMi4ZRw8WTIf9B",
  "iss": "https://site.com/",
  "exp": 1733702662,
  "iat": 1733702542
}
```

You can also customize the JWT to include additional data. However, it’s recommended to keep the structure minimal and use the `Access Token` for fetching additional information as needed.

## Conclusion

Passwordless login is convenient for everyone, and most cloud services, like Salesforce, now offer a Passwordless Login Flow. Leverage this feature to simplify the login process and improve the user experience.
