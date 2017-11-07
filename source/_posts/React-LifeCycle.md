---
title: React LifeCycle
date: 2017-09-04 15:43:08
tags: [React]
author: Tu-Szu-Chi
---
![React LifeCycle](https://imgur.com/3JSxD1b.png)
圖為React主要的**生命週期**LifeCycle流程，以下內容將針對各週期的觸發時機以及使用場合做解說

***

# componentWillMount

* **時機**
所有DOM加載之前

* **場合**
可在root-component做連結外部API的設定，ex.Firebase config

不建議在此做**this.setState**，因**setState**是屬異步(Async)操作，不能保證能立即執行完畢
設Default state時，官方也建議寫在**constructor()**裡
整體來說**componentWillMount**是較少被用到的LifeCycle，可選擇將Code寫在**constructor()**中

``` javascript
constructor(props){
    super(props)
    // Do something...
}
```

# componentDidMount

* **時機**
所有DOM加載之後

* **場合**
Call API

建議有需要做AJAX API皆可寫在**componentDidMount**裡
因Call API後通常會牽涉到**state**來操作，在此實作AJAX可確保所有Component皆已加載

# componentWillReceiveProps

* **時機**
接收到新的props前

* **場合**
比較**this.props** & **nextProps**

```javascript
componentWillReceiveProps(nextProps){
    (nextProps.id !== this.props.id) &&
        this.setState({LoadingData:true})
        // Get data by nextProps.id ...
}
```

以上範例表示說當新的**props.id !== this.props.id**時，我們就要去獲取新的Data
且**componentWillReceiveProps**在初始化**(initial props -> mounting)**時，並不會呼叫此LifeCycle

# shouldComponentUpdate

* **時機**
有新的props || state

* **場合**
回傳true或false決定Component是否要**re-render**

>(nextProps , nextState) => **Boolean**

雖然避免部分做多餘的**re-render**看似能提升效能，但據說React團隊認為不該輕易使用它
尤其當你使用了後，卻察覺不出提升的效益，那就不要去用它
更多的資訊可參考這篇*[什麼時候要使用shouldComponentUpdate](http://www.infoq.com/cn/news/2016/07/react-shouldComponentUpdate)*

# componentWillUpdate

* **時機**
有新的props || state 且 **shouldComponentUpdate** return true

* **場合**
和componentWillReceiveProps相似
此LifeCycle較少用到，且在此不能使用到**this.setState**，這會造成不斷的輪迴週期

# componentDidUpdate

* **時機**
Component update後

* **場合**
Call API when component updated

此處和**componentDidMount**也有點相似，官方也建議有需要在Component更新後做的Network request寫在這
也可以做些需要reset的動作

# componentWillUnmount

* **時機**
Component將卸載

* **場合**
Cancel network request / Remove event listeners

最常觸發的時機應為**跳頁**時，如有**setTimeout**或**setInterval**的network request記得在此取消掉
或是任何在**componentDidMount**建立的DOM也在此移除

***

## Reference

*[官方文檔](https://facebook.github.io/react/docs/react-component.html)*
*[React Lifecycle Methods](https://engineering.musefind.com/react-lifecycle-methods-how-and-when-to-use-them-2111a1b692b1)*
*[什麼時候要使用shouldComponentUpdate](http://www.infoq.com/cn/news/2016/07/react-shouldComponentUpdate)*