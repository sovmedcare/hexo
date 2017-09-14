---
title: Immutable.js 簡介
date: 2017-09-08 15:15:02
tags: [FRP,React]
---
Javascript中的對象是可變的(Mutable)

``` javascript
let task = { id : 1, status : "Load" };
let task2 = task
task2.id = 2 // task.id 也是 2
```

我們可以輕易地修改對象，這在專案複雜後是種隱患
雖然可以用Object.assign()來解決，但當assign的對象比數不小時，效能會變差
Immutable.js很好的解決這些問題
***

# 原理

Immutable.js背後有兩個重要的原理

* **Persistent Data Structure**

經由old-data創建new-data時，old-data可用且不可變，這也是**Functional Programming**重要概念-沒有副作用(Side Effect)

* **Structural Sharing**

當有某個節點Update時，不會整個Copy一份，只會新建需要的變動的部分，其餘參照原本的
這大幅的優化效能且也節省了記憶體
![Sharing](https://goo.gl/BdnHfB "取自「Immutable详解及React中实践」")

## 優點

### 降低Mutable帶來的複雜度

``` javascript
const mutableFn = data => {
    doSomething(data); // data = {id:1,value:2}
    console.log(data.value) // data.value is ?
}
```

在有Mutable的可能時，我們無法保證data不變，但用Immutable的話就可以確認依然為2了

### 節省記憶體

因為有Structural Sharing，許多對象可以被重複使用，沒用到的也會自動回收
以下截選[camsong的文章](https://github.com/camsong/blog/issues/3)中的一段Code，能清楚表達Structural Sharing優點

``` javascript
import { Map } from 'immutable';
let a = Map({
  select: 'users',
  filter: Map({ name: 'Cam' })
})
let b = a.set('select', 'people');

a === b; // false
a.get('filter') === b.get('filter'); // true
```

# 結語

在專案日漸複雜龐大時，Immutable.js能有效地提升效能和維護性，但專案較小且不複雜時或許就不一定要用它了
經由Immutable.js建的對象並不是原生Javascript對象，要特別注意，所以要操作幾乎都是要用Immutable.js的Method

***

# Reference

[Immutable.js, persistent data structures and structural sharing](https://medium.com/@dtinth/immutable-js-persistent-data-structures-and-structural-sharing-6d163fbd73d2)
[Immutable詳解及React中實踐](https://github.com/camsong/blog/issues/3)