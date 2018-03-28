---
title: Monad
date: 2018-03-27 16:29:04
tags: [Haskell]
author: tancc
---

`Monad`對工程師而言，如果沒牽扯到數學相關的範疇其實不是那麼可怕難懂的東西。

```haskell
fmap :: (a -> b) -> f a -> f b
```

`Functor`實作了`fmap`，使得可將普通函數`(a -> b)`lift到`f a -> f b`；

```haskell
(<*>) :: f (a -> b) -> f a -> f b
```

`Applicative`實作了`<*>`，讓我們可以將保裹在某 structure 中的函數`f (a -> b)`，apply 到同樣含有相同結構的數值中`f a -> f b`

他們做的事情其實都是 function application，只是應用的情境不同而已。而從工程面來看，Monad 其實也是一樣的，也為我們做了某情境下的函數調用的抽象。

# 1 Monad TypeClass

Monad typeClass 的部分定義如下
```haskell
class Applicative m => Monad m where
    {-# MINIMAL (>>=) #-}
    return :: a -> m a
    return = pure
    (>>=)  :: m a -> (a -> m b) -> m b
    join   :: m m a -> m a
```
其中可以看到 Monad 必須是 Applicative，所以`return`的預設定義就是`pure`，將一個數值包進 monadic structure `m`。

`>>=`(bind)，是使用monad進行操作時十分常看到的東西。
從他的 type signature 不難發現，他要做的事也是 function application，只是這次要應用的函數是`(a -> m b)`，而這函數會所 return 的值是將 input 包上一個額外的結構`m`，而最終`>>=`所返回值`m b`會跟輸入`m a`是具有相同結構的。

# 2 Monad laws
如同`Applicative Functor`，就算一個型別實作了`>>=`成為了 Monad 的instance了，也還不能夠說是一個`Monad`。

必須還要在滿足三個 Monad laws
```haskell
-- 1. right identity
m >>= return    = m
-- 2. left identity
return x >>= f  = f x
-- 3. associativity
(m >>= f) >>= g = m >>= (\x -> f x >>= g)
```
這部分可以自行拿一些已知的Monad去驗證(List, Maybe, Either Monad)，而這在自行定義 Monad 時也是要自己去確保滿足的，Haskell compiler 並不會跟你說符不符合。

---

# 3 Maybe Monad

當在做除法運算的時候，會不希望除數為 0，否則後續運算可能會出現非預期的結果，因此定義了一個`safeDivide`
```haskell
safeDivide :: Double -> Double -> Maybe Double
safeDivide _ 0 = Nothing
safeDivide a b = Just (a / b)
```
`safeDivide`會回傳`Maybe Double`，這樣子的話做其他的運算或者說在做函數組合時，其他函數 input 型別都也要改成`Maybe`型別，才可以拿其結果作為輸入來使用。
所以在其他函數內部，都需根據 Maybe 的兩種可能性`Just a`與`Nothing`做不同的運算。

```haskell
maybeFunc1 ::Num a => Maybe a -> Maybe a
maybeFunc1 n =
  case of n of
    Nothing -> Nothing
    Just a -> ...
```
可以想像，每個 maybeFunc 都要去做這些判斷是會非常麻煩，要寫很多重複的程式碼。

Maybe Monad 就可以幫我們將判斷這些`case ... of`的工作抽象出來

```haskell
instance Monad Maybe where
  return = pure
  -- >>= ::  Maybe a -> (a -> Maybe b) -> Maybe b
  (Just x) >>= k = k x
  Nothing  >>= _ = Nothing
```

其`>>=`的定義，幫我們做掉判斷`Nothing`跟`Just`然後做不同任務的工作。
當是 Nothing 時，就不理會後續的函數`k`，直接返回 Nothing。
若是 Just 時，則將其中的值`a`，傳進後續要執行的函數`k`中。

也可以看到，以`Maybe`而言，使用`>>=`我們就可以不斷串接有這種`a -> Maybe b`型別的函式，因為`(Just x) >>= k`回傳的也還是`Maybe`型別的數值，可以繼續用`>>=`串接下一步的運算。

事實上就是，如果我們有一系列的函數`a -> m b`，我們可以藉由`>>=`將這些相依的運算做序列組合。

所以原本的函數組合就很容易可以這樣寫
```haskell
safeDivide >>= maybeFunc1 >>= maybeFunc2...

--  (safeDivide 10 0) >>=...
-- = Nothing >>= ...
-- = Nothing
```

同樣類似的行為，如`Either Monad` 也能幫助我們省下判斷`Left`和`Right`的[程式碼](https://hackage.haskell.org/package/base-4.10.1.0/docs/src/Data.Either.html#line-141)

---

# 4 List Monad

首先看一下如何定義 List Monad 內容
```haskell
instance Monad [] where
  return = pure
  -- >>= :: [a] -> (a -> [b]) -> [b]
  m  >>= f  =  concat (map f m)
```
`return`和其在`Applicative`中[定義](https://hackage.haskell.org/package/base-4.10.1.0/docs/src/GHC.Base.html#line-807)的`pure`一樣，就是把值丟進一個 List 中。
`>>=` 的行為是將`f` map 到 List 中，然後concat最終結果。

```haskell
a = [1,3,5,7]

f :: a -> [a]
f = \x -> [x, x]

a >>= f
= [1,3,5,7] >>= \x -> [x, x]
= concat (map (\x -> [x, x]) [1,3,5,7])
= concat [[1,1],[3,3],[5,5],[7,7]]
= [1,1,3,3,5,5,7,7]
```

其實上面 List Monad 的bind`>>=`所做的事情，可以用List comprehension達到。
而實際上，[原始碼](https://hackage.haskell.org/package/base-4.10.1.0/docs/src/GHC.Base.html#line-820)中，List Monad 的`>>=`就是用 List comprehensions 實作的。

```haskell
instance Monad []  where
  xs >>= f = [y | x <- xs, y <- f x]

a >>= f
= [y | x <- [1,3,5,7], y <- (\x -> [x, x]) x]
= [1,1,3,3,5,5,7,7]
```
所以從List Monad來看，因為兩者行為的等價，也比較能明白為何有人會說 Monad 做的事情，其實就是 flatmap 或 concatmap。

---

# 5 Do notation

如果只用`>>=`的話有可能會發生程式碼巢狀結構太深的問題
```haskell
f >>= \a ->
   (g a) >>= \b ->
     (h b) >>= \c ->
       return (a, b, c)
```
這時候可以使用do notation這個語法糖，來幫助用imperative programming的形式由上至下撰寫代碼，增加可讀性
```haskell
do
  a <- f
  b <- g a
  c <- h b
  return (a, b, c)
```

---

# 6 Type signature
這段希望能從type signature來得到一些操作上的直覺

### Function Application

從 TypeClass 知道，若一個東西是 Monad，那他必然是 Applicative 亦即也必然是 Functor。

所以Functor、Applicative、Monad在應用的行為上肯定有一定程度的一致性。
```haskell
<$>        ::   (a -> b) -> f a -> f b
<*>        :: f (a -> b) -> f a -> f b
-- 將 >>= 做flip
-- (>>=)  :: m a -> (a -> m b) -> m b
flip . >>= :: (a -> m b) -> m a -> m b
```

三者從 type signature 來比較，更有一開始所說的，都是將 function application 做不同應用情境的抽象。

### Dependent computation

同樣從`>>=`的型別去看
```haskell
(>>=)  :: m a -> (a -> m b) -> m b
```
`>>=`接受了`m a`，而第二個參數是個函數`(a -> m b)`，其中的`a`從哪來，就是從`m a`的運算結果而來。

因此`>>=`所做的事情，就是將一個 monadic structure 所包裹的 computation `m a`，傳遞到串接的函數中 `(a -> m b)`，讓此函數根據傳遞進來的參數去做某些運算，最後再返回`m b`。而此`m b`又可搭配其他 monadic function `(a -> m b)`，繼續根據函數返回值操作下去。

而這種相依關係也是 Monad 較靈活的原因之一，因為它可以根據函數返回的值再去後續的運算。而這是 Applicative 無法做到的，`<*>`所串接的函數彼此是不相依的。

### General concatenation

如果都是去 lift 普通函數並且 apply 到有額外結構的值中，如`f a`, `m a`，那麼用 fmap 就行了。
但如果這個函數`a -> m b`，是返回一個有 monadic structure 的值時，使用fmap會發生什麼事情呢？
```haskell
-- 將 f 用 m 來表示
<$> :: (a -> b) -> m a -> m b
-- 將 b 代換成 m b
<$> :: (a -> m b) -> m a -> m (m b)

-- ex
fmap (\x -> [x]) [1,2,3]
= [[1], [2], [3]]
```
可以看到最終返回的結果是`m (m b)`，但我們並不想要有改變原有的架構，變成巢狀的結構，如巢狀的 List。所以要想辦法將 `m (m b)` 轉換成 `m b`，也就是去 flattern 或是 concate 這個雙層的結構。

而這就是 Monad 的一個特色所在，`join`。
```haskell
join :: Monadm m => m (m a) -> m a
```
可以看到，`join`所做的事情就是去concate這兩層`m m`為一個`m`。因此也可以看出 Monad 其實提供了更 general 的 concat 方法。

所以我們用`join`和`fmap`其實就可以構造出`>>=`

```haskell
(>>=) : m a -> (a -> m b) -> m b
m >>= f = join $ fmap f m
```
因此可以看出，其實 Monad 就是提供一個方法，讓我們去 map 函數，最後再將其結果join起來。

---

# 7 結語

`Monoid`, `Funtor`, `Applicative`, `Monad` ... 這些 typeclass，都代表著某些行為的抽象。

所以這些東西真的有存在的必要嗎？個人認為就工程的角度而言都沒有。
今天就算把名稱換成`Apple`, `Orange`, `Banana`, `Pineapple`也是可以(並且比較不會讓人畏懼？)，叫什麼名字都不是重點。並不是因為他是`Monad`我們才可以做到什麼什麼，一樣可以用其他方式來達成一樣的目的。
我們需要是程式行為上的抽象或是本質上的改變，使得程式碼的重用性和表達能力能夠提升，而這也正是 functional programming language 的強項。

而因為這些被抽象的行為產生出來的輪廓 (type signature)，可以跟範疇論 (category theory) 去結合，所以拿範疇論的專有名詞作為 typeclass 的名稱，較能將數學與程式結合的意圖反應出來。

除此之外我們可以看到，不同型別(Maybe, List, ...)，在不同 typeclass instance 的定義中，儘管都有做`fmap`,`<*>`, `>>=`等等的函數存在，但他們所做的事情、邏輯是獨立的，但最終他們所具有的結構是一致的。

我們其實也可以將其命名為`listFmap`, `maybeFmap` ... 等等，但因為統一了 interface 大家的名稱保持一致，並且具有相同的結構，這帶來的好處就是能再更近一步的去重用這些代碼，建構更高層次的抽象方法。

# 參考資料
[What I Wish I Knew When Learning Haskell 2.3 ( Stephen Diehl )](http://dev.stephendiehl.com/hask/)

[Monad - HaskellWiki](https://wiki.haskell.org/Monad)

[Learn You a Haskell for Great Good!](http://learnyouahaskell.com/)
