---
layout: post
title: "How JavaScript helped me find a Car"
date: 2025-04-14 10:00:00 -0500
categories: nextjs react javascript etl
---

![How JavaScript helped me find a Car](/assets/nextjs-javascript-etl-jetta-chicago/banner.png)

I'm looking for a Jetta, and of course, I want to make sure I get the best option. Instead of manually checking car dealers' websites, I'm writing ETLs to collect car data, save it as JSON, and use a Next.js app to display the results.

## Architecture

![Architecture](/assets/nextjs-javascript-etl-jetta-chicago/architecture.png)

I'm going to write one ETL (Extract, Transform, Load) per website (source).

### Extract

This step involves opening the website and retrieving the data. Sometimes it's in HTML, sometimes in JSON.

### Transform

I'm interested in the following properties:

```
title
price
year
link
vin
mileage
```

Each website may have a different structure, but the goal is to extract these values.

### Load

For this demo, I'm keeping it simple by saving a JSON file per site. In a real-world setup, the data would be stored in a database.

### Aggregator

This script will combine all the JSON files into a single one, simulating what a database would do in a real example.

### Next.js

A basic web app to display the data in a table-like format with simple filters.

## Example

Let's take a look at the following dealer's website:

![Jetta Dealership Website](/assets/nextjs-javascript-etl-jetta-chicago/jetta-car-dealer-website.png)

Notice how the website displays the properties I'm interested in. Fortunately, at the time of writing, this site returns a JSON response, which makes it easy to traverse and extract car data. The ETL looks like this:

```js
const fs = require("fs");

const extract = async (url) => {
  const response = await fetch(url);

  return await response.json();
};

const transform = (data) => {
  return data.DisplayCards.map((VehicleCard) => {
    return {
      title: VehicleCard.VehicleName,
      price: VehicleCard.VehicleInternetPrice,
      year: VehicleCard.VehicleYear,
      link: VehicleCard.VehicleDetailUrl,
      vin: VehicleCard.VehicleVin,
      mileage: VehicleCard.Mileage,
    };
  });
};

const load = (cars, name) => {
  fs.writeFileSync(`./_sites/${name}.json`, JSON.stringify(cars));
};

const main = async () => {
  const site = {
    url: "https://www.vwoaklawn.com/api/vhcliaa/vehicle-pages/cosmos/srp/vehicles/25795/2631261?st=Price+asc&Make=Volkswagen&mileagerange=0-50000&host=www.vwoaklawn.com&baseFilter=dHlwZT0ndSc=&displayCardsShown=NaN",
  };

  const data = await extract(site.url);

  const cars = transform(data);

  load(cars, "vwoaklawn");
};
```

## Site_A.json

After running the ETL, I’ll get a JSON file like this:

```json
[
    {
        "title": "Volkswagen Jetta 1.4T SE",
        "price": 19640,
        "year": 2021,
        "link": "https://www.cityvwofchicago.com/inventory/certified-used-2021-volkswagen-jetta-1-4t-se-fwd-4d-sedan-3vwc57bu4mm003393/",
        "vin": "3VWC57BU4MM003393",
        "mileage": 27203
    },
    ...
]
```

## Aggregator

This [helper](https://github.com/garciadiazjaime/website-cars/blob/main/support/aggregator.js) combines all the `Site_N.json` files into a single `cars.json` file, which will be used by the web application.

## Next.js

A simple web app that displays `cars.json`. The page looks like this:

![Jetta Nextjs Web app](/assets/nextjs-javascript-etl-jetta-chicago/jetta-chicago-ui.png)

## Demo

Make sure to check out the [Website](https://volkswagen-chicago.mintitmedia.com/) and take a look at the [Codebase](https://github.com/garciadiazjaime/website-cars).

So now, I know which Jetta to get 🤓

Thanks for reading!

**Disclaimer: I'm not sponsored in any way—just genuinely trying to find the best 🚗.**
