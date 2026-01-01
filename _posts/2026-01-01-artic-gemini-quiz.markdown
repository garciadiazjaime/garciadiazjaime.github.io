---
layout: post
title: "One Artwork, Seven Questions: Exploring the Art Institute API and Gemini"
date: 2025-12-22 10:00:00 -0500
categories: nextjs art gemini copilot
---

![Exploring the Art Institute API and Gemini](/assets/2026-01-01-artic-gemini-quiz/banner.png)

New year, new project. As a fan of the [Art Institute of Chicago](https://www.artic.edu/), I just realized they have a public [RESTful API](https://api.artic.edu/docs/) and decided to build a simple web app: one piece of art a day and seven questions to learn more about the artwork.

This is how I built it:

## 1. Artist list

First, I asked GPT to provide a list of famous artists found at the Art Institute of Chicago. I’ll keep the list private so as not to spoil the artists you’ll find in the app.

## 2. Artwork

Second, I hit the [/search](https://api.artic.edu/docs/#get-artworks-search) endpoint to request one artwork from _artist X_. The endpoint is part of an Elasticsearch service, meaning it offers good tooling around filters. Here’s my query:

```javascript
const query = {
  query: {
    bool: {
      must: [
        { term: { is_public_domain: true } },
        { term: { "artist_title.keyword": artist } },
        {
          match: {
            medium_display: { query: "Oil on canvas", fuzziness: "AUTO" },
          },
        },
      ],
    },
  },
};
```

There are three filters:

a) **Public domain**. This is very important to avoid copyright issues.

b) The artist’s name needs to match. There are a lot of artists and a lot of great works, but for now I’m interested in specific names.

c) I like `oil on canvas`, so I’m filtering by it, although this is more of a personal choice.

## 3. Quiz

Now that I have an artist’s name and one of their artworks (which could actually be found at the museum), the next step is to use AI to generate the questions. Here’s the prompt I’m using:

```sh
  Generate 7 quiz questions about the following artwork: ${artist} - ${artwork};
  use the following example questions as a guide: ${JSON.stringify(exampleQuestions)};
  the questions should be a mix of easy, intermediate, hard, and expert difficulty levels;
  each question should have 3 options labeled A, B, and C;
  provide the correct answer for each question;
  return the result as a JSON array of question objects with the following structure: { difficulty: string, question_number: number, question_text: string, options: { A: string, B: string, C: string }, correct_answer: string (A, B, or C) };
  only return the JSON array without any additional text;
  the questions should only be about the artwork
```

Then I pass the prompt to library `@google/genai`

````javascript
import { GoogleGenAI } from "@google/genai";
const ai = new GoogleGenAI({});
const response = await ai.models.generateContent({
  model: "gemini-2.5-flash",
  contents: prompt,
});
const cleanJson = response.text.replace(/^```json|```$/g, "").trim();
const questions = JSON.parse(cleanJson);
````

A couple of things to mention here:

a) You need a `GEMINI_API_KEY`; get one from [Google](https://aistudio.google.com/).

b) I’m using the model `gemini-2.5-flash`, but it has some rate limitations. Not bad considering it’s free, but if you need to make more requests, you can try another model like `gemma-3-12b-it`, which allows more requests. Each model is trained differently, and at least for this simple case, I didn’t notice any significant difference.

![Model Rate Limitations](/assets/2026-01-01-artic-gemini-quiz/models-limitations.png)

c) One last thing: the agent usually returns something like this:

````text
```json [
  {
    "image": "...",
    "quiz_title": "...",
    "questions": []
  }
}
```
````

So I had to remove ```json` and then parse the rest of the string, which was the array of questions I was expecting.

## 4. Web app

Finally, I’m going with **Next.js** plus **Copilot** to build the UI. I wrote a bunch of prompts, but the idea was to:

- Display the image found in the JSON
- Show one question at a time
- Keep score of correct answers
- Once the quiz is finished, show a short summary

And that’s it. Please visit the [quiz app](https://artic.mintitmedia.com/) and check out the [codebase](https://github.com/garciadiazjaime/website-artic/tree/main).

![One Artwork, Seven Questions](/assets/2026-01-01-artic-gemini-quiz/website.png)

Happy 2026 coding!
