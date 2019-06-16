---
layout: post
title: You might not need debug
date: 2019-06-16
comments: true
external-url:
categories: Node.js
---

The [debug](https://github.com/visionmedia/debug) package is one of the best-known and most popular packages on npm. Started in 2011 by celebrity open source developer TJ Holowaychuk, it's been a staple of the Node.js ecosystem for as long as most can remember.

Recently, the current maintainers of the package have begun modernizing the codebase and dropping support for older browsers. These breaking changes, introduced in versions 4+, force consumers of isomorphic packages to re-evaluate their usage of `debug()`.

This post argues that the value provided by `debug` is minor, and  consumers should consider eliminating it as a dependency.

## What it does

`debug` enables debug message to be turned on and written to standard out by setting an environmental variable. If a package isn't performing like you'd expect, debug may lend clues as to what's going on under the hood.

Consider the following snippet:

```js
const debugFactory = require('debug')
const debug = debugFactory('project')

debug('first call')
debug('second call %o', { foo: 'bar' })
```

which outputs:

![debug-output](/assets/2019-06-16-you-might-not-need-debug/debug-output.png){:width="700px"}

## What it provides

> Observing the output above demonstrates the **core value** of turning on debug output to standard out via an environmental variable.

Also demonstrated are many of the minor features it provides:

- Atttractive output; colorized text + pretty printing objects
- Namespacing messages via a prefix
- Measuring the delta in time between the current and previous statement

There are many other features supported, all documented in the project's [README](https://github.com/visionmedia/debug), however it has been the observation of this author that few projects use these advanced features. Most consumers of the package tend to use it for constant text and outputting variables.

## What it costs

With every dependency comes a cost, and `debug` is no exception:

- 5.8 kb minified
- Contains itself + one other dependency
- The aforementioned problems with browser compatibility and transpilation
- Many of the advanced features tend to go unused

## How to replace it

An isomorphic, cross-browser, barebones implementation of `debug()` can be written in three lines of code:

```js
function debug (...messages) {
  if (typeof process !== 'undefined' && process.env && process.env.DEBUG && process.env.DEBUG.match(/^project$/)) {
    console.log(messages.map(JSON.stringify).join(','))
  }
}
```

which ouputs:
```bash
localhost:~/project-debug|⇒  DEBUG=project node index.js
"first call"
"second call",{"foo":"bar"}
```

Clearly the snippet above does not provide all of the features provided by `debug` (doing so would be to re-implement `debug` all over again). However, this does demonstrate the **core value** may be delivered with minimal code. This should call into question whether `debug`'s minor features are worth the costs listed above.