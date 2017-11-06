---
title: Currying介紹
date: 2017-10-16 17:45:28
tags: [FP]
author: Tancc
---

[節錄自wiki](https://en.wikipedia.org/wiki/Currying)
> currying is the technique of translating the evaluation of a function that takes multiple arguments (or a tuple of arguments) into evaluating a sequence of functions, each with a single argument.

舉例來說，若是有一函式`gn2`，他的input為某一需要多個參數的函式`fn`，output為需要一個參數的函式`fn2`而此函式亦回傳需要一個參數的函式…，直到原函式fn的參數皆被滿足。則`gn2`所做的就是`currying`函式`fn`。

```javascript
const curry = fn => {
    const getCurriedFn = prev => arg => {
        const args = [...prev, arg]
        return args.length < fn.length ? getCurriedFn(args) : fn(...args)
    }
    return getCurriedFn([])
}

const add = (a, b, c) => a + b + c
const curriedAdd = curry(add)
const add2 = curriedAdd(2)
const add2And3 = add2(3)
console.log(add2And3(4)) // 9
```

# 優點
- 搭配partial application，寫出更泛用、復用性高的函數
若函數支援currying的話，我們可以只先放入某幾個參數(`partial application`)來製造出可以被其他情境使用的函數。而不用每次都把函數寫死，傳入完整的參數。
```javascript
const add2 = add(2) // 用在需要加2的地方
const splitAt5 = splitAt(5) // 將字串從第5個字元處切開
const doubleList = map(x => x * 2) // 將陣列中數字都兩倍
...
```

- 更為簡潔易讀的函數
因為搭配了`partial application`，只需傳入幾個特定的參數，可以寫出較為語易化且簡短的函數。
由上述可以看出，這種方式得到的函數，我們可以很清楚從字面上知道他想做什麼。

- 更方便去做 function composition
我們寫程式時，會只針對問題的各個小流程去定義函數。由於每個函數可被定義的很語易化，能清楚了解每個步驟的意圖，再藉由函數組合就能達到想要的結果

```javascript
// 簡單範例 沒太大意義...
const joinWithDash = join('-')
const strEq = eqBy(String);
const uniqByStr = uniqWith(strEq)

const transStr = compose(
    joinWithDash,
    uniqByStr,
    reverse
) // 先reverse，將重複的字刪除再用'-'連結

map(transStr)(['apple', 'banana', 'cat', 'door']) // ["e-l-p-a","a-n-b","t-a-c","r-o-d"]
```

# 使用關鍵

- 撰寫函數時，欲操作的資料要放在最後一個參數
這也是[Hey Underscore, You're Doing It Wrong!](https://www.youtube.com/watch?v=m3svKOdZijA)中提到的`Underscore`不好的地方，他所提供的函數將要操作的資料放在第一個參數，使得無法將其currying後去做composition。
所以較好的方式是，撰寫函數時能把要操作的那個變數，放在最後參數的最後一個。所以較方便我們去撰寫`pointfree style`的函數，然後去做`function composition`。

# 相關資料

[Hey Underscore, You're Doing It Wrong!](https://www.youtube.com/watch?v=m3svKOdZijA)