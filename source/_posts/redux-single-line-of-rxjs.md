---
title: RxJs實現Redux功能
date: 2017-09-21 11:47:06
tags: [RxJS]
author: Tu-Szu-Chi
---

**Redux**的出現是為了解決數據的管理方式，而RxJS是更進階地解決異步*(Async)*操作的困擾
關於上述兩者的文章已有相當多的資源可以參考，在此分享一個很不錯的[演講](https://www.youtube.com/watch?v=AslncyG8whg)，是由Netflix的資深工程師-***Jay Phelps***，對RxJs應用在React以及之於Redux的關係有很好的解釋*(redux-observable)*

# 開始

原本的React & Redux，我們會**dispatch(action)**去改變state，在Observable來看可以這樣表示：
> state·····state··state-··········state·····>

當action被觸發都會回傳一個新的state，並且React會依照這新的state來決定是否re-render
在React中，*Store*即state儲存的地方，state又是由諸多的child-state*(Component)*組成，Component底下又會有自己的*actions*，action是經由*reducer*來回傳新的state....
以上看起來一層又一層的連帶關係其實沒那麼複雜，在Observable看來，每一個行為(action)都是一條流，我們可以這樣表示：

```javascript
const store$ = action$
  .startWith(initState)
  .scan(reducer)
```

store$開始先有startWith初始化狀態，scan功能就是Rx版的Array.protype.reduce，這使我們每當有action發生，經過reducer後會回傳一個新的state
實際上我們的action$不會只有一種，可以試著用[Merge](http://reactivex.io/documentation/operators/merge.html)來統合全部的action-stream到store裡；這部分的Design pattern有許多種，redux概念也沒有想像中複雜，可以動手試試，例如：

```javascript
const store$ = initState$.merge(reducer$) // reducer$也是由好幾個reducer$合成
  .scan((state, reducer) => state.merge(reducer(state)))
  .publishReplay(1)
  .refCount()
```

# 結語

Rx'Library真的是項很高效的工具，尤其有諸多語言版本，理解概念&熟悉操作可以用在許多平台上，在我看來是值得投資的
當然它和redux一樣都實踐了Flux style的管理方式，只是學習成本似乎更高，所以也不用盲目地一概都用RxJs，簡單的專案用redux或許是更高效開發的選擇

# Reference

* [Redux in a single line of code with RxJS](http://rudiyardley.com/redux-single-line-of-code-rxjs/)