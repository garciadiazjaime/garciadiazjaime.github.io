---
layout: post
title: "When Should You Write Unit Tests?"
date: 2024-09-02 08:00:00 -0500
categories: javascript reactjs nextjs web-development unit-test
---

![When Should You Write Unit Tests?](/assets/react-nextjs-unittest/banner.png)

## Unit-test

I'm a huge fan of unit-tests, especially when working in a team environment. While other types of tests like integration or end-to-end are important, in my experience, a solid unit test suite often reduce the need for additional testing layers.

That said, when I'm working on a personal project, I rarely write unit tests, and here's why:

a) Experimentation

Most of my personal projects are solutions to problems I've encountered, like finding [coupon codes](https://coupons.garitacenter.com/), discovering [Chicago music concerts](https://www.chicagomusiccompass.com/), or checking [border wait times](https://www.garitacenter.com/). These projects are my playground for experimenting with new APIs. For me, personal projects are about exploration and innovation, not rigorous testing.

b) Solo Development

Since I'm the sole contributor to these projects, I typically skip unit tests. In rare cases where a friend collaborates, we still don't prioritize writing unit tests.

---

However, in a professional setting, I always advocate for unit test, and here's why:

a) Refining implementation

Writing unit tests often highlights areas for improvement. While testing a component, I might notice that it has too many responsibilities or that certain logic could be abstracted for broader use cases. For me, unit testing is a way to assess and refine code. If a test is difficult to write, it usually signals that the implementation needs a refactor.

b) Change Resilience

Unit tests are invaluable when modifying code. If a test breaks after a change, it prompts me to verify that the change is intentional. If it is, the test is updated accordingly. Additionally, during cleanups, unit tests provide confidence that refactoring won't break existing functionality.

c) Documenting Behavior

Well written unit tests serve as great documentation, clearly outlining what a piece of code is supposed to do. In my view, unit tests are one of the best ways to document and communicate expected behavior.

## Coverage

Tools like `jest`, often provide coverage reports that show the percentage of code covered by tests. However, it's crucial to understand that quantity and quality are distinct concepts. Coverage reports measure quantity but don't guarantee the quality of tests.

Quality is directly tied to business logic. Knowing what's needed from the application and clearly expressing that in your tests. While I prioritize quality over quantity, quality naturally relies on having a sufficient number of tests. Without tests (0 quantity), there's no quality. The more tests you have, the better chances of achieving quality. But this doesn't mean you should write as many tests as possible; focus on quality instead.

Once you're confident in the quality of your tests, set minimum coverage thresholds. It's fine to adjust these thresholds as needed, as long as you maintain quality.

## GIVEN-WHEN-THEN

This is a format inspired by Behavior-Driven Development and has proven useful when writing requirements. In several teams I've worked with, adopting this format has helped align expectations between Product and Engineering.

In my post: [Try/Catch vs .then().catch()](https://www.garciadiazjaime.com/posts/javascript-next-tick), I provided an example of a component that makes an HTTP request and updates a list on the UI.

You can view the [source code here](https://github.com/garciadiazjaime/demo-reactjs/blob/main/app/try-catch-vs-then-catch/page.tsx).

The UI looks like this (very simple):

![Try/Catch vs .then().catch()](/assets/await-vs-then/banner.png)

The logic can be defined as follows:

```
GIVEN a user opens the page
WHEN the application loads
THEN a list of all music events should be displayed.
```

## Arrange-Act-Assert (AAA)

When writing unit tests, I like to follow the AAA pattern: Arrange, Act, Assert.

This pattern isn't about what the test is checking but rather how the unit test is structured. It makes the test more readable and easier to understand by clearly defining three distinct phases:

1. Arrange

Set up the configuration and any necessary precondition before invoking the component or function you're testing.

2. Act

Render the component or call the function that you want to test.

- Assert

Verify the outcome by checking if the function returns the correct value or if the component renders as expected.

Using this approach helps to maintain clear, organized, and effective unit tests.

## Writing a unit-test

Alright, now that we've covered the GIVEN-WHEN-THEN and AAA pattern, let's dive into writing the unit test:

```js
import { render, screen, act } from "@testing-library/react";

import Page from "./page";

describe("Page", () => {
  describe("GIVEN a user opens the page", () => {
    describe("WHEN the application loads", () => {
      test("THEN a list of all music events should be displayed.", async () => {
        // arrangement
        window.fetch = () =>
          Promise.resolve({
            json: () => [{ name: "mock event name" }],
          });

        // act
        await act(async () => {
          render(<Page />);
        });

        // assert
        expect(screen.getAllByText("mock event name").length).toEqual(2);
      });
    });
  });
});
```

Output:

```sh
npx jest app/try-catch-vs-then-catch/page.test.js
 PASS  app/try-catch-vs-then-catch/page.test.js
  Page
    GIVEN a user opens the page
      WHEN the application loads
        ✓ THEN a list of all music events should be displayed. (26 ms)

Test Suites: 1 passed, 1 total
```

---

Let's revisit the key reasons for writing tests:

- Refining implementation

Even with a simple component, there are often multiple ways to structure the code. By writing tests, I can continue refining the implementation with confidence, knowing that the primary functionality remains intact.

- Change Resilience

If we decide to add a loading indicator or provide user feedback for errors, these changes shouldn't disrupt the core functionality already covered by tests. This gives us confidence when implementing new features or refactoring.

- Documenting Behavior

Unit tests serve as documentation for the expected behavior defined by the product owner. Using the GIVEN-WHEN-THEN format, we can clearly see the connection between requirements and implementation.

## Conclusion

Here's a quote that a like:
"The code easiest to maintain is the code that was never written"

However, if code needs to be written, adding unit tests significantly reduces the chances of introducing bugs. In a professional setting, always prioritize writing unit tests and focus on quality over quantity.

## Links

- [Codebase](https://github.com/garciadiazjaime/demo-reactjs/blob/main/app/try-catch-vs-then-catch/page.tsx)
