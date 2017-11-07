---
title: React PureComponent and Performance
date: 2017-10-12 10:04:16
tags: [React, javascript]
author: Tu-Szu-Chi
---

React在15.3.0新增了`PureComponent`的類別，我們基本上可以直接無痛從`extends Component`轉到`extends PureComponent`
大部分情況下直接使用**PureComponent**會是較方便的做法，省去了自己寫**shouldComponentUpdate** (自己寫的話還會報錯)
但也不代表用了**PureComponent**整個效能就會提升，根本的療法還是要將Component寫的`pure-render`，意指假如傳給Component的**prop** & **state**都是一樣，那**vDOM**回傳的結果也該是一樣

# Non-pure

最常見的non-pure寫法就像是以下

```js
// 每次render傳進去的都是新的Object
<Button style={{color: 'red'}} />

<Children data={{x: 1, y: 2}} />

// inline-function, 每次render傳進去的都是新的Function
<Button2 onClick={e => console.log(e)} />
```

這樣的**inline**寫法，雖然會造成每次一有變動，每個`Children Component`都會去檢查一次，很多東西要檢查「理論上」耗費的時間也會變多，但我們真的就該改掉這樣的寫法嗎？[這篇文章](https://cdb.reacttraining.com/react-inline-functions-and-performance-bdff784f5578)給了我們些值得探討的問題，程式變慢真的是這些**non-pure**造成的嗎？

# 實驗

我在[Github](https://github.com/sovmedcare/react-purecomponent-test)上開了一個用來實作`PureComponent`的專案，有興趣的人也可以直接到[這](https://sovmedcare.github.io/react-purecomponent-test/)打開log看看
其中的一個結論是

> 當傳給`Children Component`的props「是」pure，更新children會觸發children的ComponentDidUpdate & render，但畫面上不會有任何改變，真的被skip掉了；而`Parent Component`的更新，並不會觸發children的ComponentDidUpdate & render

# 結語

***「不要過早開始優化」***，這是文章中作者很強調的一點，當你要做優化時，要好好做實測；是**inline-style**造成的嗎？還是有更大的問題在Code裡？
優化沒辦法一次到位，從使用的過程中逐漸察覺出可加強的部分，首要關注的還是專案本身的產出
[官方](https://reactjs.org/docs/react-api.html#reactpurecomponent)也有建議到，因`PureComponent`做的是**shallow compare**，所以較複雜的data structures可以用`immutable objects`來加快比對速度*(ex. Immutable.js)*

# Reference

[React, Inline Functions, and Performance](https://cdb.reacttraining.com/react-inline-functions-and-performance-bdff784f5578)
[React PureComponent 源码解析](https://segmentfault.com/a/1190000006741060)
[我的實驗](https://github.com/sovmedcare/react-purecomponent-test)