---
title: 使用Immutable.js
date: 2017-10-17 18:26:55
tags: [React, FP]
author: Tancc
---

# 解決的問題

- mutable state
JavaScript中，變數的值預設是mutable的，所以在程式中的數值、狀態等等都是可變的。mutable這性質常會導致非預期的結果，而通常導致某數值或是狀態改變的地方太多了難以掌握，所以在追蹤和維護那些可變的狀態時，就是非常費工的任務了。

- 物件比對不便
JavaScript中在做相等判斷時通常會用`===`，但若比較的對象是物件，這只有做到shallow compare，無法知道其內層是否相等，需要自己再去做deep compare。但這過程就比較複雜且效能很差。

- `Object.assign`的效能問題
為了解決上述的問題，一個方法就是想辦法擁有immutability。為了達到此目的，原生JavaScript能使用的方法有幾個：`Object.freeze`，但會使得許多操作變得更加複雜麻煩；
在做有關物件改變的操作時使用`Object.assign`或是`spread operator`來返回全新的物件，間接得到物件的不可變性。但是`Object.assign`等方法需要做許多複製的行為，當物件龐大時效能問題就會出現了。

# 特色及優點

- persistent data structures

對資料做改動時不會修改到資料本身，會保留舊資料回傳新資料。也就是FP中常提及的`immutability`的概念，所以少了`side-effect`所造成的困擾。

- 使用Trie結構存放資料

[Trie資料結構的介紹](https://en.wikipedia.org/wiki/Trie)。因為使用了`Trie`所以在資料比對上十分便利，只需檢查hashcode是否相同即可，不用做如`deepCompare`這類耗資源的方式。

- structural sharing

![structural_sharing](https://cdn-images-1.medium.com/max/400/1*zYtOUrDIj0_-vLoBTbQGUQ.png)

不像純JS的`Object.assign`的方式，將修改的物件整份複製。使用`Immutable.js`時，只有修改到的地方的節點會重新產生，而沒有變動的節點會共用。因此可以節省許多記憶體

- 方便的API
提供了許多API如`updateIn`、`setIn`、`getIn`等等，讓我們在對巢狀資料的操作上方便許多

# shouldComponentUpdate

在開發React專案時，`immutable.js`是非常好用的工具。由於re-render是很耗效能的行為，所以常需要去撰寫`shouldComponentUpdate`中的比對邏輯，減少re-render的次數。
使用純JavaScript的話，須去寫許多客制的比較邏輯，難以避免的還有deep compare行為，這會使得光是判斷物件相等與否，就耗掉了需多運算資源。
而使用`immutable.js`的話，使用shallow compare就可以滿足所有需求了。因為在`immutable.js`中，若是物件彼此內部不同，必定會是不同的物件。


# 參考資料
[Immutable.js, persistent data structures and structural sharing](https://medium.com/@dtinth/immutable-js-persistent-data-structures-and-structural-sharing-6d163fbd73d2)