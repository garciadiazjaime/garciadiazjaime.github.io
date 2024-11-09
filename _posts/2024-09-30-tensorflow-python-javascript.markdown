---
layout: post
title: "Bridging Machine Learning with TensorFlow: From Python to JavaScript"
date: 2024-09-30 10:00:00 -0500
categories: javascript python machinelearning tensorflow
---

![Bringing Machine Learning to Life with TensorFlow](/assets/tensorflow-python-javascript/banner.png)

## Bringing Machine Learning to Life with TensorFlow

As a JavaScript developer, diving into Machine Learning isn’t as daunting as it might seem. While it’s technically possible to handle everything with Node.js packages, the Python ML ecosystem is just too rich and well-established to ignore. Plus, Python is a blast to work with. So, it makes sense to use Python for the heavy lifting on the backend. Once you’ve got your model ready, you can export it to a front-end friendly format and load it on the client to run predictions.

## Generating a Model

In this post, we’re going to build a model to predict an artist's popularity based on their number of Twitter followers.

The first step is to get our hands on a dataset. For this project, we’ll be using a [artists.csv](https://demo.garciadiazjaime.com/tensorflow-load-model/artists.csv) file that looks like this:

```csv
twitter_followers,popularity,handle
111024636,94,justinbieber
107920365,91,rihanna
106599902,89,katyperry
95307659,97,taylorswift13
66325495,87,selenagomez
66325135,71,selenagomez
60943147,83,jtimberlake
54815915,82,britneyspears
53569307,85,shakira
```

As you can see, there are two key values here: `twitter_followers` and `popularity`. This sets us up nicely for a Sequence Model, where `x` will be `twitter_followers` and `y` will be `popularity`.

**Sequence Model** is one of the easiest options for building a model. While the choice ultimately depends on the specific use case, I’m keeping it simple and sticking with this approach for now.

## Building the Backend

When you're building a model, there are some basic tasks you’ll need to tackle:

- Clean or normalize the data
- Split the data into Training (80%) and Testing (20%)
- Choose a model along with settings like optimizer and loss function
- Train the model (fit)
- Evaluate the model
- Save the model

The following code gives you a good overview of these tasks, though it’s not the complete picture. You can check out the full code on [Github](https://github.com/garciadiazjaime/django-models/blob/main/event/management/commands/artist_popularity_model.py#L111).

### Python: Getting Started with TensorFlow

```python
def get_model(x, y):
    x_normalized = layers.Normalization(
        axis=None,
    )
    x_normalized.adapt(np.array(x))

    model = tensorflow.keras.Sequential([x_normalized, layers.Dense(units=1)])

    model.compile(
        optimizer=tensorflow.keras.optimizers.Adam(learning_rate=0.1),
        loss="mean_squared_error",
    )

    model.fit(
        x,
        y,
        epochs=2,
        verbose=0,
        validation_split=0.2,
    )

    return model

def main:
  train_features, test_features, train_labels, test_labels = split_data(dataset)

  model = get_model(
      train_features["twitter_followers"],
      train_labels,
  )

  test_loss = model.evaluate(
      test_features["twitter_followers"], test_labels, verbose=2
  )

  model.export("./saved_model")
```

As you can see, the Python code is pretty straightforward. There's a `main` function that handles splitting the data, getting the model, evaluating it, and finally saving it.

In a nutshell, these are the essential steps to create a model. But let’s be real: building a model that actually works is both an art and a science. My goal here is just to show how easy it can be to get started with Python. However, a lot goes into making a model that performs well—like having a solid dataset, cleaning and normalizing your data, picking the right model and settings, and having the computing power to train it. All these tasks require a serious investment of time and effort!

## Consuming the Model on the Frontend

Now that we've got our model trained and saved, it’s time to bring it into the frontend. This step is where we’ll load the model in a web-friendly format, so we can run predictions right in the browser. Whether you're using TensorFlow.js or another library, integrating machine learning into your web app opens up a world of possibilities. Let’s dive into how to do that!

TensorFlow offers an npm package called `tensorflowjs_converter` that helps convert saved models into JSON and binary.

```sh
tensorflowjs_converter --input_format=tf_saved_model model/saved_model out/public
```

- `tf_saved_model`: This is the format used to save the model.
- `model/saved_model`: This is the directory where the model was saved when the Python code was executed.
- `out/public`: This is the output directory where the frontend-friendly files are saved. The folder structure will look like this:

```
ls -la out/public

group1-shard1of1.bin
model.json
```

This setup makes it easy to access the necessary files for your web application.

### Javascript: Using TensorFlowJS

You can check out the full code on [Github](https://github.com/garciadiazjaime/demo-reactjs/blob/main/app/tensorflow-load-model/page.tsx).

```javascript
const model = await tensorflow.loadGraphModel("model.json");

const getPopularity = (followers) => {
  const followers = 1_000;
  const normalized = followers;
  const x = tensorflow.tensor(normalized).reshape([-1, 1]);

  const result = model.predict(x);
  const values = result.arraySync();

  const y = values[0][0].toFixed(2) * 100;
  const popularity = y;

  return popularity;
};
```

As mentioned earlier, this model aims to "predict popularity" based on the number of Twitter followers. While it might seem like a simple example, it effectively demonstrates how to generate a model on the backend and consume it on the frontend.

Take a look at how `getPopularity` processes the input a bit, but the key line is `model.predict(x)`, which uses the model to predict a value (`y`) based on the input `x`.

Head over to the [demo page](https://demo.garciadiazjaime.com/tensorflow-load-model) and try out a few Twitter handles. It’s a fun way to see how the model predicts popularity based on follower count.

## Conclusion

TensorFlow is an awesome library that provides tools for both backend and frontend development. Any JavaScript developer can dive into creating a model using Python or a similar language, then easily import that model into the frontend for running predictions.

While machine learning is a vast field that requires a lot of knowledge, tools like TensorFlow help bridge the gap between software and machine learning developers. It makes the journey a lot smoother for those looking to incorporate ML into their projects!
