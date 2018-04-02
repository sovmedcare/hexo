---
title: Haskell 系列 - Monoid (ㄠ半群)
date: 2018-3-28 18:26:55
tags: [FP, Haskell, type, category theory, group thoery]
category: 
author: jackypan1989
---

# Monoid (ㄠ半群)

Haskell 是跟純數學息息相關的程式語言，有許多抽象化的方式都來自數學理論(例如範疇論跟群論)
這裡的 typeclass Monoid 正是代表某些 data 的特性，首先看看在 haskell 中的 Monoid

```haskell
> :i Monoid
class Monoid a where
  mempty :: a
  mappend :: a -> a -> a
  mconcat :: [a] -> a
  {-# MINIMAL mempty, mappend #-}
```

我們可以發現 Monoid 必須至少實作兩個方法，一個為 mempty，另一個是 mappend
在數學理論中，定義如下，滿足以下兩個特徵的稱為 monoid

1. 有單位元
2. 有一個二元operator滿足結合律

1的意思是裡面有個元素，其他元素跟他結合都還是自己
2的意思是 a op (b op c) = (a op b) op c，表示括號不影響結果

用你的想像力，String是不是就是一個最明顯的例子，單位元是空字串("")，其他字串怎麼跟他前後結合都不會影響結果，那二元operator自然而然就是字串串接了(++)，下面用String舉例

```haskell
> "" ++ "ABC" ++ ""
"ABC"

> ("HELLO" ++ "ABC") ++ "!" == "HELLO" ++ ("ABC" ++ "!")
True
```

所以String還有haskell裡面一些代表或是處理字串的type基本上一定都是Monoid，當然更廣義的來看[]都是monoid，有空的array跟array的串接的話就是monoid

# list monoid ([])

看一下 [] 的實作

```haskell
class Semigroup a => Monoid a where
    -- | Identity of 'mappend'
    mempty  :: a

    -- | An associative operation
    --
    -- __NOTE__: This method is redundant and has the default
    -- implementation @'mappend' = '(<>)'@ since /base-4.11.0.0/.
    mappend :: a -> a -> a
    mappend = (<>)
    {-# INLINE mappend #-}

    -- | Fold a list using the monoid.
    --
    -- For most types, the default definition for 'mconcat' will be
    -- used, but the function is included in the class definition so
    -- that an optimized version can be provided for specific types.
    mconcat :: [a] -> a
    mconcat = foldr mappend mempty

instance Semigroup [a] where
    (<>) = (++)
    {-# INLINE (<>) #-}

    stimes = stimesList

instance Monoid [a] where
    {-# INLINE mempty #-}
    mempty  = []
    {-# INLINE mconcat #-}
    mconcat xss = [x | xs <- xss, x <- xs]
```

可以看到 Monoid 因為有 superclass Semigroup 定義了 mappend = (<>)
所以他裡面就不用重複實作，直接用 Semigroup 的 mappend
因此可以發現 mempty  = [] (單位元), mappend  = (++) = (<>)

用monoid方式的抽象來操作看看

```haskell
> import Data.Monoid
> "" <> "ABC"
"ABC"
```

# Sum monoid 跟 Product monoid

我第一次看到 monoid 定義的時候
我就想到了，數字也是這樣
例如

1. 加法的時候
  單位元是 0
  二元的operator 是 (+)

  任何數加上0都沒差,而且括號不影響 (1+2)+3 == 1+(2+3)

1. 乘法的時候
  單位元是 1
  二元的operator 是 (*)

  任何數乘上1都沒差,而且括號不影響 (1\*2)\*3 == 1\*(2\*3)

不過因為單位元不同，而且 instance 只能有一種實作
所以 Haskell 創造了 Sum 跟 Product 這兩個 newtype 來代表我剛剛說的

```haskell
-- Sum Monoid
newtype Sum a = Sum { getSum :: a }

instance Num a => Semigroup (Sum a) where
    (<>) = coerce ((+) :: a -> a -> a)
    stimes n (Sum a) = Sum (fromIntegral n * a)

instance Num a => Monoid (Sum a) where
    mempty = Sum 0

-- Product Monoid
newtype Product a = Product { getProduct :: a }

instance Num a => Semigroup (Product a) where
    (<>) = coerce ((*) :: a -> a -> a)
    stimes n (Product a) = Product (a ^ n)

instance Num a => Monoid (Product a) where
    mempty = Product 1
```

我一開始會覺得說為何會需要這些東西
但其實 Haskell 就是不斷利用數學一些抽象的技巧來層層疊疊
來幫助大家去優化最佳化，而不是每個data都有自己獨特實際實作很長的api

例如 String, [], Sum, Product 的這種monoid特性
不就隱含了一件常見的事情，這些事情是可以分開去做的，而且不會影響結果
所以我們也可以作一個分散式的function
例如 getData(xxx) <> getData(yyy) <> getData(zzz)
可以讓這三個function由不同cpu去跑，反正執行順序根本沒差
而這個 typeclass 正是把這種特性給抽象出來

# 參考資料

1. haskell wiki
2. learn you a haskell
3. [知乎：為何需要monoid](https://www.zhihu.com/question/25406710)