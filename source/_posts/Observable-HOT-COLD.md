---
title: RxJs Observable - HOT & COLD
date: 2017-09-13 12:15:25
tags: [RxJS,FP]
author: Tu-Szu-Chi
---
![Observable Hot & Cold](https://image.slidesharecdn.com/untitled-150216041015-conversion-gate02/95/reactive-programming-with-rx-22-638.jpg?cb=1424081878 "截自Google Image")

Rx'Library Observable可大致分為兩種類型 - *Hot* & *Cold*
在往下介紹前，可以先將*Hot*想成廣播電台(Radio)，*Cold*是CD
Hot Observable的訂閱者，會讀取同樣的source；Cold Observable則每個訂閱者的source都是獨立的

***

# Cold Observable

只有當訂閱者訂閱的時候，Cold Observable才會推送資料，且每個訂閱者都是獨立的流，就像是CD只有當你放進去按播放後才會有音樂
Observable.interval就是種Cold類型的流

```javascript
const source = new Observable.interval(100).take(3)
```

可以在這[JS Bin](https://jsbin.com/kiwitozejo/1/edit?js,console)看到observerA & observerB是各自的數據流

# Hot Observable

不管有沒有被訂閱，只要創建後就開始有數據流，Mouse Click Event就是最清楚的範例
Click事件一直在發生，但只有當你訂閱時你才收得到，且收到的是訂閱後的「觸發事件」，之前的不會收到
當然，如果你取消訂閱了，數據流依然會持續產生
這類的情境像是「股市最新報價」，你只會收到最新的那一個報價，不會收到五分鐘前的那筆
Hot Observable原理其實是我們和數據流間還有一個「**中間人**」，會一直監聽數據流，我們再跟他取資訊就好(真是忙碌的傢伙)
**中間人**會有一份訂閱者的清單，這樣才知道推播時要推送給哪些人

## Subject

先附上[JS Bin](https://jsbin.com/fumakubupa/1/edit?js,console)的Code
之前有提到訂閱者(Observer)會有三個Method - next、error、complete，理所當然Subject作為中間人他也該具備這些特質
並且有一份訂閱者的清單，有興趣的可以看官方的[Source Code](https://github.com/ReactiveX/rxjs/blob/master/src/Subject.ts#L22)
Subject可以訂閱也可以被訂閱，他既是Observable也是Observer
他還有其他三個變形 - BehaviorSubject、ReplaySubject、AsyncSubject，能在我們訂閱時有不同效果

## Operator

上一段的Code中，我們先新建Subject，再一一訂閱，這讓程式碼變的冗長，Rx'Library有提供些Operator讓我們寫的更簡潔點

### multicast

```javascript
const source = new Observable
                    .interval(300)
                    .take(3)
                    .multicast(new Rx.Subject())
```

multicast會回傳可連結(connect)的Observable，要真的執行connect後，Subject才會去訂閱source，且開始推送數據

```javascript
const realObserver = source.connect()
```

connect會回傳subscription，將此unsubscribe才會真正退掉Observable，將observerA/observerB退訂中間人還是會收到數據流

```javascript
realObserver.unsubscribe()
```

以下這種寫法可以更簡潔

```javascript
const source = new Observable
                    .interval(300)
                    .take(3)
                    .publish()
```

### refCount

但要多寫一段connect也是很麻煩，我們希望一有人訂閱就開始推送(observers.length > 0)，refCount可幫助我們

```javascript
const source = new Observable
                    .interval(300)
                    .take(3)
                    .publish()
                    .refCount()
```

要退訂不讓數據流推送的話，只要讓訂閱人數變成0即可(範例中就是讓observerA & observerB unsubscribe即可)

再給一個簡潔的寫法，refCount + publish

```javascript
const source = new Observable
                    .interval(300)
                    .take(3)
                    .share()
```

# 結語

Rx'Library是一個很實用但也相對需要花時間學習的工具，畢竟要去習慣「Everything is stream」的思維
在此附上RxJs的核心開發者之一 - *Andre Staltz*所寫的[應用範例](http://jsfiddle.net/staltz/8jFJH/48/)，推薦大家去看看

***

# Reference

[30 天精通 RxJS(22): 什麼是 Subject？](http://ithelp.ithome.com.tw/articles/10188633)
[Hot vs Cold Observables](https://medium.com/@benlesh/hot-vs-cold-observables-f8094ed53339)
