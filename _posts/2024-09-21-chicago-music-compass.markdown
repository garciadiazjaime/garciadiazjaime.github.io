---
layout: post
title: "Chicago Music Compass: From Paper to Production (and Yes, AI Too)"
date: 2024-09-28 10:00:00 -0500
categories: javascript webdev productivity ai
---

![Front-End Architecture: How to Host Your Web App for Free](/assets/front-end-free-hosting/banner.png)

## Chicago Music Compass

A web application designed to help users find [Chicago music concerts](https://www.chicagomusiccompass.com/).

## Motivation

I've been living in Chicago since 2014, and one of the things I love most is the music scene.

The `Chicago Music Commission` did a study called [A Summary Report On The Music Industry In Chicago](https://www.riaa.com/reports/a-summary-report-on-the-music-industry-in-chicago/), and one of the key takeaways was that Chicago ranked fourth among all U.S. cities for the number of concerts and performances back in 2004.

This just shows how much live music data the city generates.

Today, I want to share my journey with you and the different tools I've used along the way.

## Insights

Whenever I start a project, I always check out Google Trends and play around with a few queries. It’s a great way to get a feel for what people are actually searching for.

For example, in this project, I’m interested in the keywords: `Chicago Concerts`. So, let’s take a look at the search trends over the **last 7 days**.

_Note: Charts generated on Sep. 08, 2024_

### Chicago Concerts vs. Events: A Search Trend Comparison

![Search Trends: Chicago Concerts vs. Chicago Events](/assets/chicago-music-compass/google-trends-chicago.png)

As we can see, there's more interest in "Events" than "Concerts," which makes sense since "Concerts" are just a subset of "Events." However, the difference isn’t huge, so we can infer that "Concerts" make up a significant chunk of those "Events" searches. In other words, when people search for events, there's a good chance they're specifically looking for concerts.

### Popular Related Searches for Chicago Concerts

![Top Queries for Chicago Concerts](/assets/chicago-music-compass/google-trends-queries-concerts.png)

There are some interesting insights in the "Related queries." For example, "Space Evanston" popped up. It’s a venue that hosted a folk music festival that week, so it makes sense that people were searching for that festival.

### Popular Related Searches for Chicago Events

![Top Queries for Chicago Events](/assets/chicago-music-compass/google-trends-queries-events.png)

Speaking of events, that week featured the "Greek Fest," so it makes sense that "Taste of Greektown" shows up as a top related query. Even though this event isn’t specifically about music, there are still some local bands performing.

[source](https://trends.google.com/trends/explore?date=now%207-d&geo=US&q=chicago%20concerts,chicago%20events&hl=en)

### Music near me

![Music near me Queries](/assets/chicago-music-compass/google-trend-music-near-me.png)

You’ll notice that "music near me" has a lot more search volume than the previous two keywords. Since it’s not location-specific, that makes sense. Still, it’s useful for gathering insights into user search patterns.

[source](https://trends.google.com/trends/explore?date=now%201-d&geo=US&q=chicago%20concerts,chicago%20events,music%20near%20me&hl=en)

---

Google Trends is a super handy tool that shows you the relevant keywords for a specific topic.

Output:

- Define your target keywords; in my case, it’s `Chicago Concerts`.

## Domain

Once I’ve figured out the keyword direction, the next step is to ask ChatGPT for some domain recommendations.

![Domain Suggestions for Chicago Music Compass](/assets/chicago-music-compass/domain-suggestions.png)

Output:

- Grab the domain and upload at least a splash page with some content. This way, Google can start getting a feel for your website.

## Implementation

In general, I like to follow this approach: **Small Iterations**, **Continuous Delivery**, **Quick Feedback**, and then repeat.

### Small Iterations

Start by defining a small task that can improve the site based on its current stage, and aim to deliver it within a week. If it looks like it’ll take longer than that, break it down into smaller pieces. The goal is to have achievable deliveries. If a feature is going to take more than a week, consider whether it’s really needed at this point.

### Coding

Make the most of tools like GitHub Copilot. If you’re a student, they offer a free account, and if not, think about it as a worthwhile investment. I’m not usually a fan of paying for services, and most of the help Copilot provides can be found with other AI tools like GPT. But the integration with your IDE is super handy—it makes you way more efficient. Instead of constantly switching between windows to get answers, you can get help right where you’re writing code.

![copilot example](/assets/chicago-music-compass/coding/copilot.png)

![Getting GPT's Help for Setting Up Unit Tests](/assets/chicago-music-compass/coding/gtp-setup-unittests.png)

GitHub Copilot combined with GPT is a fantastic combo for tackling most of your coding challenges. Use these tools to get a better grasp of the issues and their solutions. They’re super helpful, but the more you understand what’s going on, the better prepared you'll be for the next challenge!

![Developer Memes: GPT Edition](/assets/chicago-music-compass/coding/meme-ai-stack-docs.png)

### Monitoring

**Continues Delivery:** Connect your GitHub repository to automated services like Vercel or Netlify. This way, every commit triggers an automatic deployment, making your workflow smooth and efficient!

![Integrating with Netlify](/assets/chicago-music-compass/coding/netlify.png)

**Quick Feedback:** This one’s a bit trickier, but tools like Google Analytics, Search Console, and BetterStack can really help. Plus, don’t hesitate to share your project on Reddit or similar platforms—there’s definitely a community out there that can help you validate your ideas and proposals!

![Using Google Analytics](/assets/chicago-music-compass/coding/google-analytics.png)

![Exploring Google Search Console](/assets/chicago-music-compass/coding/search-console.png)

![Logs in BetterStack](/assets/chicago-music-compass/coding/betterstack.png)

![Engaging with the Community](/assets/chicago-music-compass/coding/reddit.png)

> Check out this post if you want to learn more about [how Chicago Music Compass was built](https://www.chicagomusiccompass.com/blog/how-cmc-was-built)!

Output:

- A web application that’s integrated with Continuous Delivery tools and monitoring services.

## Conclusion

![Framework: From Paper to Production](/assets/chicago-music-compass/framework.png)

- Find what motivates you
- Back up your instincts
- Solve a problem
- Monitor and ask for user feedback
