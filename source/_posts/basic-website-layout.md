---
title: 用 CSS 完成常見網頁版型
subtitle: hello
date: 2020-08-25 12:45:22
tags: [CSS]
author: dannnyliang
---

想要在網頁上達成特定的視覺效果，做法很多  
而且建議同樣的結果，可以多學幾種做法，當情境限縮選擇時，至少還有其他路可以選擇  
但，也不是所有做法都值得學習
> 做法仍有好有壞

<br />

# 什麼是較好的做法？
- **不用寫太多**  
屬性精簡：`margin-top`, `margin-right`, `margin-bottom`, `margin-left` 可被精簡成 `margin`
- **易讀**  
相關的樣式集中、不分散在其他元素
- **具 RWD 性質**  
在裝置寬度改變時，仍保持原比例
- **具擴展性**  
元素被重複使用，或是有其他元素加入時，原本的樣式不需要改動
- **樣式不相依**  
元素本身要達成的效果，不需要其他元素配合，也不影響其他元素  

本篇主要會針對樣式的「擴展性」、「相依性」進行討論，找出即使介面設計修改，仍能穩定的樣式寫法

<br />

# 遇到的問題
{% asset_img layout-wireframe.png layout-wireframe %}  

**需求：**
```
- Header1, Header2 固定出現在上方
- Header1 向下捲動時收合，向上捲動時展開
```

**實作：**  
1. 建立一個 state 紀錄 `Header` 高度
2. 頁面渲染、滾動事件觸發時，更新此 state
3. 將 state 傳到 `Header` 元件、`Content` 元件

這樣的實作會出現些問題：
- 需 state, props，且若無 styled-component 配合會更難達成
- 需多個元件配合才能達到效果
- 更新邏輯複雜，容易有 bug

# 練習 I - Fixed Header
先不用想得那麼複雜，從一個簡單的範例開始 ([CodeSandbox](https://codesandbox.io/s/before-fixed-header-s8nlj))

**需求：**
- (Required) `Header` 固定在畫面上方不動
- (Required) 其他的內容可上下捲動
- (Optional) 擴展性：多複製一個 `<Header />` 時，仍符合需求
- (Optional) 低相依：`Content` 元件的樣式，不需要使用 `menuHeight`

<br />

## 做法1 - Pass Height
([CodeSandbox](https://codesandbox.io/s/after-fixed-header-pass-height-szpqp))  
- 複製 `<Header />` 時，會跟原本的 `Header` 重疊，沒有擴展性
- `Content` 的樣式需要知道 `Header` 的高度，相依度高
- `Header` 的背景有透明度的話會發現，`Content` 捲動時會跑到 `Header` 的下面  
如果這是其中一項視覺需求，雖然沒達成 Optional 項目，但仍可考慮採用此作法

<br />

## 做法2 - Seperate Area
([CodeSandbox](https://codesandbox.io/s/after-fixed-header-separate-area-b6eb3))  
- 在 `Content` 外多包一層 `Wrapper`，用來定義 `Content` 存在的空間，如此一來 `Content` 就不需要知道 `Header` 的高度，可以達到低相依  
- 但複製 `<Header />` 時，仍需調整 `Wrapper` 的實作，擴展性仍不佳

<br />

## 做法3 - Grid
- grid 的強項除了二維排列以外，也能輕鬆規劃版型  
- 作法2 實際上就是先規劃區塊，再將元素放入個區塊中，因此用 grid 可以輕鬆重現
- 但是缺點和作法2 相同

<br />

## 做法4 - Flex
([CodeSandbox](https://codesandbox.io/s/after-fixed-header-flex-fwge4))  
- 一樣是從「先規劃區塊」的想法出發，flex 本身的特性會用子元素填滿父元素，因此只需要定義 `Header` 高度後，`Content` 就會自動撐滿空間
- 複製 `<Header />` 時，`Content` 仍會自動撐滿空間，滿足擴充性
- 不只是 `Content`，連父元素都不需要知道 `Header` 的高度，滿足低相依性
- 四種做法中最好的一個（ flex 會成為主流不是沒有道理的👍

<br />

# 狀況 2 - 多區塊滾動
除了 Fixed Header 以外，另一個要會的版型是「多區塊滾動」  
**上下:** 在前面的練習的 `Header` 中加入 `overflow: scroll` 就可以直接達成，上下區塊各自滾動的效果  
**左右:** 也就是常見的 SideMenu，從這個 **[範例](https://codesandbox.io/s/before-independent-scroll-xb6pu)** 開始試著實作吧！

**需求：**
- (Required) `Menu` 和 `Content` 元素左右排列
- (Required) `Menu` 和 `Content` 元素可以各自滾動

<br />

## 作法 1 - fixed width
- 直接訂死寬度 `${MenuWidth}px`, `calc(100% - ${MenuWidth}px)`
- 最直觀的做法，但擴展性不足，且相依度高

<br />

## 作法 2 - padding width
- `Menu` 元素 `${MenuWidth}px + position: fixed`、`Content` 元素 `paddding-left: ${MenuWidth}px`
- 和做法1 大同小異

<br />

## 作法 3 - ~~inline block~~
- 要把元素排入同一列中，一定會想到 `inline-block`，但筆者嘗試了許久，頂多做到「左右同步滾動」，沒辦法「分區塊滾動」
- 猜測無法達成「分區塊滾動」的效果，因為 `inline-block` 是定義在父元素，讓子元素可以排在同一列。實際上是對父元素滾動，不是對子元素觸發滾動，因此無法分區

<br />

## 作法 4 - grid
([CodeSandbox](https://codesandbox.io/s/after-independent-scroll-grid-fp9dz))  
- 一樣從規劃版型的觀念出發，先畫好 Menu 區域與 Content 區域
- 低相依，但擴展性低

<br />

## 作法 5 - flex
([CodeSandbox](https://codesandbox.io/s/after-independent-scroll-flex-p060t))  
- 兼具低相依、高擴展

<br />

# 結語
上面兩個練習都列出了多種做法，不是說一定要滿足「高擴展」、「低相依」才是唯一正解，每種做法的結果都有些微差異，要找出最符合自己開發情境及需求的做法。flex 看似強大，但如果遇到無法使用 flex 的狀況，這時其他做法就派上用場了。  
如果還有其他更好、更有創意的作法，歡迎告訴我們喔 🎉