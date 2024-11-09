---
layout: post
title: "Gemini API: The Free Tier That Makes Developers Happy"
date: 2024-10-14 10:00:00 -0500
categories: javascript programming gemini openai
---

![Gemini API: The Free Tier That Makes Developers Happy](/assets/gemini-api-free-tier/banner.png)

GPT rocks, but it no longer offers an API with a free tier—at least not anymore. Luckily, Google does, with `Gemini API` and `Studio AI` (Google's version of ChatGPT).

At the time of writing, here's what the `Gemini API` free tier offers:

![Gemini API Free Tier Quotas](/assets/gemini-api-free-tier/gemini-api-free-tier-qoutas.png)

As you can see, that's more than enough to start playing around with the API, so there's really no excuse not to integrate it into our projects.

Their [Quick Start](https://ai.google.dev/gemini-api/docs/quickstart) guide is super straightforward.

In my case, I'm using Gemini for a pretty pointless task, but the goal is to show just how easy it is to integrate their API.

Check out the following snippet, which generates "funny quotes" based on a popularity number.

```javascript
const { GoogleGenerativeAI } = require("@google/generative-ai");

const genAI = new GoogleGenerativeAI(process.env.GEMINI_API_KEY);
const model = genAI.getGenerativeModel({ model: "gemini-1.5-flash" });

const popularity = 42;

const prompt = `Write a funny quote, under 200 characters, about popularity on a scale from 0 to 100, where 0 is the least popular and 100 is the most. The quote should describe someone at ${popularity}.`;

const result = await model.generateContent(prompt);

console.log(result);
```

Output:

> He's a solid 42 on the popularity scale. Not quite 'cool,' but definitely not 'that guy nobody talks to.'

That's it! A couple of things to note:

- Notice how an `API_KEY` environment variable is required. You can grab this from AI Studio once your account is set up.
- The model selected is `gemini-1.5-flash`. Studio AI offers multiple models, so you can experiment with different ones to find the best fit.
- The rest is just regular JavaScript, passing a prompt to the `generateContent` method and logging the result.

## Demo

In my previous post, [TensorFlow: From Python to JavaScript](https://www.garciadiazjaime.com/posts/tensorflow-python-javascript), I shared a demo that predicts the [popularity of a Twitter account](https://demo.garciadiazjaime.com/tensorflow-load-model). Feel free to check it out, and if you click "Tweet my result" it'll generate a tweet using the snippet above.

- UI:

  ![Demo: Predicting Popularity](/assets/gemini-api-free-tier/demo-predict-popularity.png)

- Tweet:

  ![Tweeting a Funny Quote about Predicting Popularity](/assets/gemini-api-free-tier/demo-tweet-popularity.png)

You can find the source code [here](https://github.com/garciadiazjaime/demo-reactjs/blob/main/netlify/functions/joke.ts)

## Get Code Feature in Studio AI

Additionally, Google offers Studio AI, which is similar to ChatGPT but with an interesting feature: `Get Code`. You can enter a prompt, and if it's what you want for your service, just click the button, and it gives you the code you need to run the same prompt from your own code.

- Command:

![Prompting a Command on Studio AI](/assets/gemini-api-free-tier/ai-studio-command.png)

- Get Code:

![Getting the Code from Studio AI](/assets/gemini-api-free-tier/ai-studio-get-code.png)

## Choosing Between GPT and Gemini for Your Project

I'm a huge fan of ChatGPT, but as a developer, I've found Studio AI's free tier super useful. For experimentation, OpenAI isn't too expensive, but nothing beats a free tier. Both have solid documentation.

In my humble opinion, `GPT` still gives better answers than `Gemini`, but in the early stages of a project, I'd go with Gemini and switch to GPT when accuracy becomes critical and the investment makes sense.

In the meantime, I'd follow a proxy pattern, so if switching becomes necessary, it's an easy task.

Let's take a look at the following snippet:

```javascript
const { GoogleGenerativeAI } = require("@google/generative-ai");

async function getJokeFromGenerativeAI() {
  const genAI = new GoogleGenerativeAI(process.env.GEMINI_API_KEY);
  const model = genAI.getGenerativeModel({ model: "gemini-1.5-flash" });

  const popularity = 42;

  const prompt = `Write a funny quote, under 200 characters, about popularity on a scale from 0 to 100, where 0 is the least popular and 100 is the most. The quote should describe someone at ${popularity}.`;

  const result = await model.generateContent(prompt);
  return result;
}

// proxy
async function getJoke() {
  const joke = await getJokeFromGenerativeAI();

  return joke;
}
```

This way, consumers can call `getJoke()` without worrying about what's happening under the hood. As time passes, and let's say `GPT` is needed, the change becomes simple:

```javascript
const OpenAI = require("openai");

async function getJokeFromOpenAI() {
  const openai = new OpenAI();

  const popularity = 42;

  const prompt = `Write a funny quote, under 200 characters, about popularity on a scale from 0 to 100, where 0 is the least popular and 100 is the most. The quote should describe someone at ${popularity}.`;

  const completion = await openai.chat.completions.create({
    model: "gpt-4o",
    messages: [{ role: "user", content: prompt }],
  });

  return completion.choices[0].message;
}

// proxy
async function getJoke() {
  const joke = await getJokeFromOpenAI();

  return joke;
}
```

Notice how `getJoke` now calls the new method: `getJokeFromOpenAI`. Since both methods follow the same contract—**they both return a Promise that resolves into a string**—the consumers of `getJoke` won't notice the change and don't need to update anything.

[OpenAI Docs](https://platform.openai.com/docs/overview)

## Conclusion

Every day, more applications are integrating AI, to the point where users are starting to expect it, just like they expect fast and user-friendly websites. As a developer, it's important to know the options out there: [Custom Model](https://www.garciadiazjaime.com/posts/tensorflow-python-javascript), [Open Source Model](https://www.garciadiazjaime.com/posts/named-entity-recognition), and Private Model, and leverage them to our advantage. Who knows what's next, but whatever it is, it will definitely be built on the shoulders of AI.
