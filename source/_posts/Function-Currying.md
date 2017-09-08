---
title: Function Currying
date: 2017-09-08 13:48:30
tags: [FRP]
---
![Haskell Curry](https://camo.githubusercontent.com/b51957f9615c4df77574d0bb393ac6fbce50acd7/687474703a2f2f73332e616d617a6f6e6177732e636f6d2f6c7961682f63757272792e706e67 "Haskell Curry (Google Image)")
函數「柯里化」這單看字面上意思真的猜不出想表達什麼，因為Curry一詞其實是取自人名
**[Haskell Curry](https://en.wikipedia.org/wiki/Haskell_Curry)**-偉大的邏輯學家，Haskell這語言命名也是取自於他
名字有了含意後我們來瞭解其中
***
## 概念
經過柯里化的Function會回傳 **Function**||**Value**
[維基百科](https://zh.wikipedia.org/wiki/柯里化)是這麼解釋
>如果你固定某些參數，你將得到接受餘下參數的一個函數

解釋的有點饒口，所以我們往下來看點範例
## 範例
接下來會使用到[Ramda](http://ramdajs.com)來協助我們進行Curry
```javascript
const addNumber = (x,y) => x + y; 
addNumber(1,2) // 3
```
我們要呼叫addNumber就是傳入兩個參數進去，那如果想要一個Function固定會將傳進來的值加100呢？以下
```javascript
const addNumber = R.curry( (x,y) => x + y ); // 柯里化
const add100 = addNumber(100)
add100(1) // 101 
```
那如果參數不只兩個呢？
```javascript
const addNumber = R.curry( (x,y,z) => x + y + z ); 
const add100 = addNumber(100)
add100(1)(2) // 103 
```
以上是比較簡單的應用，我從[ScottSauyet的文章](http://fr.umio.us/favoring-curry/)截選幾段程式來演示Curry的好處
#### 原始
``` javascript
getIncompleteTaskSummaries = function(membername) {
    return fetchData()
        .then(function(data) {
            return data.tasks;
        })
        .then(function(tasks) {
            var results = [];
            for (var i = 0, len = tasks.length; i < len; i++) {
                if (tasks[i].username == membername) {
                    results.push(tasks[i]);
                }
            }
            return results;
        })
        .then(function(tasks) {
            var results = [];
            for (var i = 0, len = tasks.length; i < len; i++) {
                if (!tasks[i].complete) {
                    results.push(tasks[i]);
                }
            }
            return results;
        })
        .then(function(tasks) {
            var results = [], task;
            for (var i = 0, len = tasks.length; i < len; i++) {
                task = tasks[i];
                results.push({
                    id: task.id,
                    dueDate: task.dueDate,
                    title: task.title,
                    priority: task.priority
                })
            }
            return results;
        })
        .then(function(tasks) {
            tasks.sort(function(first, second) {
                var a = first.dueDate, b = second.dueDate;
                return a < b ? -1 : a > b ? 1 : 0;
            });
            return tasks;
        });
};
```
#### 處理後
``` javascript
var getIncompleteTaskSummaries = function(membername) {
    return fetchData()
        .then(R.get('tasks'))
        .then(R.filter(R.propEq('username', membername)))
        .then(R.reject(R.propEq('complete', true)))
        .then(R.map(R.pick(['id', 'dueDate', 'title', 'priority'])))
        .then(R.sortBy(R.get('dueDate')));
};
```
這差異就很明顯了，這也是為何柯里化在Function Programming如此重要
## 總結
Currying是促使Function Programming簡潔的原因之一
我們也可以經由上面「處理後」的程式碼發現這樣更語意化，明白每一步在做的事情
***
## Reference
[Favoring Curry](http://fr.umio.us/favoring-curry/)
[Curry化 - Javascript Functional Programming 指南](https://jigsawye.gitbooks.io/mostly-adequate-guide/content/ch4.html)
