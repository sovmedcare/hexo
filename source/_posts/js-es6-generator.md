---
title: Javascript ES6 Generator 介紹
date: 2017-09-27 17:42:49
tags: [javascript]
---
在ES6中，Promise將我們從「**Callback Hell**」稍微解放出來

```js
asyncFun1(() => {
  // do something...
  return result
})
.then(asyncFun2)
.then(asyncFun3)
// ...more and more asyncFun
```

但Promise也有缺點，例如當**asyncFun3**需要調用**asyncFun1**的result時，該怎做呢？可能會用巢狀式Promise，或者用變數暫存每一次的result之類，我們都知道這些不是個好方法

# Generator

**generator**可讓我們的異步流程寫得更synchronous-style，免除了之前的困擾
先用個範例將Promise小改成Generator

```js
function* logGen(name) {
  const url = 'https://api.github.com/users/' + 'name'
  const result1 = yield fetch(url)
}
const gen = logGen('sov')

const result = gen.next()
result.value
  .then(data => data.json())
  .then(data => gen.next(data))
```

*Ln:7*的result會收到*Ln:3* yield回傳的Promise
待API回傳後，*Ln:10* **gen.next(data)** 會將API回傳的data賦值給result1

但我們不可能一直*.then*下去，這樣又回到地獄了，所以我們需要一個Function幫我們管理這整個流程，可以自動接著下去做事的傢伙
[Co](https://github.com/tj/co) - 知名的TJ大神在幾年前就實現了Generator概念，有興趣的可以參考他的實踐方式，在此我們先寫一個簡易版的管理器用用

```js
const co = (gen, parameter) => {
  const g = gen(parameter)

  next = (data, err) => {
    let res

    if (err) {
      return g.throw(err)
    } else {
      res = g.next(data)
    }
    if (!res.done) {
      res.value
        .then(data => data.json())
        .then(data => { next(data) })
    } else {
      console.log('DONE! ',res.value)
    }
  }

  next()
}
```

這簡易版的可以協助我們連續的Promise請求，可在這[JS Bin](https://jsbin.com/puxurihewi/2/edit?js,console) Run看看，在此也附上我設定的「任務」Code

```js
function* logGen(name) {
  const url = 'https://api.github.com/users/'

  const result1 = yield fetch(url + name)
  const result2 = yield fetch(url + 'abc')
  const result3 = yield fetch(url + 'def')

  return result1.id + '~' + result3.id
}

co(logGen, 'sov')
```

但這樣的流程管理器只能處理都是*yield Promise*的需求，固我們可以加個工具幫我們判斷是否為Promise

```js
const isPromise = obj => Boolean(obj) && (typeof obj.then === 'function')
```

將*isPromise*加入到我們的流程管理器 - [JS Bin](https://jsbin.com/niguxijuye/1/edit?js,console)

```js
if(!res.done) {
  isPromise(res.value)
    ? res.value
        .then(data => data.json())
        .then(data => { next(data) })
    : next('It is not Promise')
} else {
  console.log('DONE! ',res.value)
}
```

>對應任務的不同再將流程管理器改造一下就可以用了

# 進階 (Observable-style)

接下來這章節主要是想將Generator的Iterable-style寫成Observable-style的方式

![GTOR](https://cdn-images-1.medium.com/max/1400/1*7ZdFWFlA9dSRCv2naCjihA.png "Kris Kowal's - A General Theory of Reactivity")

這表格來自於*Kris Kowal's* - [GTOR: A General Theory of Reactivity](https://github.com/kriskowal/gtor)，比較不好理解的部分是Iterable/Observable的差別
>Iterable是**pull** value，Observable是**push** value

## Interable

Generator通過**next()**來pull value

```js
const gen = function* () {
  yield 1;
  yield 2;
  yield 3;
}

const iterator = gen();

iterator.next(); // { value: 1, done: false }
iterator.next(); // { value: 2, done: false }
iterator.next(); // { value: 3, done: false }
iterator.next(); // { value: undefined, done: true }
```

## Observable

而所謂的Observable通常是指Value「可使用」後，會有通知/訂閱機制(subscription/notification)將Value送出去
在此我們要將流程管理器抽象化成Observable模式

```js
const co = (gen) => (...args) => ({
  subscribe: (onNext, onError, onCompleted) => {
    next(gen(...args), {onNext, onError, onCompleted})
  }
})
```

co管理器多了*`.subscribe()`*，就跟Rx'Library中的Observable一樣，並且接收三個Function作為參數
原本的迭代器`next()`也要改動一下
> 因為是簡易版，所以沒有加上errorHandler

```js
const next = (iter, callbacks, prevData = undefined) => {
  const { onNext, onCompleted } = callbacks
  const res = iter.next(prevData)
  const value = res.value

  if(!res.done) {
    if(isPromise(value)) {
      value
        .then(data => data.json())
        .then(data => {
          onNext(data)
          next(iter, callbacks, data)
        })
    } else {
      onNext(value)
      next(iter, callbacks, value)
    }
  } else {
    return onCompleted()
  }
}
```

這樣當每次Value「可使用」後，我們會直接push到`onNext()`去，這邊任務的範例是會接收一個Array of name，去call API得到這個name的Github user

```js
function* logGen(...names) {
  const url = 'https://api.github.com/users/'

  for(let i = 0; i < names.length; i++) {
    yield fetch(url + names[i])
  }
}
```

再來是產生一個`偽-Observable`對象

```js
const asyncFun = co(logGen)

asyncFun('sov','abc', 'ww', 'tt')
  .subscribe(
    (val) => console.log('Success: ', val.login + '-' + val.id),
    (err) => console.log('Error: ', err),
    () => console.log('Completed!')
)
```

這樣就完成了(完整的[JS Bin](https://jsbin.com/xusekusaka/1/edit?js,console))!而這樣做的好處就是...
> 你會發現RxJS原來這麼好用

# 結語

其實`co()`在這就是個「迭代器(Iterator)」角色，讓我們不用一直像以下這樣做

```js
generatorFun.next()
generatorFun.next()
generatorFun.next()
// ... more and more next()
```

> 也可以直接看Co.js的[原始碼](https://github.com/tj/co/blob/master/index.js)，並不複雜

看到這裡，希望你對Generator有更深的認識，或許也會對`ES7 async/await` 和 `RxJS`存在的用意更有感觸~

# Reference

[The Hidden Power of ES6 Generators](https://medium.com/javascript-scene/the-hidden-power-of-es6-generators-observable-async-flow-control-cfa4c7f31435)
[Teaching RxJS](https://www.ericponto.com/blog/2016/12/05/teaching-rxjs/)
[初探ES6 Generators](http://www.codedata.com.tw/javascript/es6-3-generator/)