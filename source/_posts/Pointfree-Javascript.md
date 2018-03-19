---
title: Pointfree Javascript
date: 2017-09-08 10:46:00
tags: [FP]
author: Tu-Szu-Chi
---
截自[Haskell wiki](https://wiki.haskell.org/Pointfree)
>**Pointfree Style**
>It is very common for functional programmers to write functions as a composition of other functions, never mentioning the actual arguments they will be applied to

Pointfree這種風格在中文也譯作「無值」、「隱含式」
以下範例多會使用 **[Ramda](http://ramdajs.com)** 這函式庫來協助我們寫成Functional Programming
p.s. Ramda中的方法多是Curry化，還不懂Curry化的可以參考[這篇文章](https://jigsawye.gitbooks.io/mostly-adequate-guide/content/ch4.html)

***

# 程式的本質

即是在解決問題，而最常碰到的問題形式多為
**-->輸入-->運算-->輸出-->**
運算我們可以先看作是個fn(Function)
**-->a-->fn-->b-->**

```javascript
fn = a => b
```

fn這Function接收一個a參數(Input)，並回傳b(Output)
當運算可能比較複雜時，流程會變成
**-->a-->fn1-->(m)-->fn2-->(n)-->fn3-->b**
m,n分別表示經過fn1,fn2出來的結果，我們可以用Ramda來寫成

```javascript
fn = R.pipe(fn1, fn2, fn3)
```

R.pipe會將參數先傳入到fn1，得出的Output在傳入fn2依序下去，與之相對的是R.compose

# Pointfree概念

經過以上例子的演示，我們可以用更語意化的例子會比較清楚

```javascript
const addOne = x => x + 1;
const square = x => x * x;
const addOneThenSquare = R.pipe(addOne,square);
addOneThenSquare(2) // 9
```

先將要處理的流程拆解出來，並先定義好，再組合起來，這就是**Pointfree**的核心概念
>定義的時候不使用所要處理的值，只合成運算過程

addOneThenSquare也可以寫成這樣，但就是不是Pointfree Style且也沒那麼簡潔&易讀

```javascript
const addOneThenSquare = x => R.pipe(addOne,square)(x);
// 或是
const addOneThenSquare = x => square(addOne(x));
```

更多的範例可以參考[阮一峰老師的文章](http://www.ruanyifeng.com/blog/2017/03/pointfree.html)

# 結語

**Pointfree**這種風格剛開始會不太習慣，無法直接看到Function本身的參數，但大部分情況可以通過註解來輔佐
畢竟**Pointfree**不單只是讓我們的Code更簡潔、語意化
我們會從原本的「面向對象」(data.id,data.name...)轉而更專注於「處理」的動作(map,reduce,filter)
這樣會更瞭解整個流程在做什麼，取代用許多for-loop,if-else的嵌套
而且拆成各個「單一職責」的Function也較好做Testing & Debug

***

# Reference

[Pointfree編成風格指南 by 阮一峰](http://www.ruanyifeng.com/blog/2017/03/pointfree.html)
[Thinking in Ramda (Pointfree Style)](https://zhuanlan.zhihu.com/p/27626482)
[Pointfree Javascript](http://lucasmreis.github.io/blog/pointfree-javascript/)