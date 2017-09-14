---
title: RxJs Observable 介紹
date: 2017-09-13 10:32:21
tags: [RxJs,FRP]
---
![Everthing is stream](https://cdn-images-1.medium.com/max/1600/1*j-SOtxql-Sqmvj0i0TWqMg.jpeg "截自Medium")

# 關於RxJs

**RxJs**是**ReactiveX**的一個JS Library(還有其他平台 ex.RxJava、RxSwift...)
>「An API for asynchronous programming with observable streams」

以上是ReactiveX官網標題，也響應了文章頂部的圖片，將任何動作、事件都看成是「流」，我們去監聽、觀察，經過轉換、合併、過濾，再給予對應的Callback

## Observable

Observable是Rx'Library的核心，它的概念是由兩個Design Pattern融合起來 - *Observer* & *Iterator*

### Observer

其實跟我們平常用到的**addEventListener**概念一樣，註冊一個監聽事件，當觸發時執行**clickHandler**

``` javascript
document.body.addEventListener('click', clickHandler)
```

    當然，一個事件可以有多個Listener，這在後續會解釋

### Iterator

想像成有一個**指針**指向一個資料序列(ex.Array)，我們想要得到這序列的資料必須透過Iterator的方法 - next()
當我們呼叫next()後，Iterator才會丟出一筆資料出來

``` javascript
arr = [1,2,3,4] //假設arr是Iterator
arr.next() // 1
arr.next() // 2
// ...
```

總結來說Observable這個可觀察的流(stream)，上面會有資料隨著時間推送，它可以經過序列的方式處理資料(map、filter、reduce...)，甚至也可以合併兩個不同的流，來讓Listener得到資料

## Observer

Observer是較會頻繁用到的功能(其餘還有Subject、Schedulers...)，他也是之前比喻的Listener
我們用實例來講解這些，首先建立一個要觀察的流(Observable)

``` javascript
let observable = Rx.Observable
    .create(observer => {
        observer.next('S');
        observer.next('O');
        observer.next('V');
        observer.complete();
    })
```

>實務上我們的Observable比較不會這樣建立，通常會直接用fromEvent、fromPromise之類的，而不是create，針對某個事件來做
>在此只是範例演練而已

這個observable會依序推送S->O->V給observer，接著我們要有一個Observer來監聽這個流

``` javascript
observable.subscribe({
    next: (val) => console.log(val),
    error: (err) => console.log(err),
    complete: () => console.log('Complete !')
})
```

>.subscribe(next, error, complete)，Observer就是由三個Method組成，負責處理接收到資料後、發生錯誤後和整個observable結束後

當這個流被訂閱後馬上就會送出資訊給監聽的人，可以在這個[JS Bin](https://jsbin.com/ceragiwogo/edit?js,console)試試

## Operator

有了流&監聽者後，之前也有提到Observable可以像序列一樣處理資料，所以這裡加上Function Programming的核心進去

``` javascript
const observable = Rx.Observable
    .create(observer => {
        observer.next('S')
        observer.next('O')
        observer.next('V')
    })
    .map(str => str + '!')
// ...subscribe
// 'S!'
// 'O!'
// 'V!'
```

再舉一個例子

``` javascript
const observable = Rx.Observable
    .create(observer => {
        observer.next('S')
        observer.next('O')
        observer.next('V')
    })
    .scan((x, y) => x + y);
// ...subscribe
// 'S'
// 'SO'
// 'SOV'
```

其實scan就是Observable版的reduce，兩者的差別，前者必會回傳一個Observable物件，後者就是回傳Value(Int、Array、Function...)
所以讀者可以把observable印出來，就可以看見滿滿的Observable Method

## 結語

這篇文章只概述了Rx'Library核心，其餘還有很多強大的功能，能幫助我們寫出漂亮的Function Reactive Programming
有興趣的讀者可以去[RxJS Marbles](http://rxmarbles.com)自己拖拉體驗一下

***

## Reference

[30天精通Rx JS](http://ithelp.ithome.com.tw/articles/10186104)
[TB-Reactive Programming 簡介與教學(以 RxJS 為例)](http://blog.techbridge.cc/2016/05/28/reactive-programming-intro-by-rxjs/)
[Rx JS Marbles](http://rxmarbles.com)