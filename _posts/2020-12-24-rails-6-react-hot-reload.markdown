---
layout: post
title: "Rails 6 + React hot reload"
subtitle: ""
date: 2020-12-24
categories: [programming]
---

Following some simple guilde how to add react hot reloading in rails 6 app did not work for me and resulted with following error in the console
```
Warning: Functions are not valid as a React child. This may happen if you return a Component instead of from render. Or maybe you meant to call this function rather than return it.
```

It turned out it missed two small details. First was incorrect import `'react-hot-loader` instead of `'react-hot-loader/root` and missing `hmr: true` in `webpacker.yml`. However lets cover all the steps from the beggining.

# Enabling React hot reload

## Install and configure react hot loader

Install `react-hot-loader`:

```
yarn add react-hot-loader
```

Configure this plugin in `babel.config.js`:

```
plugins: [
  'react-hot-loader/babel',
]
```

Enable `hmr` in `webpacker.yml`:
```
dev_server:
  hmr: true
  ...
```

## Modify your root component

First import hot loader:
```
import { hot } from 'react-hot-loader/root'
```
It is important that it's imported before `React`
Then change the way the component is exported:
```
export default hot(App)
```

Whole component should look like this:
```
import { hot } from 'react-hot-loader/root'
import React from 'react'

const App = () => {
  return <div>Hello World!</div>
}

export default hot(App)
```

Cheers
