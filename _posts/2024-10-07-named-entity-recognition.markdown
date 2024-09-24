---
layout: post
title: "ETL: Extracting a Person's Name from Text"
date: 2024-09-23 10:00:00 -0500
categories: python machinelearning named-entity-recognition programming
---

![ETL: Extracting a Person's Name from Text](/assets/tensorflow-python-javascript/banner.png)

Let's say we want to scrape **[chicagomusiccompass.com](https://www.chicagomusiccompass.com/)**.

As you can see, it has several cards, each representing an event. Now, let's check out the next one:

![Name of a Music Event](/assets/named-entity-recognition/chicagomusiccompass.png)

Notice that the name of the event is:

```
jazmin bean: the traumatic livelihood tour
```

So now the question is: **How do we extract the artist's name from the text?**

As a human, I can "easily" tell that `jazmin bean` is the artist—just check out their [wiki page](https://en.wikipedia.org/wiki/Jazmin_Bean). But writing code to extract that name can get tricky.

We could think, "Hey, anything before the `:` should be the artist's name," which seems clever, right? It works for this case, but what about this one:

```
happy hour on the patio: kathryn & chris
```

Here, the order is flipped. We could keep adding logic to handle different cases, but soon we'll end up with a ton of rules that are fragile and probably won't cover everything.

That’s where **Named Entity Recognition (NER)** models come in handy. They’re open source and can help us extract names from text. It won’t catch every case, but most of the time, they’ll get us the info we need.

With this approach, the extraction becomes way easier. I'm going with `Python` because the community around Machine Learning in Python is just unbeatable.

```python
from gliner import GLiNER

model = GLiNER.from_pretrained("urchade/gliner_base")

text = "jazmin bean: the traumatic livelihood tour"
labels = ["person", "bands", "projects"]
entities = model.predict_entities(text, labels)

for entity in entities:
    print(entity["text"], "=>", entity["label"])
```

Which generates the output:

```sh
jazmin bean => person
```

Now, let’s take a look at that other case:

```
happy hour on the patio: kathryn & chris
```

Output:

```sh
kathryn => person
chris => person
```

[source-GLiNER](https://github.com/urchade/GLiNER)

Awesome, right? No more tedious logic to extract names, just use a model. Sure, it won’t cover every possible case, but for my project, this level of flexibility works just fine. If you need more accuracy, you can always:

- Try a different model
- Contribute to the existing model
- Fork the project and tweak it to fit your needs

## Conclusion

As a Software Developer, it's highly recommended to stay updated with the tools in the Machine Learning space. Not everything can be solved with just plain programming and logic—some challenges are better tackled using models and statistics.
