---
layout: post
title: "How to Connect Your Next.js React Application to Redis"
date: 2025-04-14 10:00:00 -0500
categories: nextjs react redis javascript
---

![How to Connect Your Next.js React Application to Redis](/assets/react-server-action-redis/banner.png)

Recently, I had to add Redis to a React application running on Next.js using server actions. Here are the important pieces.

1. Select an npm package

I went with [ioredis](https://www.npmjs.com/package/ioredis). At the time of writing, it had 7 million downloads, and after opening their repo, the last commit was from two days ago. This means the project is active.

```sh
npm i ioredis
```

2. Initialize a redis client

```javascript
const redis = new Redis(process.env.REDIS_CONNECTION_STRING);
```

If another team takes care of the infrastructure they will provide a "connection string", that looks something like this:

```
rediss://[user]:[password]@[host]:[port]
```

For this demo I'm using [Aiven](https://aiven.io/), they offer a 1GB free account, which is more than enough for a POC or prototype.

3. Read from Redis

In the previous step, if all went well: valid credentials, Redis server running, correct configurations; a Redis client instance should have been created. This means reading is as easy as running the following lines:

```javascript
export async function readFromRedis() {
  try {
    const value = await redis.get("name");
    return value;
  } catch (error) {
    console.error("Error reading from Redis:", error);
    return "";
  }
}
```

Note: In my case, `name` is the key I'll use to save data into Redis.

4. Write to Redis

```javascript
export async function writeToRedis(name: string): Promise<void> {
  try {
    await redis.set("name", name);
  } catch (error) {
    console.error("Error writing to Redis:", error);
  }
}
```

Notice how the key is `name`

5. Ensure helpers are run on the server

Is important to run these helpers in the server so the credentials are not exposed in the browser. Make sure to use `"use server";`. Take a look at the [example](https://github.com/garciadiazjaime/demo-reactjs/blob/main/app/react-server-action-redis/actions.ts).

Notice how I'm using an environment variable to store the Redis connection string: `process.env.REDIS_CONNECTION_STRING`. This information is sensitive and shouldn't be exposed in the browser.

This adds an extra jump from the browser to your server to the Redis server, but it is a secure way. There's an alternative to securely connect from the browser, but I would probably still go with a server function.

6. Finally, plug the methods into your React application.

```javascript
const [inputValue, setInputValue] = useState("");
const [name, setName] = useState("");

const handleButtonClick = async () => {
  await writeToRedis(inputValue);
  setInputValue("");
  await fetchName();
};

const fetchName = async () => {
  const nameFromRedis = await readFromRedis();
  setName(nameFromRedis);
};

useEffect(() => {
  fetchName();
}, []);
```

Take a look at the full [component](https://github.com/garciadiazjaime/demo-reactjs/blob/main/app/react-server-action-redis/page.tsx).

## Demo

Make sure to take a look at the [demo page](https://www.garciadiazjaime.com/react-server-action-redis)

## Conclusion

Adding Redis is pretty straightforward, especially if you have a team that takes care of the infrastructure. If not, you might need to spend a bit of time with AWS or any other provider, making sure you set up the Redis server properly and follow secure guidelines.

Once the Redis server is set up, plugging it into your React application is just about selecting an npm package that works for you. These packages do the heavy lifting and provide helpers to write and read, which is what you will need most of the time.

Thanks for reading.
