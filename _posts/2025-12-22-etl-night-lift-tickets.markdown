---
layout: post
title: "Another ETL: Night Lift Tickets"
date: 2025-12-22 10:00:00 -0500
categories: nextjs react etl copilot
---

![Another ETL: Night Lift Tickets](/assets/2025-12-22-etl-night-lift-tickets/banner.png)

I live in Chicago, and one thing I like about the winter is having the chance to go snowboarding. I’m not very good at it, but I enjoy it. Usually, I would open multiple websites until I find a place to go. So every time I find myself doing a repetitive manual task—in this case, **opening multiple websites with similar content for the same purpose**—my brain goes directly to ETL.

And yes, there are already some catalog websites that show different hills around, but not the way I want. In my case, I’m interested in **night lift tickets**, because they have the best price, and I want to **see the places on a map** to find out how far they are from my place.

I wrote a couple of ETLs; they all have the same structure: **Extract, Transform, and Load**. In this case, **Extract** and **Load** are basically the same for all ETLs, which is why I put those functions in a `common.js` file, while the only custom part lives in **Transform**, which helps extract the price from the HTML using a CSS selector.

For this part, the HTML is passed to `cheerio`, which is a nice package that enables something similar to `jQuery`, making it easier to find elements in the HTML. Take a look at the following line:

```js
const price = $($(".seasoncontainer.liftcontainer.clsDaynight.clsNight li")[2]);
```

Of course, if the CSS classes change on the site, then the scraper won’t work. This is one of the challenges of using this approach in ETLs. In production, you would have an alert to help you know whenever the price can’t be extracted. Or now, you could use AI—maybe [Gemini](https://www.garciadiazjaime.com/posts/gemini-api-free-tier)—and try to extract the price using a model.

I went with a CSS selector because these sites rarely change their code.

```js
const cheerio = require("cheerio");

const { extract, load, loggerInfo } = require("./common");

async function transform(html) {
  const $ = cheerio.load(html);

  const price = $(
    $(".seasoncontainer.liftcontainer.clsDaynight.clsNight li")[2]
  )
    .find("p")
    .text()
    .replace("$", "");

  return {
    price: parseInt(price),
  };
}

async function main() {
  const place = {
    id: "cascademountain",
    website: "https://www.cascademountain.com",
    name: "Cascade Mountain",
    lat: 43.502728,
    lng: -89.515996,
    gmaps: "https://maps.app.goo.gl/YWdnQvZiJZwPhj79A",
    url: `https://www.cascademountain.com/lift-tickets/`,
  };
  loggerInfo("etl start", { id: place.id });

  const html = await extract(place.url);
  const data = await transform(html);
  await load(place, data);

  loggerInfo("etl done", { id: place.id });
}

main().then(() => {});
```

The only dynamic property is `price`; the others are hardcoded. The thought is that prices change over time, but a business is either running or closed. The hardcoded values could be extracted using, maybe, Google Maps APIs, but considering that the number of places is limited, hardcoding them for now is fine. If my site gets real traffic, then I would consider writing a script to use Google Maps APIs and programmatically get a more comprehensive list of ski places.

The `extract` function simply makes a `fetch` request to the URL passed. There’s a variation in the codebase that uses [puppeteer](https://github.com/garciadiazjaime/website-ski/blob/main/etl/common.js#L16), because some sites don’t like scrapers.

```js
async function extract(url) {
  loggerInfo("extracting", { url });

  const response = await fetch(url);
  const html = await response.text();
  return html;
}
```

The `load` function only saves the data in a `json` file. For now, a file is enough. At some point, this should be a database; I would go with **DynamoDB** because it’s cheap.

```js
async function load(place, extraData) {
  const data = { ...place, ...extraData };
  loggerInfo("load", data);

  const filename = `public/sites/${data.id}.json`;
  await fs.writeFile(filename, JSON.stringify(data, null, 2));

  loggerInfo("saved", { filename });
}
```

`loggerInfo` is nothing more than a wrapper around `console.log`, which for now is enough. But in a production-like application, you could plug in something like **New Relic** or any other logging service.

```js
const loggerInfo = (...args) => {
  console.log(...args);
};
```

Alright, so that’s the ETL. Now that I have the information, the rest is to build a website to show the places. I usually go with **Next.js** and then use **Copilot** to build it, so the prompt looks like this:

```
On this Next.js project, build a page with the following requirements:
- Only one page.
- The page should show a map using Google Maps.
- The map should show a marker for each place found in `all-places.json`, displaying the price.
- The page should have Google Analytics.
- The page should let the user click a marker and open a small card with the place information: name, link to Google Maps, and link to the `url` found in the JSON.
- The UI should be simple and use inline styles.
```

Something like that is a good starter. Of course, after that there would be some adjustments—some will be taken care of by it, and some by them.

And that’s it. Please visit the site and let me know what you think:

- [goofy](https://goofy.mintitmedia.com/)

![Another ETL: Night Lift Tickets](/assets/2025-12-22-etl-night-lift-tickets/website.png)

- [codebase](https://github.com/garciadiazjaime/website-ski/tree/main)

Happy coding.

P.S. Yes, I’m goofy.
