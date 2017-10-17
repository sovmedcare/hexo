---
title: 5個RxJs有趣範例
date: 2017-09-15 11:24:30
tags: [FRP, RxJS]
---
之前提到過RxJs，這篇文章主要是從Rx'Library官方的練習題挑出5個來介紹，都蠻好理解的
有興趣的朋友也可以直接[試試看](http://reactivex.io/learnrx/)，run起來對了才能解鎖下一題，挺有趣的

***

# Retrieve id, title, and a 150x200 box art url for every video

首先要先介紹一下Function Programming編程精神，data的結構大概是以下這樣的Array

``` javascript
var movieLists = [
  {
    name: "Instant Queue",
    videos: [
      {
        "id": 70111470,
        "title": "Die Hard",
        "boxarts": [
          { width: 150, height: 200, url: "http://cdn-0.nflximg.com/images/2891/DieHard150.jpg" },
          { width: 200, height: 200, url: "http://cdn-0.nflximg.com/images/2891/DieHard200.jpg" }
        ],
        "url": "http://api.netflix.com/catalog/titles/movies/70111470",
        "rating": 4.0,
        "bookmark": []
      },
    {
      "id": 654356453,
      "title": "Bad Boys",
      "boxarts": [
        { width: 200, height: 200, url: "http://cdn-0.nflximg.com/images/2891/BadBoys200.jpg" },
        { width: 150, height: 200, url: "http://cdn-0.nflximg.com/images/2891/BadBoys150.jpg" }
      ],
      "url": "http://api.netflix.com/catalog/titles/movies/70111470",
      "rating": 5.0,
      "bookmark": [{ id: 432534, time: 65876586 }]
    }
    ]
  }
  // ...
```

因要針對每一筆videos做檢索，所以先

``` javascript
movieLists.map(obj =>
  obj.videos.map(video =>
    // ...
  )
)
```

到這一部已經可以拿到id,title了，接著要過濾width !== 150的boxarts

``` javascript
movieLists.map(obj =>
  obj.videos.map(video =>
    video.boxarts.filter(image => image.width === 150)
    // ...
  )
)
```

最後直接回傳我們要的Object形式

``` javascript
movieLists.map(obj =>
  obj.videos.map(video =>
    video.boxarts
          .filter(image => image.width === 150)
          .map(boxart => { return {id:video.id, title:video.title, boxart:boxart.url}})
  )
)
```

到此階段回傳的Array會多了兩層

``` javascript
[[[{"boxart": '...', "id": 70111470, "title": "Die Hard"}], ...]]]
```

可以在外層兩個map各補上降維的Method

``` javascript
movieLists.map(obj =>
  obj.videos.map(video =>
    video.boxarts
          .filter(image => image.width === 150)
          .map(boxart => { return {id:video.id, title:video.title, boxart:boxart.url}})
  ).concatAll()
).concatAll()
```

雖然這是個簡單的FP範例，但可以感受出每個步驟都具有語意化

# Retrieve url of the largest boxart

再來也是要介紹FP核心之一的**reduce**
data結構如下，要做比較的動作，找出Size最大的圖片_(width * height)_

``` javascript
var boxarts = [
  { width: 200, height: 200, url: "http://cdn-0.nflximg.com/images/2891/Fracture200.jpg" },
  { width: 150, height: 200, url: "http://cdn-0.nflximg.com/images/2891/Fracture150.jpg" },
  { width: 300, height: 200, url: "http://cdn-0.nflximg.com/images/2891/Fracture300.jpg" },
  { width: 425, height: 150, url: "http://cdn-0.nflximg.com/images/2891/Fracture425.jpg" }
]
```

reduce第一個參數要傳入處理動作的Function，第二個參數是初始值，預設為Array中第一個元素，所以我們不用特別傳入

``` javascript
boxarts.reduce((currentImg, current) =>
  (currentImg.width * currentImg.height > current)
  ? currentImg.width * currentImg.height > current
  : current)
  .map(item => item.url)
```

以上為簡單的「比較」邏輯，reduce第一個參數中的Function會帶有兩個參數，第一個為當前的元素，第二個為當前的值
因reduce會回傳一個Array，而在這裡我們的Array中只會有一個元素，即Size最大的boxart
map同樣會回傳一個Array，裡面也只有一個url字串

# Completing sequences with takeUntil()

**takeUntil()** 我覺得是個很好用的功能，可以預先註冊好什麼時候停止取資料

``` javascript
const stopButtonClicks = Observable.fromEvent(stopButton,'click')
  microsoftPrices = pricesNASDAQ
    .filter(priceRecord => priceRecord.name === "MSFT")
    .takeUntil(stopButtonClicks)
```

我們假設pricesNASDAQ是一個NASDAQ股市報價的stream，而我們想從中取得微軟的報價所以利用了**filter**
並註冊了當**stopButton**按下時就停止取stream資料，可以在這[JS Bin](https://jsbin.com/zuvavetuno/1/edit?js,console,output)試試
會發現當stopButton按下後，會觸發Observer中的complete，其餘還有**takeLast**、**takeWhile**可以試試

# Throttle Input

接下來要介紹Rx'Library中非常實用的功能之一 - **throttleTime**
我們可以設定一段時間，例如*throttleTime(1000)*，表示一秒內所有的數據，我們只截取第一筆
這剛好可以解決我們針對某些API或DOM Event要避免短時間內重複觸發的問題，最典型的就是搜尋框(search box)

```javascript
const input = Rx.Observable
  .fromEvent(document.getElementById('input'),'keydown')
  .throttleTime(1000)
```

與之相對的是**debounceTime**，可以在[JS Bin](https://jsbin.com/zuvavetuno/1/edit?js,console,output)試試

# Distinct Until Changed Input

這是最後一個要介紹的Method，只取和前一個相比有變化的值，這概念是否有點熟悉？
在React中我們也是希望Component要有改變到state才render，故有了*shouldcomponentupdate*這生命週期
那如果有方法能幫我們確保拿到的一定是有變化的數據，是否就能少寫這*shouldcomponentupdate*了呢
一樣附上[JS Bin](https://jsbin.com/zuvavetuno/1/edit?js,console,output)給大家試試

# 結語

Rx'Library是個功能很實用的工具，也因為功能很多，所以需要花時間去摸索，當碰到問題時才會發現可以用幾個Method輕鬆的解決
Rx實做多了，也會發現自己Function Programming精進了不少