---
title: 漫談 FRP - ReactJS vs CycleJS 
date: 2017-09-08 15:43:08
tags: [React, Cycle]
---

2014年，CycleJS 作者 André Staltz 寫了一篇 Don't React ，大力批評 ReactJS 並不 reactive。這篇文章在當時引起軒然大波，雖然在三年後的今天已經被移除，但我們依然可以在 youtube 上面看到古老的戰神影片 [André Medeiros: Don't React (Webbisauna 2014)](https://www.youtube.com/watch?v=9QObt0SGriI)。這三年期間，我們都可以看到兩個 lib 的變化，還有社群推行方向，以下會針對兩個 lib，並討論 FRP (響應風格的函數式編程)。

# FRP (響應風格的函數式編程)

* FP (函數式編程)
所有東西都是函數，在語言中是第一公民，函數可以當作參數外 ```(x, func) => func(x)```，而函數也可以回傳另一個函數 ```x => y => x + y```，因此可以創造出各種高階函數(Higher Order Function)，也可以進行各種合成 (compose) 來完成各式各樣的功能，不用像物件導向，必須用很多種設計模式去優化代碼，我必須說，整個開發者世代花了好多時間導入物件導向，最後得到『多用合成，少用繼承』的結論，但其實在函數式編程中，合成是多麼渾然天成，一招就能打天下 :p

* Reactive (響應風格)
函數中不會直接去操作外部的函數，取而代之的是都是去反應外部函數(參數)做了什麼事，而去改變，在物件導向風格常見於觀察者模式 (Observer pattern)，好處是分工非常明確，也容易追蹤代碼潛在問題。

# 兩者比較

* CycleJS
Cycle 更強調 reactive，因此你不會在裡面看到 this 關鍵字，但因為你不可能全部都沒有 side effect。所以這部份 cycle 利用 driver，把他認為髒的 side effect 抽出來處理。另一方面 Cycle 本身全部都是使用 RxJS / xstream，去管理所有的資料流，user input 會轉換成流，state 與 state 改變也會轉變成流，甚至連 render dom 也是流，因此整個架構就是多個流的合成與串接，action -> state -> render -> action 形成一個超級流的大循環，我想這也是為什麼叫做 Cycle 的原因。

* ReactJS
原本 lib 本身其實談不上太多 reactive，因為 reactive 的部分只有 ```render()``` 這個函數，Component 內的其他函數都是 active，因此會看到 this & this.setState 這種東西，反觀 CycleJS 就不會看到 this 這個 JS 一直以來常常會出包的關鍵字，當然在 redux 出來後，以及 rx-react, redux-rx, 甚至 redux-observable 引進後，user input 與 state 改變才開始真正一步步 reactive 化 / stream 化。

# 結論

我個人覺得之所以 fb 會大力推動 react 的原因，有一個很大重點在於想要推函數式編程 (functional programming)，但不可能一開始就推個 Haskell 大家都嚇死，等到大家慢慢熟悉這個體系，開始 componenet base programming，寫一段時間後，就會開始寫高階元件 (High Order Component，也就是一個函數會傳進一個 component 再吐一個 component) ，然後又開始學習 render 永遠只根據 prop 改變的 stateless component (pure functional component)。你就會發現你入坑了(你也可以看到 fb 釋出這些想法的進程)，開始感受 FP 的奧妙，原來 component 的各種組成合成，就跟 FP 中的函數合成一樣。

Cycle: 天生全部 reactive，架構上更好更美
React: 需要第三方協助(rx-react...)，但有大量社群支持

總歸一句，在前端這麼吃重 ui 操作與呈現的情況下，社群發展到最後一定是慢慢往類似或是相同方向前進。
至少三年前的爭議已經平息，接下來就看社群是不是可以往更函數式編程前進了(ELM, Purescript ... etc)。

## Reference

1. [CycleJS](https://github.com/cyclejs/cyclejs)
2. [rx-react](https://github.com/fdecampredon/rx-react)
3. [redux-rx](https://github.com/acdlite/redux-rx)
4. [redux-observable](https://github.com/redux-observable/redux-observable)
