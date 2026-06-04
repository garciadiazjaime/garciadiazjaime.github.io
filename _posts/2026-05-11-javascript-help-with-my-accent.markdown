---
layout: post
title: "Javascript help me with my accent"
date: 2026-05-11 10:00:00 -0500
categories: ai javascript llm-models
---

![Javascript help me with my accent](/assets/2026-05-11-javascript-help-with-my-accent/banner.png)

I built the site [Mati](https://mati.mintitmedia.com/) which shows one sentence a day; the goal is to repeat the sentence and get feedback.

![Mati website](/assets/2026-05-11-javascript-help-with-my-accent/mati.png)

## TLDR

Cloud Text-to-Speech offers natural voices, way better than the boring robotic voices browsers have by default. Google offers a free tier which is enough for experimenting with the service.

Speech-to-Text while not new, now most browsers add an LLM layer which tries to understand what was said, making for better transcripts. However, if you are looking for feedback on your pronunciation, you can use a **speech model** (`wav2vec2` or `whisper`) only for the transcription part, without the guessing layer.

## Back End

There's a daily cron that runs two functions to generate a sentence and get its corresponding audio. Then the site is deployed using the new sentence.

![Backend Architecture Components](/assets/2026-05-11-javascript-help-with-my-accent/backend-architecture-components.jpeg)

### Sentence Generation

I'm using the LLM `gemma-4-31b-it` to generate a random text every day:

```javascript
import { GoogleGenAI } from "@google/genai";

const ai = new GoogleGenAI({ apiKey: GEMINI_API_KEY });

async function generateSentence() {
  const SYSTEM_PROMPT_TEACHER = `You are a helpful and precise assistant for language learning.
  You create one everyday sentence and aligned translations in American English.`;

  const USER_PROMPT = `Generate one sentence for today's speaking practice.
    Respond with ONLY this JSON object and nothing else:
    {"en":"..."}

    Do not explain. Do not reason. Do not check your work. Just output the JSON.

    Rules:
    - en: American English.
    - Natural, spoken everyday language — not textbook phrasing.
    - Interesting and pleasant to say out loud — varied sounds, natural rhythm.
    - 10 to 14 words in the English version.
    - No extra keys. No markdown.`;

  const payload = {
    model: "gemma-4-31b-it",
    contents: [
      { role: "user", parts: [{ text: SYSTEM_PROMPT_TEACHER }] },
      { role: "model", parts: [{ text: "Understood." }] },
      { role: "user", parts: [{ text: USER_PROMPT }] },
    ],
  };

  const response = await ai.models.generateContent(payload);

  const text = response.candidates?.[0].content?.parts?.slice(-1)[0]?.text;

  return text;
}
```

### Audio Generation

Then the sentence is converted into **audio base64**:

```javascript
const apiKey = process.env.GOOGLE_CLOUD_TTS_API_KEY;

async function synthesizeGoogleTTS(text) {
  const response = await fetch(
    `https://texttospeech.googleapis.com/v1beta1/text:synthesize?key=${apiKey}`,
    {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        input: {
          ssml: buildSSML(text),
        },
        voice: {
          languageCode: "en-US",
          name: "en-US-Neural2-F",
        },
        audioConfig: {
          audioEncoding: "MP3",
        },
        enableTimePointing: ["SSML_MARK"],
      }),
    }
  );

  const payload = await response.json();

  return {
    audioBase64: payload.audioContent,
  };
}
```

> Browsers offer Text-To-Speech by default, but each browser has different voices. By using this Google service, the voice sounds more natural (higher quality) and is the same across all browsers.

## Front End

When the user loads the page, the HTML is served from a CDN and the client downloads the audio base64, so when the user clicks "listen", the audio provided by the Google Text-to-Speech service is played. The audio sounds like a natural voice.

![Frontend Architecture Components](/assets/2026-05-11-javascript-help-with-my-accent/frontend-architecture-components.jpeg)

### Transcript

The user clicks "record" and here either the user is on Mobile Safari or not.

- SpeechRecognition (Mobile Safari)

If the user is on Mobile Safari (apologies to other mobile users, since I didn't test on other mobile browsers), the native API `webkitSpeechRecognition` is used, which does Speech-to-Text.

> At the time of writing, mobile browsers didn't support large files (i.e., Transformers) and WebAssembly as well as desktop browsers, so trying to run `Whisper` on a mobile browser threw some errors.

```javascript
async function start(onSuccess) {
  const recognition = new speechWindow.webkitSpeechRecognition();

  recognition.onresult = (event) => {
    const transcript = Array.from(
      { length: event.results.length },
      (_, i) => event.results[i][0].transcript
    ).join(" ");

    onSuccess(transcript);
  };

  recognition.start();
}
```

- Whisper

If the user is not on Mobile Safari then `Whisper` is used, specifically a light version which **provides a rawer transcript useful for pronunciation feedback**.

```javascript
// worker
async function transcribe(audio) {
  const automaticSpeechRecognition = pipeline(
    "automatic-speech-recognition",
    "Xenova/whisper-tiny",
    {
      dtype: {
        encoder_model: "q4",
        decoder_model_merged: "q4",
      },
    }
  );

  const output = await automaticSpeechRecognition(audio, {
    task: "transcribe",
    language: "english",
    return_timestamps: false,
    temperature: 0,
    no_repeat_ngram_size: 3,
    repetition_penalty: 1.15,
    condition_on_prev_tokens: false,
    max_new_tokens: 96,
  });

  const transcript = output.text;

  post({ type: "result", transcript });
}

// client
async function start(onSuccess) {
  const mediaStream = await navigator.mediaDevices.getUserMedia();
  const mediaRecorder = new MediaRecorder(mediaStream);
  const recordedChunks = [];

  mediaRecorder.ondataavailable = (event) => {
    recordedChunks.push(event.data);
  };

  mediaRecorder.onstop = async () => {
    const blob = new Blob(recordedChunks, { type: recordedChunks[0].type });
    const audio = await blobToMonoPcm(blob, 16_000);
    const activeWorker = getWorker();
    const transferableAudio = new Float32Array(audio);

    const transcript = await new Promise((resolve, reject) => {
      const onMessage = (event) => {
        if (event.data.type === "result") {
          resolve(event.data.transcript);
          return;
        }
      };

      activeWorker.addEventListener("message", onMessage);
      activeWorker.postMessage(
        {
          type: "transcribe",
          audio: transferableAudio,
        },
        [transferableAudio.buffer]
      );
    });

    onSuccess(transcript);
  };

  mediaRecorder.start();
}
```

The final solution is a bit more complicated, but that's the gist. In Mobile Safari, the implementation is simpler because the heavy lifting is done by `webkitSpeechRecognition`, which has a neat `onresult` API that returns the transcript. The caveat here is that Safari will try to "guess" the transcript.

For all other users, `Whisper` is used. Notice that the task is split between traditional client code and a web worker. The client code captures the audio and passes it to the web worker, which converts the audio to text using a **Small Language Model**; this is a key part to avoid the guessing layer.

### Feedback

Once the transcript is generated, it is compared against the original phrase using the following metrics:

- **Levenshtein distance**: edit distance (e.g., apple -> aple)
- **Jaro-Winkler**: similarity score (e.g., Martha -> Sartha)
- **Metaphone**: reduces a word down to a short code (e.g., Stephen -> STFN)

Each metric provides a score which the app uses to determine whether the user pronounced one of the words in the phrase, using different thresholds.

```json
// Attempt 1 — strict
{
  "levenshteinThreshold": 0.72,
  "jaroWinklerThreshold": 0.88
}
```

```javascript
function getScores(referenceWord, attemptWord) {
  const levenshteinScore = getLevenshteinSimilarity(referenceWord, attemptWord);
  const jaroWinklerScore = jaroWinkler(referenceWord, attemptWord);
  const metaphoneScore = Metaphone.compare(referenceWord, attemptWord);

  return {
    levenshteinScore,
    jaroWinklerScore,
    metaphoneScore,
  };
}
```

## Conclusion

At some point, I tried comparing audio waves, but there are many variables that make it hard: noise, pace, rhythm, etc. I also wanted all the processing in the browser, which is especially hard for mobile in terms of computing resources (RAM, CPU, GPU). Finally, I pivoted to transcripts and the metrics mentioned.

Native APIs lose a lot of pronunciation feedback during transcript processing. Maybe browsers will enable a flag to provide raw transcripts; I doubt it. On the other hand, **I'm sure mobile browsers will eventually have more resources to run LLMs on the client which could provide audio feedback**.

Raw transcripts are definitely good for feedback, but they are a 0-1 metric: you either say it or not. Even with an accent, you can pass. But in terms of accent reduction, better feedback is needed, like identifying which parts of the word need more work, which letter wasn't pronounced, or what movement the mouth and tongue need to make in order to produce a natural sound. While this is a good start and I'm glad Javascript is helping me once again, there is still more that can be done. More to come in the future for sure.
