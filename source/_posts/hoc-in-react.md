---
title: 使用HOC
date: 2017-10-16 17:45:28
tags: [React, FP]
author: tancc
---

以往當我們要對component進行功能上的擴充時，以前常見的方法如`mixin`、`inheritance`等等，但現在都不推薦使用這些方式了。

用`inheritance`的話，會使得複用性變低，且當你只想要簡單的功能時，所繼承的物件背後可能相依著一大串東西。如一句名言[“You wanted a banana but you got a gorilla holding the banana”](https://www.johndcook.com/blog/2011/07/19/you-wanted-banana/)。

`mixin`較大的問題在於不具有immutability，會去修改到原有的component。當在測試或是發生問題時，我們難以確認當前的狀態是component本身所導致的抑或是mixin的override所造成的。

# 什麼是 Higher-Order Component

是從 [Higher-order function](https://en.wikipedia.org/wiki/Higher-order_function) 這名字而來的

而 Higher-Order Component 可簡單解釋為：

> a higher-order component is a function that accepts a component, and returns a new component that wraps the original

也就是一個function，他的輸入是一個component，輸出一個新的component。

簡單的形式大概會長這樣

```javascript
const hoc = WrappedComponent => class extends Component {
    // do something...
    render() {
        return <WrappedComponent {...this.state} {...this.props} />
    }
}
```

# 例子

我們常會有一些頁面是有鎖權限的，例如只有登入的使用者才能進入，若沒有登入則跳到登入畫面

假設現在有兩個頁面(Account, Setting)是必須登入才可看見，在沒有用HOC時，最直覺的方法會是這樣寫

```javascript
// Account
import React, {Component} from 'react';
import Login from 'components/Login'
export default class Account extends Component {
    render() {
        const {user} = this.props
        if (user.isLogin) {
            return <div>Account</div>
        } else {
            return <Login />
        }
    }
}
// Setting
import React, {Component} from 'react';
import Login from 'components/Login'
export default class Setting extends Component {
    render() {
        const {user} = this.props
        if (user.isLogin) {
            return <div>Setting</div>
        } else {
            return <Login />
        }
    }
}
```

可以發現明顯會有兩個問題：

- 檢查是否登入的邏輯重複了。這使得之後如果要改變驗證邏輯時，必須去更改每個component做了許多重複的步驟，也使得產生bug的機會變多了。

- component除了畫面呈現的樣子外，還需額外關心”何時要呈現”這件事。可以想像到的是，若是邏輯變多且沒有抽出時，component的程式碼會變得相當龐大，會難以維護、debug。比較好的做法是component只關注在畫面上，其餘邏輯部分抽出。

# 使用 HOC

同樣實作上述例子，但這次用HOC去改寫，將確認是否登入的邏輯抽出。

```javascript
// authorized.js
import React, {Component} from 'react';
import Login from 'components/Login'
const authorized = WrappedComponent => class extends Component {
    render() {
        const {user} = this.props
        if (user.isLogin) {
            return <WrappedComponent {...this.props} />
        } else {
            return <Login />
        }
    }
}
export default authorized
// Account.js
import React, {Component} from 'react';
export default class Account extends Component {
    render() {
        return <div>Account</div>
    }
}
// Setting.js
import React, {Component} from 'react';
export default class Setting extends Component {
    render() {
        return <div>Setting</div>
    }
}
// 使用方法
const authorizedAccount = authorized(Account)
const authorizedSetting = authorized(Setting)
// ...
```

# 優點

從上面例子可以看到幾個優點

- 共用邏輯抽出，增加復用性、減少重複代碼
當把邏輯抽出時，可以省掉許多重複的代碼。且當我們的認證方法改變或是出問題時，只須去更動或檢查`authorized.js`這檔案即可，在維護以及修改上方便許多。

- component專注在畫面呈現上

- 使用composition的方式，返回全新的component，具有immutability
- 較易測試
測試component時就只需測試他的畫面正確性，而不需考慮其他如商務邏輯的部分。而測商務邏輯就直接測其邏輯的正確性，不需考慮畫面的呈現。
且因為具有immutability的特性，測試時不會有如文章開頭所寫到的使用mixin，因為mutable state所造成的問題。

- 程式較具有彈性
當有其他的功能需要加上去時，我們只需要去定義各種對應的HOC，藉由函數的組合就能達到想要的效果，如`compose(hocA, hocB, hocC)(Comp)`，也就是FP中常提及的`function composition`。

# 何時使用
當在撰寫component時，發現了許多重複的程式碼，如重複的邏輯條件判斷或是重複的前置動作等等，或是夾雜了畫面以外的商業邏輯時，就代表是可以嘗試使用HOC做進一步抽象的時機了！

許多library也都有用到HOC這技巧，常見如`react-redux`中的`connect`、`react-router`中的`withRouter`，以及`recompose`等等。

# 參考資料

[Higher Order Components in React](https://www.sitepen.com/blog/2017/08/15/higher-order-components-in-react/)