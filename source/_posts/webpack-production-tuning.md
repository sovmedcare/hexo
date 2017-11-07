---
title: Webpack production bundle 優化 (72x)
date: 2017-09-28 15:43:08
tags: [webpack, javascript, optimization, production, 優化]
author: jackypan1989
---

本篇將會介紹 webpack 3 中常見的優化方式
包含問題分析，plugin介紹等等

# 優化成效

  1. 大部分的情境下，使用者用量(專案為內部使用者為主)
  從 3.6 MB 降到 51 KB (app)
  降為原本的 **1.38 % (縮減 98.62 %)**

  1. 在完全沒有 cache 機制，必須重拉 vendor 情況下
  也是從 3.6 MB 降到 726 KB (app + vendor)
  降為原本的 **19.6 % (縮減 80.4 %)**

![](https://i.imgur.com/39u6xKz.png)

# 優化流程

## 1. 觀察現有狀況與問題

  僅使用 DefinePlugin(with production), UglifyJsPlugin(with minimize)
  bundle 出來的大小為 3.6MB, 存在很大改善空間(3G網路讀取時間過長)
  因為不是所有 module 都在每次 build 都改動到

  ![imgur](https://i.imgur.com/2cfimJS.png)

## 2. 利用 BundleAnalyzerPlugin 分析 bundle 成分

  [BundleAnalyzerPlugin](https://github.com/th0r/webpack-bundle-analyzer) 會協助解析 bundle 中有哪些單元
  他會生成一個網頁，透過該網頁可以看到用到的 module 以及它的大小

  ![](https://cloud.githubusercontent.com/assets/302213/20628702/93f72404-b338-11e6-92d4-9a365550a701.gif)

## 3. 利用 CommonsChunkPlugin 抽出 lib

  透過分析發現，第三方 lib 佔了大部分的成分(例如 react, rxjs, redux, moment, antd ...etc)
  但這些 lib 的改動(例如升級 react 版本)，在 production build cycle 中次數是很少的
  因此透過 CommonsChunkPlugin 把所有第三方都抽出來，與我們自己的 code 分開
  (例如產生出 app.js, vendor.js 這樣一來就每次只更新 app.js 即可)

  ps: 也可以在 entry 中，指定那些比較大的 lib

## 4. Uglyify 優化

  除了調整 minify, souceMap 外，compress選項也可以另外調整
  (例如不要在生成支援 ie8 的 code, 去掉 dead code, comment等)

## 5. IgnorePlugin

  如果有使用 momentJS 可以發現，他會 bundle 所有的語言包
  這時就可以利用 IgnorePlugin 去掉沒用到的語言包

## 6. ModuleConcatenationPlugin & HashedModuleIdsPlugin

  1. ModuleConcatenationPlugin
  參考 RollupJS 將有相關的 module 放在同一個閉包裡面，所以會減少閉包數量

  1. HashedModuleIdsPlugin
  給打包的 module 一個穩定 hash 值 (如果沒啟用，抽出第三方 lib 就會大打折扣)

## 7. Gzip (非常有效降低大小)

  利用 CompressionPlugin 先 prebuild 好 gzip 檔案
  而 gzip 需要 server side 支持
  下面列幾種方法
  1. express 的 middleware 插件
  1. nginx 設定 static gzip file 支持
  1. server on-the-fly 支持(不建議，因為會吃系統效能)

## 8. 利用 chunkhash 與 htmlWebpackPlugin 來達到 cache

  修改 output 裡面的 chunkhash 來達到確認是否有更新
  因為只要有添加新的模組或修改 chunkhash 就會改變
  (app.38d8273182132939.js -> app.38d8d73182139823.js)

  但是問題在於原本 express 的 view template 中是固定的
  因此透過 htmlWebpackPlugin 把新的得到的 chunkhash inject 回去
  這樣就能完美使用瀏覽器的 cache 機制

# 真實案例

```javascript
// 優化前
plugins: [
  new webpack.LoaderOptionsPlugin({
    minimize: true,
    debug: false
  }),
  new webpack.DefinePlugin({
    'process.env': {
      'NODE_ENV': JSON.stringify('production')
    }
  }),
  new webpack.optimize.UglifyJsPlugin()
]

// 優化後
plugins: [
  // new BundleAnalyzerPlugin(),
  new webpack.DefinePlugin({
    'process.env': {
      'NODE_ENV': JSON.stringify('production')
    }
  }),
  new CleanWebpackPlugin(path.join(__dirname, 'dist')),
  new webpack.optimize.ModuleConcatenationPlugin(),
  new webpack.optimize.CommonsChunkPlugin({
    name: 'vendor',
    filename: 'vendor.[chunkhash].js',
    minChunks: module => module.context && module.context.indexOf('node_modules') >= 0
  }),
  new webpack.IgnorePlugin(/^\.\/locale$/, /moment$/),
  new webpack.optimize.UglifyJsPlugin({
    compress: {
      warnings: false,
      screw_ie8: true,
      conditionals: true,
      unused: true,
      comparisons: true,
      sequences: true,
      dead_code: true,
      evaluate: true,
      if_return: true,
      join_vars: true
    },
    output: {
      comments: false
    },
    sourceMap: true
  }),
  new webpack.LoaderOptionsPlugin({
    minimize: true,
    debug: false
  }),
  new webpack.HashedModuleIdsPlugin(),
  new CompressionPlugin({
    asset: '[path].gz[query]',
    algorithm: 'gzip',
    test: /\.(js|html)$/,
    threshold: 10240,
    minRatio: 0.8
  }),
  new HtmlWebpackPlugin({
    template: path.join(__dirname, 'src', 'server', 'view', 'template.html'),
    inject: true,
    filename: path.join(__dirname, 'src', 'server', 'view', 'index-prod.html')
  })
]
```

## Reference

1. [webpack 官方](https://webpack.js.org/)
