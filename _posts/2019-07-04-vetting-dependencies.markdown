---
layout: post
title: Vetting Dependencies
date: 2019-07-04
comments: true
categories: Node.js
---

Dependencies are almost _too easy_ to add to a project, and over time become so ensconced, are very difficult to remove. Too often a dependency is introduced without proper due diligence or critical thought. This article provides a series of considerations to review before adding a dependency. It is written specifically in regards to the Node.js ecosystem, however the concepts transfer to any package ecosystem.

## Quality

Gauging quality is a critical deciding factor; it's a huge waste of resources to introduce a dependency, only to discover later (the hard way) that it's not production ready. Here are some quick ways to identify red flags which may be indicative of a poor quality dependency:

- Large numbers of dependencies and overall bundle size. [Bundlephobia](https://bundlephobia.com/) provides a good visual snapshot.
- Security vulnerabilities. Use the free [snyk scanning](https://snyk.io/vuln) tool to identify vulnerabilities for any package.
- Large numbers of open issues and/or pull requests
- Number of downloads on npm; a larger user base increases the likelihood an issue will be reported, improving quality.

## Activity

It's never a good idea to add an abandoned project as a dependency. These can be the cause of many future problems, including unresolved security vulnerabilities, important changesets left unmerged, and future incompatibilities with the latest version of the Node.js runtime.

A few ways to take the temperature of a project's activity:

- Number of commits and merged pull requests in the past few months; are the maintainers receptive to new changesets, and keeping the project from going stale?
- The project's `pulse` tab in Github; is there a healthy number of contributors, or is this really the work of a single maintainer?
- Number of downloads on npm; if the project maintainer requires help, a larger community provides a greater recruitment pool and reduces the chance of abandonment.

## Alternatives

> Many packages may solve your problem, but which one is right for your project?

 The travesty of having so many packages in the `npm` ecosystem is having a plethora of projects that all do the same thing. This leads to decision fatigue and sometimes results in a culture of "if it works, it's fine and belongs `package.json`" - for shame! Having so many projects to pick from only makes the evaluation of alternatives all the more important! Some tools for identifying alternatives:

- [npms.io](https://npms.io) provides better search than npm and google, and gives a `score` for each package based on its quality, maintenance, and popularity. There are other considerations beyond these 3 criteria (such is the purpose of this article), but the score provides a solid litmus test.

- [npmcompare.com](https://npmcompare.com/) is another tool for cross comparison of similar projects. It's strength lies in its display of metadata in a tabular format, making it easy to compare and contrast. _Beware: the overall score provided at the bottom of their table sometimes does not reflect the best decision and should be viewed with skepticism_. In particular, the scores weight towards popularity without taking into account whether the project is actively maintained. This results in veteran projects which have been deprecated or mostly abandoned scoring very well, because of their high usage rates. Still, `npmcompare` is a helpful tool for quickly comparing and contrasting various metrics.

> As a general rule of thumb, when considering multiple alternatives, pick the one which most specifically and concisely targets your problem, ceteris paribus.

## Is the problem being solved _worth_ the cost of a dependency?

Many projects offer a large feature set, while only a fraction of these are actually used by the consuming application. It's important to evaluate whether the long term cost of the dependency is worth the value being provided by the sub-feature(s) utilized. Sometimes it's better to write a small snippet, or look for a simpler alternative (related: [You might not need debug](/blog/2019/06/16/you-might-not-need-debug/)).

## devDependencies should be viewed with skepticism!

> Too often, `devDependencies` are added on a whim, becoming a dumping ground for every nice-to-have, gee-whiz development tool ever encountered.

Dev dependencies have a cost! Longer time to `npm install`, longer build times on CI, and one more package for contributors to grok. Project bloat is a major concern, and `devDependencies` should be no exception.
Ask if the devDependency being added truly advances the project as a whole, or if it's only adding incremental value for developers who understand it while increasing the cognitive burden for everyone else.

