---
layout: post
title: "React + AWS Cognito: Email Authentication Setup Guide (First Part)"
date: 2024-11-04 10:00:00 -0500
categories: javascript programming react aws-cognito
---

![React + AWS Cognito: Email Authentication Setup Guide (First Part)](/assets/aws-cognito-user-password-authentication/banner.png)

This is the first in a two-part series where we’ll build a React app using AWS Cognito for email-based user authentication. Part 1 focuses on setting up the necessary AWS configurations, while Part 2 will dive into the React code to tie it all together.

We’ll work with the following AWS services:

- **Amazon Simple Email Service (SES)**
- **AWS Cognito**

Let’s start by setting up our AWS resources.

## Amazon Simple Email Service (SES)

### Domain SES Identity

For testing purposes, verifying domain ownership in SES is optional, as AWS offers a workaround. However, for production, verifying ownership is essential to allow SES to send emails on behalf of your domain.

Here’s the setup process:

1. Go to **Amazon Simple Email Service**.
2. Select **Identities**.
3. Click **Create Identity**.

In the setup, I chose "Domain" and used the example `domain.com`.

![AWS SES Entity Domain Creation](/assets/aws-cognito-user-password-authentication/aws-ses-domain-create.png)

- Click **Create identity**.

Then, you’ll see a page similar to this one:

![AWS SES Entity Domain Confirm](/assets/aws-cognito-user-password-authentication/aws-ses-domain-confirm.png)

Navigate to the **Publish DNS records** section, and use those values to add the records in your domain provider.

Once the DNS records are set up in your domain provider, you should see your domain verified, looking something like this:

![AWS SES Entity Domain Verified](/assets/aws-cognito-user-password-authentication/aws-ses-domain-verified.png)

Perfect, your domain is now verified, which allows SES to send emails on your behalf. This verification isn't required for testing since AWS provides an alternate method, but it’s essential for production.

### Email SES Identity

When testing, this step is important because the email address you use in your authentication flow needs to be added to AWS's "allow list." Here, we’ll add and verify an email address.

Head to:

- **Amazon Simple Email Service**
- **Identities**
- **Create identity**

This time, select **Email address**:

- Enter the email address you want to verify.
- Click **Create identity**.

![AWS SES Entity Email Creation](/assets/aws-cognito-user-password-authentication/aws-ses-email-create.png)

Once the identity is created, you'll receive an email from AWS containing a verification link. Make sure to check your inbox and click on that link to verify your email.

![AWS SES Entity Email Confirm](/assets/aws-cognito-user-password-authentication/aws-ses-email-confirm.png)

Once you've verified your email, you should see a label indicating that it's `verified`.

![AWS SES Entity Email Verified](/assets/aws-cognito-user-password-authentication/aws-ses-email-verified.png)

At this point, you should have both your domain and email verified. While the domain verification is optional during testing, it becomes necessary in production. On the other hand, email verification is required for testing but not for production.

> Note: As part of the authentication flow, a confirmation code will be sent to this email account. If the email account is not verified, it won't receive the code.

## Amazon Cognito

The last piece to configure is **Cognito**. This service enables account authentication, and in this case, we'll use **email** for authentication. Here's how it works:

- The user creates an account with their email and password.
- They verify their email by entering the code sent by AWS.
- Once verified, the user can log in using their email and password.

Instead of handling authentication yourself, you can leverage AWS Cognito.

Let's go to:

> Note: For most steps, I'm sticking with the default options, so I'll only mention the custom choices I make. Depending on your project, you may want to configure different settings.

- **Cognito**
- **Create user pool**
- **Step 1**: Check **Email**

![AWS Cognito Step 1](/assets/aws-cognito-user-password-authentication/aws-cognito-step1.png)

- **Step 2**: Select **No MFA**; this isn't necessary for testing.

![AWS Cognito Step 2](/assets/aws-cognito-user-password-authentication/aws-cognito-step2.png)

- **Step 3**: I kept the default options.

![AWS Cognito Step 3](/assets/aws-cognito-user-password-authentication/aws-cognito-step3.png)

- **Step 4**: Choose your verified "From email address."

Cognito will send an email with a verification code, ideally from your domain, which is why the domain needs to be verified in the previous section. Here, you can see that AWS offers the option to **"Send email with Cognito"** which is suitable for development. However, in production, you’ll want to ensure that your domain is verified for a more professional and reliable email sending process.

![AWS Cognito Step 4](/assets/aws-cognito-user-password-authentication/aws-cognito-step4.png)

- **Step 5**: In addition to adding a pool and client name, **the key part is to expand the "Advanced app client settings" and enable `ALLOW_USER_PASSWORD_AUTH`**. This setting allows users to authenticate using their email and password, which is essential for your authentication flow.

![AWS Cognito Step 5](/assets/aws-cognito-user-password-authentication/aws-cognito-step5.png)

- **Step 6**: This is the review step, and there's nothing to edit here. Just make sure everything looks good before moving forward.

![AWS Cognito Step 6](/assets/aws-cognito-user-password-authentication/aws-cognito-step6.png)

Once created, you should see it on the dashboard like this:

![AWS Cognito Pool Created](/assets/aws-cognito-user-password-authentication/aws-cognito-created.png)

That's it! You now have everything set up on AWS. In the next post, I'll demonstrate how to connect your React app with Cognito to authenticate a user using their email. Look out for the post, which will be published next Monday.
