---
layout: post
title: "Shai Hulud Scanner"
date: 2025-12-27 10:00:00 -0500
categories: nginx reverse-proxy software-architecture
---

![Shai Hulud Scanner](/assets/2025-11-27-shai-hulud-scanner/banner.png)

This week I spent some time looking for infected npm packages. Initially, the warning was to not install any package or run any AI agent, so I went ahead and created this Node.js script.

```javascript
const scanForShaiHulud = (content) => {
  if (!content) return;

  const report = {
    total: 0,
    warning: [],
    infected: [],
  };

  Object.keys(content.packages).forEach((packageName) => {
    if (!packageName) {
      return;
    }

    let cleanedName = packageName.replace("node_modules/", "");
    if (cleanedName.includes("/node_modules/")) {
      cleanedName = cleanedName.split("/node_modules/")[1];
    }

    report.total += 1;

    if (infectedPackages[cleanedName]) {
      if (
        infectedPackages[cleanedName].includes(
          content.packages[packageName].version
        )
      ) {
        report.infected.push({
          name: cleanedName,
          version: content.packages[packageName].version,
          affectedVersions: infectedPackages[cleanedName],
        });
      } else {
        report.warning.push({
          name: cleanedName,
          version: content.packages[packageName].version,
          affectedVersions: infectedPackages[cleanedName],
        });
      }
    }
  });

  return report;
};

const fileContent = fs.readFileSync("./package-lock.json", "utf-8");
const parsedContent = JSON.parse(fileContent);
const report = scanForShaiHulud(parsedContent);
console.log(report);
```

Before this week, I knew that `package-lock.json` declared all the specific versions installed, but I hadn’t really spent time understanding the structure. Now I know that `package-lock.json` stores, in the key `.packages`, a map of all the packages installed in the repository—either as dependencies of the project or dependencies of dependencies. The point is that `.packages` holds a map with all the packages installed, so it can be used to check if any of the infected packages reported by [wiz-sec-public](https://github.com/wiz-sec-public/wiz-research-iocs/blob/main/reports/shai-hulud-2-packages.csv) matches the version.

The helper is pretty simple: it iterates over every single key found in `.packages` and checks it against the list passed.

At the end, the report shows if there’s any:

- `warning`: package reported by Wiz as infected, but not the same version
- `infected`: package reported by Wiz as infected, with a matching version

In the case of `infected`, you must remove that package; otherwise, whatever secrets are stored in your project locally could be exposed.

In the case of `warning`, I would do the same—remove the package—just to be safe. But if the version is lower than the one reported, in theory you shouldn’t be in trouble.

Feel free to download the [shai-hulud-scanner](https://github.com/garciadiazjaime/demo-reactjs/blob/main/app/shai-hulud-scanner/shai-hulud-scanner.js) to your Node.js project. Once you run it, you should see a report like this:

![Terminal Shai Hulud Scanner Report](/assets/2025-11-27-shai-hulud-scanner/terminal-shai-hulud-scanner-report.png)

Hopefully everything is clear; otherwise take immediate action.

Additionally, you can visit this [Shai-Hulud Scanner](https://demo.garciadiazjaime.com/shai-hulud-scanner) and upload a `package-lock.json` file. The same scanner function will run, and you can see the results in the browser.

![Online Shai Hulud Scanner Report](/assets/2025-11-27-shai-hulud-scanner/online-shai-hulud-scanner-report.png)

**Note**: This post is meant for package-lock.json. If you are using `yarn` or `pnpm`, this won’t work because those package managers use different lock files. In any case, you should be able to adjust the logic a bit and make it work.

Luckily, we didn’t find any threats, proving that the Wiz list is comprehensive. I’m a big fan of open source—I believe in the power of community—but as with everything, there are risks too. Someone will always try to break the rules, which in some way pushes us to be more aware and spend more time on security. After this week, I’ve seen numerous packages for detecting this kind of threat, and that’s one of the perks of open source: code written by the people, for the people.

Happy coding.
