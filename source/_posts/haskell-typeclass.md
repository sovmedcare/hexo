---
title: Haskell 系列 - typeclass (類型類)
date: 2018-3-23 18:26:55
tags: [FP, Haskell, type, typeclass]
category: 
author: jackypan1989
---

# Type class (類型類)

首先看一下 (+) 這個 operator 的 type

```haskell
> :t (+)
(+) :: Num a => a -> a -> a
```

可以發現除了原本的 a -> a -> a 之外
還多了一個大箭頭, 這裡的 Num a 代表後面類型 a 一定是 Num 這個 typeclass
我們又稱為類型約束, 如果用 oo 講法來看, 他就像是 interface
你必須實作一些 Num typeclass 的 function 才能屬於 Num

在看一下 Num 的詳細資訊

```haskell
> :i Num
class Num a where
  (+) :: a -> a -> a
  (-) :: a -> a -> a
  (*) :: a -> a -> a
  negate :: a -> a
  abs :: a -> a
  signum :: a -> a
  fromInteger :: Integer -> a
  {-# MINIMAL (+), (*), abs, signum, fromInteger, (negate | (-)) #-}
  -- Defined in ‘GHC.Num’
instance Num Word -- Defined in ‘GHC.Num’
instance Num Integer -- Defined in ‘GHC.Num’
instance Num Int -- Defined in ‘GHC.Num’
instance Num Float -- Defined in ‘GHC.Float’
instance Num Double -- Defined in ‘GHC.Float’
```

這裡表示 Num 這個 typeclass 底下必須
實作 (+), (*), abs, signum, fromInteger, (negate | (-)) ( | 代表至少一個要作)
目前有 Word, Integer, Int, Float, Double 這幾個 type 是 Num 的 instance

# 其他常用的內建 typeclass

## Show

將 值或是data 轉為String
例如

```haskell
> :t show
show :: Show a => a -> String

> show 1
"1"
```

## Read

跟 show 相反, 將String轉為對應的值或data
例如

```haskell
> :t read
read :: Read a => String -> a

> read "3" :: Int -- 這裡給compile提示
3
```

## Ord

實做了 compare 或是比較運算元
例如

```haskell
> :t (>)
(>) :: Ord a => a -> a -> Bool

> (>) 3 4
False

> compare 3 4
LT
```

其中 compare 會回傳 Ordering 這個 type

```haskell
> :i Ordering
data Ordering = LT | EQ | GT 
```

分別代表小於, 等於, 跟大於

# 實作 instance & 自動派生 (derive)

實作 instance 部分，舉個例子

```haskell
> data Grade = A | B | C | Fail
> show A
<interactive>:32:1: error:

> :{
instance Show Grade where
    show A = "A"
    show B = "B"
    show C = "C"
    show Fail = "no grade!"
:}

> show A
"A"

> show Fail
"no grade!"
```

對於一些簡單的類型，haskell compiler 可以將他自動派生給一些 typeclass
像是 Read, Show, Bounded, Enum, Eq, Ord 都可以
他就會自己去產生一些行為

```haskell
> :{
data Grade = A | B | C | Fail
    deriving (Read, Show, Eq, Ord)
:}

> show A
"A"

> show Fail
"Fail"

> A == C
False

> compare A B
LT
```

# newtype

之前提到 data 關鍵字 可用來定義一個 type
但現在如果只有一個 data constructor (沒有 xx | oo) 又只有一個參數時
就可以用 newtype 取代

```
data MyType = MyType Int
等於
newtype MyType = MyType Int
```

差別在於編譯期 MyType 都是一個新的 type 編譯器會幫你檢查
但實際上執行時由於只有一個 data constructor 且 唯一參數
他就直接把 該參數型態 Int 當作 MyType 型態了, 也當然省去了 data constructor 的解構

# 參考資料
1. haskell wiki
2. learn you a haskell