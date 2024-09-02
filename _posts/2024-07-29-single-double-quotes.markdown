---
layout: post
title: "JavaScript: Single Quotes or Double Quotes?"
date: 2024-07-29 08:00:00 -0500
categories: javascript
---

![JavaScript: Single Quotes or Double Quotes?](/assets/single-double-quotes/banner.png)

When it comes to using single or double quotes in JavaScript, it doesn’t matter as long as you use a tool to maintain a consistent format. The tool should automatically format the code to the expected output.

A popular formatter package is **Prettier**, which comes with a set of recommended rules for any project.

#### How to Configure Prettier:

1. **Install the package:**

   ```sh
   npm install --save-dev prettier
   ```

2. **Apply the desired rules:**
   Create the file `.prettierrc.js` and paste the following code:

   ```js
   const config = { singleQuote: false };
   module.exports = config;
   ```

3. **Format all files:**
   ```sh
   npx prettier . --write
   ```

The output should look like this:

```
npx prettier . --write
.prettierrc.js 5ms (unchanged)
app/page.tsx 107ms (unchanged)
app/layout.tsx 7ms
```

When Prettier runs, it applies all the [recommended rules](https://prettier.io/docs/en/configuration). In our configuration, only the `singleQuote` property was overridden, which is one of many rules Prettier offers.

### Summary

When working solo, using a formatter tool helps but isn't crucial. However, in a team setting, it’s essential to maintain a consistent style in the repository, regardless of the number of contributors.

Additionally, using Continuous Integration (CI) to run the formatting script on every pull request ensures that any formatting issues are caught early. Relying on automated tools for code formatting is far more efficient than manual checks. Locally, this means using a plugin for your editor, and on the codebase, it means setting up CI.
