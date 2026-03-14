---
layout: post
title: "NER: Gemini vs Spacy vs Compromise"
date: 2026-03-16 10:00:00 -0500
categories: machine-learning language-model large-language-model named-entity-recognition
---

![NER: Gemini vs Spacy vs Compromise](/assets/2026-03-07-ner-gemini-spacy-compromise/banner.png)

## TLDR

For **NER**, if accuracy is critical, go with an LLM — even an old one like [gemma-3-27b-it](https://huggingface.co/google/gemma-3-27b-it) will outperform tools or small models trained for this task. But by using an LLM you are exposing your data, making an HTTP request, and most likely incurring a cost. If accuracy is not critical and you want to stay in Javascript, [compromise](https://www.npmjs.com/package/compromise) is a good package for NER. If you want an even better package and it's OK not using Javascript, then try [Spacy](https://spacy.io/).

## Intro

Thanks to AI it feels like we are entering into Web 4 — now it's not just about having a static website (Web 1), or letting the user save their data (Web 2), or decentralization (Web 3, although the majority of companies still own our data); but about adding "AI" to your application. What does that mean? Are all applications supposed to be like ChatGPT?

While I don't think adding an LLM just for the sake of it is beneficial, I do think that playing with LLMs helps to learn about them, giving us an understanding of how, when, and why to use them.

I maintain the website [chicagomusiccompass](https://www.chicagomusiccompass.com/), which shows music events in Chicago. All events have a title, so I want to extract artist names from the title. This sounds like a good task for an LLM, right?

I found out about **Named Entity Recognition** (NER), which is a technique to extract entities from a string. So I'll try three different NER tools and compare their results.

## Gemini

As a developer, Gemini has become my first option mainly because they offer almost-free LLM tiers. They might not be the newest, but they're a great starting point. As a developer could go with OpenAI or Anthropic, but they don't offer free tiers. Ollama is a truly free alternative, but this time I'll stay with Gemini.

NER with Gemini is about the prompt:

```python
from google import genai

client = genai.Client(api_key="your_api_key")
title = "Amber Mark Presents: The Pretty Idea Tour"

PROMPT_TEMPLATE = """Extract the person's name from this event title.
Return ONLY the name, nothing else. If no person found, return empty string.

Examples:
"Saxophonist Jarrard Harris Quintet" → "Jarrard Harris"
"Jazz Night at the Blue Room" → ""
"DJ Khaled Live Concert" → "DJ Khaled"
"Annual Wine Festival" → ""

Title: "{title}"
→"""

artist = client.models.generate_content(
    model="models/gemma-3-27b-it",
    contents=PROMPT_TEMPLATE.format(title=title)
)
artist = artist.text.strip().strip('"')

print(artist)
```

Output:

```
Amber Mark
```

I'm using the model **gemma-3-27b-it** (Google hosts Gemma models on the Gemini API platform), which was released in early 2025. There are newer models, but this one offers a decent number of free requests.

![Gemini Models Free Tiers](/assets/2026-03-07-ner-gemini-spacy-compromise/gemini-models-free-tiers.png)

Notice that here I'm using an LLM to perform NER through the prompt. An LLM is trained for multiple purposes, and NER is one thing that LLMs are good at.

## Spacy

Spacy is a Natural Language Processing (NLP) library, and one of the things it's trained for is NER. It's also free and open-source — kudos to the team behind it. Gemini is proprietary and accessed through an API with rate limiting and commercial terms; at some point, if you start using it heavily, you will need to pay. Not so with Spacy, which offers a Python package you can install in your project and use freely.

```python
import spacy
nlp = spacy.load("en_core_web_sm") # language model
title = "Amber Mark Presents: The Pretty Idea Tour"
doc = nlp(title)
ent = next(filter(lambda ent: ent.label_ == "PERSON", doc.ents), None)
artist = ent.text

print(artist)
```

Output:

```
Mark Presents
```

Note: `next(filter(lambda ent: ent.label_ == "PERSON", doc.ents), None)` is the Python way to get the first entry where `label_` is `"PERSON"`.

As you can see, the output is similar to Gemini's — but is it good? That depends on your project. The upside is that processing happens locally, with no HTTP requests, meaning your data is not exposed and there's no extra cost.

## compromise

I tried three npm packages (compromise, @nlpjs/ner, and node-nlp) and out of the three I found that `compromise` provides results similar to Spacy. The Python community is larger in terms of data science packages; however, for a Javascript architecture (a website), it makes sense to go with an npm package.

```javascript
const compromise = require("compromise");

const title = "Amber Mark Presents: The Pretty Idea Tour";
const doc = compromise(title);
const people = doc.people().out("array");
const artist = people && people.length > 0 ? people[0] : "";

console.log(artist);
```

Output:

```
Amber Mark Presents:
```

As you can see, the output includes ":". This is because compromise doesn't actually understand the text — it just runs a set of rules and uses weights to decide what counts as a person. Since "Presents" can be a last name, it gets included as part of the person's name, and the colon comes along for the ride.

## Conclusion

I ran the snippets against a list of 1,000 records. Gemini was able to extract a name 66% of the time, Spacy 36%, and compromise 45%. Gemini, even with an old model, not only covered more records but also provided cleaner results. Spacy and compromise provided similar results, and even though Spacy covered fewer records, in some cases the output was cleaner — for instance, `Mark Presents` vs. `Amber Mark Presents:`. In both cases it's actually wrong, compared to Gemini's output of `Amber Mark`.

The older LLM (`gemma-3-27b-it`) outperformed the smaller models (`Spacy` and `compromise`) even though they were trained specifically for this task.

For my purposes, `compromise` is more than enough. I like staying in Javascript and NER extraction accuracy is not critical for me. If accuracy were a factor, I would go with an LLM.

Finally, all I did was use NER to adjust the styling on the title:

![NER drives title styles](/assets/2026-03-07-ner-gemini-spacy-compromise/music_event_title_ner.png)
