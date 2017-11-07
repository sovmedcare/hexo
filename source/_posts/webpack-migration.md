---
title: Webpack Migration 從 v1 到 v3 
date: 2017-09-21 15:43:08
tags: [webpack, javascript]
author: jackypan1989
---

之前幾個專案都已經直接使用 webpack v2 以上, 但剛好這次手邊有一個專案還在使用 v1, 所以這邊就一起跟大家介紹如何無痛升級到最新版 (v3.6), 包括 loader / plugin 的一些改變，其實大部分都沒變，只有一些關鍵字跟配置調整。

# 更新方式

```javascript
yarn add --dev babel-core babel-loader babel-preset-env webpack webpack-dev-server
```

# 版本差異

## 1. resolve 一律用 modules 來設定

```javascript
// v1
resolve: {
  root: path.join(__dirname, "src"),
  modulesDirectories: ...,
  extensions: ...,
  fallback: ...
}

// v3
resolve: {
  modules: [
    path.join(__dirname, 'src'),
    'node_modules'
  ]
}
```

## 2. module.loaders 改成用 module.rules 以及 use 關鍵字

```javascript
// v1
module: {
  loaders: [
    {
      test: /\.css$/,
      loader: "style!css"
    }
  ]
}

// v3
module: {
  rules: [
    {
      test: /\.css$/,
      use: ["style-loader", "css-loader"]
    }
  ]
}
```

## 3. 利用 webpack-merge

如果你有 prod / dev / test / electron 各種 config
那我會建議使用 webpack-merge 把重要部份抽出來方便管理

## 4. 其他我覺得重要的改動

- loader name 要寫完整，不能簡寫，除非還要另外使用 resolveLoader (舊選項)
- 不用裝 json-loader / UglifyJsPlugin(免另外安裝) / OccurrenceOrderPlugin(直接內建) / DedupePlugin(已移除)
- loader option 一律放在 rules 的每個 loader 下面
- babel-preset 一律改用 env 不用使用 es201x

# 效率比較(未優化前)

- v1.14.0

  build time: 39.517s
  size: 2.29MB

- v3.6.0

  build time: 32.028s
  size: 2.54MB

## Reference

1. [實際範例](https://gist.github.com/jackypan1989/a832db223a8d4a24d2edd9b6cde83da3)
1. [webpack 官方](https://webpack.js.org/)
