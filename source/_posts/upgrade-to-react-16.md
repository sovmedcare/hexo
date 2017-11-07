---
title: webpack upgrade to 16
date: 2017-09-29 16:43:08
tags: [react v16, javascript]
author: jackypan1989
---

Facebook just announced the release of React v16.0.

# New features

1. fragments
1. error boundaries
1. portals
1. custom DOM attributes
1. improved server-side rendering
1. small lib size
1. better render perf (with react-fiber)

  ```text
  https://github.com/facebook/react/issues/10686

  v15
  render functional component tree x 115,932 ops/sec ±4.10% (55 runs sampled)
  render class based component tree x 255,407 ops/sec ±4.46% (58 runs sampled)
  render class that renders functional components x 252,045 ops/sec ±5.47% (56 runs sampled)
  Fastest is render class based component tree,render class that renders functional components

  v16
  render functional component tree x 204,931 ops/sec ±2.21% (59 runs sampled)
  render class based component tree x 339,215 ops/sec ±2.98% (58 runs sampled)
  render class that renders functional components x 326,880 ops/sec ±4.51% (56 runs sampled)
  Fastest is render class based component tree,render class that renders functional components
  ```

# Install

  Just update your **package.json**
  No other configurations, and no migration issues :D
  (It's true !!!!!)

  ```javascript
  yarn add react react-dom
  ```

# Pratical performance (in chrome devtool)

v16
![img](https://i.imgur.com/t0s1dys.png)

v15
![img](https://i.imgur.com/U3XNKFd.png)

# You maybe also need to update (optional)

```javascript
yarn add prop-types recompose redux
```

# Reference

1. [React v16.0](https://facebook.github.io/react/blog/2017/09/26/react-v16.0.html)
