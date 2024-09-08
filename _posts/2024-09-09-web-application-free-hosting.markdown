---
layout: post
title: "Front-End Architecture: How to Host Your Web App for Free"
date: 2024-09-09 08:00:00 -0500
categories: javascript reactjs nextjs web-development unit-test
---

![Front-End Architecture: How to Host Your Web App for Free](/assets/front-end-free-hosting/banner.png)

In my post [Why Should You Write Unit Tests?](https://www.garciadiazjaime.com/posts/react-nextjs-unittest), I mentioned a personal project I created to help users find [Chicago music concerts](https://www.chicagomusiccompass.com/). One notable aspect of this project from an architectural perspective is that the entire web application is hosted completely for free.

The image below illustrates the architecture components:
![Chicago Music Compass Front End Architecture](/assets/front-end-free-hosting/front-end-architecture.png)

_Disclaimer:_ I am not sponsored by any of the services I mention here; I am highlighting them simply because I have found them useful.

## Project Codebase Repository

GitHub is the most popular platform for hosting your codebase for free. I've also tried alternatives like Bitbucket and GitLab, and they work just as well. Honestly, any of these options will do the job—just pick one and move forward without overthinking it.

## Front-End Workflow Orchestration

In a professional setting, you'd typically work directly with AWS or another cloud service provider, giving you full control over deployments, notifications, and monitoring. However, this approach requires more time and effort. Fortunately, services like Netlify and Vercel simplify this process by removing much of the friction. They allow for quick deployments but come with a dependency on their ecosystem. Once your site starts receiving significant traffic (thousands of visits), it’s a good idea to check the free tier limits to avoid unexpected costs. For personal projects, I’ve used these services multiple times without any issues so far.

That said, Netlify offers several features out of the box. In the architecture image, three of the components are provided automatically by Netlify:

- **GUI Integration**:

  Netlify’s GUI allows you to integrate your codebase repository—GitHub, in my case. Netlify understands the default settings for a Next.js application and uses them to deploy the code seamlessly.

- **Static Site Hosting**:

  `chicagomusiccompass.com` is a static web application, meaning there’s no server involved. When a deployment is triggered, the app generates static assets (HTML, JS, and CSS) that are stored in an S3 bucket. Netlify then handles the configuration with CloudFront, providing you with a ready-to-use URL.

- **Lambda Functions**:

  Static sites often need to fetch data from other domains. This usually requires a proxy, known as a "Back End for Front End" (BFF). Client applications, by default, don’t have access to other domains unless the server explicitly allows it via CORS, which isn’t always common practice. For this project, I'm using a proxy to pull a **JSON file** from a different domain.

Netlify manages all the deployment orchestration and provides a URL (subdomain) that you can link to your domain for a user-friendly URL.

For example, this is the **Netlify URL** for my project:

```
https://clinquant-chebakia-f64a5b.netlify.app/
```

I then configured my domain with a CNAME record to point `www` to the Netlify URL:
![CNAME record pointing to deploy URL](/assets/front-end-free-hosting/cname.png)

When a user visits [https://www.chicagomusiccompass.com/](https://www.chicagomusiccompass.com/), DNS resolves the domain to its final destination—the Netlify URL 🤓.

While there’s a lot happening here, most of it is configured through dashboards (GUI). The key is to understand how everything is connected; the rest is just navigating the UI.

## Automated Scheduled Tasks (Cron Jobs)

cron-job.org is a service that allows you to run cron jobs for free. Here’s how it works in this setup:

a) **Netlify Deploy Hook**:
Netlify provides a configurable webhook (a URL endpoint) that, when triggered, redeploys the site. This ensures that `chicagomusiccompass.com` can be updated automatically whenever needed.

![Netlify deploy hook](/assets/front-end-free-hosting/netlify-hook.png)

b) **cron-job.org Integration**:
With cron-job.org, you can schedule a cron job—in this case, set to run daily. The job simply triggers the Netlify deploy hook, prompting Netlify to redeploy (update) the site every day.

![daily cron-job](/assets/front-end-free-hosting/cron-job.png)

_Note_: While `chicagomusiccompass.com` also has back-end components, this post focuses solely on the front-end architecture.

## Summary

`chicagomusiccompass.com` is a Next.js application that, when built, generates a static site (no server) along with a couple of Lambda functions. The GitHub repository is integrated with Netlify, so every push to the repository triggers a new deployment. This process generates a new version of the static site and updates the Lambda functions. Netlify handles the deployment of these files and automatically provisions the necessary network infrastructure, allowing access to the web application via a subdomain. Additionally, I've configured the custom domain, `chicagomusiccompass.com`, to point to Netlify. The site is kept up-to-date by a daily cron job that triggers a Netlify deploy hook.

The site has been running for a few months and currently doesn't receive much traffic, but in terms of infrastructure costs, I’m not paying a cent.

In a professional setting, depending on the project requirements, I might choose a similar solution, especially in the early stages. Later, I could migrate certain components as the business grows and needs evolve.

Front-end architecture has become quite exciting these days, especially when you can leverage free services. However, remember that _if a service is free, you might be the product_.
