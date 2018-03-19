---
title: 什麼是 High Order Component
date: 2017-09-06 09:39:43
tags: [React,FP]
author: Tu-Szu-Chi
---
``` haskell
hoc :: ReactComponent -> ReactComponent
```

**HOC**(High Order Component)帶有點Function Programming概念
它本質上就是一個Function，傳入一個(或多個)React.Component，再回傳新的React.Component

# HOC在做什麼

如果有接觸過**Redux**應該有聽過**Presentational / Container Component**
Container Component主要負責**「怎麼做事情」**
![Presentational / Container Component](https://cdn-images-1.medium.com/max/1600/1*tIdBW-TqotpALD3b2xk3SA.gif "截自Tom Coleman:Understanding Higher Order Components")
[Tom Coleman](https://medium.freecodecamp.org/understanding-higher-order-components-6ce359d761b)提到一項Container Component很重要的職責
就是**負責將Global State分派給底下的Child Component**
而Redux中的**connect()**就是HOC的一種，通常我們都會用

* mapStateToProps 將需要的state轉換成Presentational的props
* mapDispatchToProps 注入需要的callback行為(ex. onClick、onHover時)

``` javascript
const mapStateToProps = (state, ownProps) => {
}
const mapDispatchToProps = (dispatch, ownProps) => {
}
```

當各個Container都負責整個Application的一塊，大家有各自的職責並做好本份就會讓整個架構很有效率

# HOC為何特別

>A recent trend in React is towards **functional stateless components**. 
>These simplest “pure” components only ever transform their props into HTML and call callback props on user interaction.

以上截自*Tom Coleman*文章中的一段話，可表達出為何要分出Presentational / Container
當將「複雜的活兒」交給特定的Container做，Presentational只管「顯示」&「拿取」，這樣碰上Bug時我們就知道該~~解決誰~~找誰來解決了

Component幾乎都需要存取Global State中的某一部分，但不能隨意地讓每個Component都可以調用，所以HOC形成了一種**約束**，由它來分配

而Container也取代了原生的mixins，可在網路上找到許多比較兩者的文章

# 結語

**High Order Component**的出現也間接表明React團隊往**Functional Programming**靠攏
Javascript也有些Library可幫我們的Code更Functional **ex.**[Ramda.js](http://ramdajs.com)
搞懂**Functional Programming**的思維，以現在來看是件值得投資的事

***

# Reference

*[Understanding Higher Order Components](https://medium.freecodecamp.org/understanding-higher-order-components-6ce359d761b)*
*[Presentational and Container Components](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0)*
*[初識React中的High Order Component](https://leozdgao.me/chushi-hoc/)*