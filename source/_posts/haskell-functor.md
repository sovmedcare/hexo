---
title: Haskell 系列 - Functor (函子)
date: 2018-3-29 18:26:55
tags: [FP, Haskell, type, category theory, functor]
category: 
author: jackypan1989
---

# map 函數

如果之前是寫 javascript 的，那你一定會用過 map 這個 function
舉個例子

```javascript
const add1 = x => x + 1

add1(5) // 答案是6
add1([5]) // 有error 因為add1 無法應用在array上

[5].map(add1) // 答案是[6]

// 更甚者, 採用ramda.js之類的lib
const mappedAdd1 = R.map(add1)
mappedAdd1([5]) // 答案一樣是[6], 但add1已經轉成了mappedAdd1
```

可以發現這map函數可以讓我們把原本一個只能處理int的函數
變成一個可以處理[int]，也就是說其實他可以穿越[]這個所代表的意義
將這個context裡面的值取出，套用add1，再把他放回context

# Functor (函子)

如同剛剛所說 functor 其實就是把許多type中可以map這件事給抽象出來
看看 haskell 中的實作

```haskell
> :i Functor
class Functor (f :: * -> *) where
    fmap :: (a -> b) -> f a -> f b
    (<$) :: a -> f b -> f a
    {-# MINIMAL fmap #-}
    -- Defined in ‘GHC.Base’
```

這 fmap 就是我們常看到的 map 函數

```haskell
  (a -> b) -> f a -> f b
```

看看他的意思
傳進一個 function 跟一個帶有context的值再回傳另一個帶有context值
對照一開始的範例，傳進一個 x=x+1, [5], 再回傳另一個 [6]
所以有些人會把 functor 比喻成盒子, 不能說錯
但他其實就是很簡單的把 function 提升(lift)成 可以處理context的 function而已
是 function 的處理能力提升而不是真正有一個容器或是box在 (可能是因為list太適合用box解釋)

在數學上就是，兩個Hask範疇之間的轉換
而且也讓原本範疇(Hask範疇)中的物件(type)與態射(function)都提升成帶有上下文或處理上下文

# List Functor

這我們剛剛舉例過了，直接看看實作

```haskell
instance Functor [] where
    {-# INLINE fmap #-}
    fmap = map
```

fmap 果然就是 map 函數

# Maybe Functor

再來看看 Maybe
maybe a 包含了 just a 跟 nothing
其中 nothing 代表可能為空值這個 context

用想像力想一下
如果是 just 6 那我們就把 6 從 just 這個 context 取出來，做完運算再用just包回去
如果是 nothing 那就根本不用管直接 nothing 因為 nothing 怎麼操作都是 nothing

回去看看原始碼

```haskell
instance  Functor Maybe  where
    fmap _ Nothing       = Nothing
    fmap f (Just a)      = Just (f a)
```

果然我們想的一樣
以maybe來說，我們也可以直接針對他進行fmap操作
fmap我們又把他用 <$> 來代表

```haskell
>:{
add1 :: Int -> Int
add1 x = x + 1
:}

> add1 <$> [1, 2]
[2,3]

> add <$> Just 3
Just 4
```

這就對了，原本只能處理Int的function
變成可以處理[Int], 連Maybe Int也可以了
所以 [], Maybe 都是 functor

# 參考資料

1. haskell wiki
2. learn you a haskell
3. hoogle