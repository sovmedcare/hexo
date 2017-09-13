---
title: RxJs Observable - HOT & COLD
date: 2017-09-13 12:15:25
tags: [RxJS,FRP]
---
![Observable Hot & Cold](https://image.slidesharecdn.com/untitled-150216041015-conversion-gate02/95/reactive-programming-with-rx-22-638.jpg?cb=1424081878 "截自Google Image")

Rx'Library Observable可大致分為兩種類型 - *Hot* & *Cold*
在往下介紹前，可以先將*Hot*想成廣播電台(Radio)，*Cold*是CD
Hot Observable的訂閱者，會讀取同樣的source；Cold Observable則每個訂閱者的source都是獨立的

# Cold Observable

只有當訂閱者訂閱的時候，Cold Observable才會推送資料，且每個訂閱者都是獨立的流，就像是CD只有當你放進去按播放後才會有音樂
Observable.interval就是種Cold類型的流

``` javascript
const source = new Observable.interval(100).take(3);
```

可以在這[JS Bin](https://jsbin.com/loqolobapu/1/edit?js,console)看到observerA & observerB是各自的數據流