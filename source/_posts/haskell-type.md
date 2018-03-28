---
title: Haskell 系列 - type (類型)
date: 2018-3-23 18:26:55
tags: [FP, Haskell, type]
category: 
author: jackypan1989
---

# Type (類型)

在 haskell 中我們是使用 data 關鍵字來定義所謂的類型
先看一個例子, Bool 是一個常見的類型, 包含兩個值(data), True 跟 False 

``` haskell
data Bool = True | False
```

在這裡 
= 號的左邊我們稱為 Type Constructor
= 號的右邊我們稱之為 Data Contructor (或者是 Value Contructor)

所以 Bool 是 Type Constructor, 用 kind 可查看這 type 的特徵
:: 的後面代表他的類型
``` haskell
> :k Bool
Bool :: *
```
出現 \* 表示他本身就是一個 Concrete type
所以 Bool 既是 type 同時也是 type constructor (但無參數)

另一方面
True 跟 False 是 Data Contructor, 用 :t 可查看他是屬於什麼 type
``` haskell
> :t True
True :: Bool
```
True 是 Bool type 的data/value, 也不需要參數

小結一下, 如果要用命令列來看 
:k 是用來查看等號左邊的 type contructor (例如 Bool)
:t 是用來查看等號右邊的 data contructor (例如 True)

# 看看比較複雜的 Maybe a 類型
``` haskell
data Maybe a = Just a | Nothing
```
這裡的 Maybe a 就如同上面的 Bool 一樣, 由兩個 data contructor 組成
但這裡的 a 我們又稱為 type variable
用來跟 Maybe 一起組合並回傳一個真正的 type 即為 Maybe a
a 用表示他有可能是 Int, String, 甚至可能是另一個 Maybe b (酷吧, 自己的定義自己用)
所以, 以類型 Maybe Int 來說
他的 data 中可能有 Just 1, Just 2, Just 4 ... & Nothing 他們的 type 都是 Maybe Int
```haskell
> :t Just "a"
Just "a" :: Maybe String

> :t Nothing
Nothing :: Maybe a

> :t Nothing :: Maybe Int
Nothing :: Maybe Int :: Maybe Int
```
在看 Nothing 的 type 時必須給他一點提示
協助他去推斷 Nothing, 否他會只給你一個 Maybe a
因為 Nothing 都是所有 Maybe a 會有的值

再回頭看看這個 type 的 kind
```haskell
> :k Maybe Int
Maybe Int :: *

> :k Maybe
Maybe :: * -> *
```

剛剛有說過一個 \* 才表示已經是 concrete type 了
這裡會出現一個 \* -> \* 表示他只是 type contructor
你必須先傳進一個 concrete type 才會回傳一個 concrete type
這就是為什麼只打一個 Maybe 還不夠, 一定要接一個 Int (concrete type)

# 注意事項

## 1. data constructor 產生出來的是值/data, 可以傳入 function, 但 type 不能傳入 function (Data constructors as first class values)

例如 f 是某個 function
則 f (True) 或是 f (Just 2) 都合法
但 f (Maybe) 就不合法了

## 2. data constructor 不是 type (Data constructors are not types)

例如我們定義一個
```haskell
data MyMaybe a = Just a | Nothing
合法

data MyMaybe a = Just (Just a) | Nothing
不合法

data MyMaybe a = Just (Maybe a) | Nothing
合法 
(Data contrutor 裡面如果要塞參數一定是要塞 type 只有 Maybe a 是 type, Just a 是值)
```

# 更多例子

```haskell
data Color = Red | Blue | Green | RGB Int Int Int
```

Color 是一個 type 而且也是一個不用傳值的 type contructor
```haskell
> :k Color
Color :: *
```

Red 跟 RGB 都是 data contructor
其中 Red 不用傳參數
RGB 要傳三個 Int 才會回傳一個 value, 其 type 是 Color
```haskell
> :t Red
Red :: Color
> :t RGB
RGB :: Int -> Int -> Int -> Color
```

# 參考資料

1. haskell wiki
2. learn you a haskell