---
title: Applicative Functor
date: 2018-03-21 15:03:23
tags: [haskell]
author: tancc
---

# 1 Applicative TypeClass

`Applicative Functor`比起`Functor`更為強大，以下是其部分typeclass定義
```haskell
class Functor f => Applicative f where
    {-# MINIMAL pure, ((<*>) | liftA2) #-}
    -- | Lift a value.
    pure :: a -> f a
    -- | Sequential application.
    (<*>) :: f (a -> b) -> f a -> f b
```

一個類型如果為`Applicative`的實例(instance)的話，則他同樣會具有funtoral structure，也就是`f a` 中的 `f`。

而和`Functor`最主要多的不同在於，多了兩個方法`pure`和`<*>`

`pure`：將型別為`a`的input，包進一個Applicative的結構`f`中，轉換成型別為`f a`的output

`<*>`： 與`fmap`相似，都是把函數提升(lift)，使其能應用在型別具有額外結構的值中。不同的是`<*>`所提升的function，本身就被包裹在`f`裡`f (a -> b)`

同時比較`fmap`跟`<*>`會比較有感：
```haskell
fmap  :: Functor     f =>   (a -> b) -> f a -> f b
(<*>) :: Applicative f => f (a -> b) -> f a -> f b
```
差別只在要lift的函數有沒有`f`被包裹起來。

---

# 2 Applicative functor laws

但一個型別就算其實做了`pure`以及`<*>`成為了`Applicative`的instance了，也還不能夠說是一個`Applicative`

除了實作`pure`以及`<*>`外，還需要滿足4個`Applicative functor laws`
```haskell
-- 1. Identity
pure id <*> v = v 

2. Composition
pure (.) <*> u <*> v <*> w = u <*> (v <*> w)

3. Homomorphisom
pure f <*> pure x = pure (f x)

4. Interchange
u <*> pure y = pure ($ y) <*> u
```

因為在Haskell中，你定義完`instance Applicative XXX where...`後，並不會幫你檢查是否滿足這些`Applicative functor laws`，因此這部分需要自己去確保。

---

# 3 一些使用情境

- 將普通函數`(a -> b -> c ->...)`，應用在多個有functoral structure/context的值`f a, fb, ...`時

```haskell
fmap  :: Functor     f =>   (a -> b) -> f a -> f b
(<*>) :: Applicative f => f (a -> b) -> f a -> f b
```
從type signature中可以看到，`fmap`無法將`f (a -> b)` 應用於 `f a,f b`
```haskell
x = fmap (+) (Just 1)
--x = Just (+ 1) 型別為 f (a -> b)

fmap x (Just 2)
-- error
-- 我們無法用fmap對x進行操作了
-- fmap的第一個參數型別要為 (a -> b)，不可為 f (a -> b)
```
這時就可以使用`<*>`搭配`<$>`來達成需求了
```haskell
(+) <$> (Just 1) <*> (Just 2)
>> Just 3
  
  (+) <$> (Just 1) <*> (Just 2)
= Just (+ 1) <*> (Just 2)
= Just 3

-- 另個例子
(\x y z -> [x*y, y*z, x*z]) <$> (Just 1) <*> (Just 2) <*> (Just 3)
>> Just [2,6,3]

  (\x y z -> [x*y, y*z, x*z]) <$> (Just 1) <*> (Just 2) <*> (Just 3)
= Just (\y z -> [1*y, y*z, 1*z]) <*> (Just 2) <*> (Just 3)
= Just (\z -> [1*2, 2*z, 1*z]) <*> (Just 3)
= Just ([1*2, 2*3, 1*3])
= Just [2,6,3]

-- 另個例子
data User = User { firstName :: String
                 , lastName  :: String
                 , email     :: String
                 } deriving (Show)
  
validate :: String -> Maybe String
validate [] = Nothing
validate s = Just s

makeUser :: String -> String -> String -> Maybe User
makeUser f l e = User
         <$> validate f
         <*> validate l
         <*> validate e
-- 可以想像makeUser如果不用<*>，勢必就必須寫很多case...of來達成

```

- 多個運算間沒有相依關係時
```haskell
(<*>) :: Applicative f => f (a -> b) -> f a -> f b
(>>=) ::       Monad m => m a -> (a -> m b) -> m b
```
從type signature中可以看到，`>>=`所串聯的運算式有相依關係的，下一個運算的依賴上一個運算的結果。

也就是說Monad `>>=`的運算其實是相依(Dependency)，但這也是他彈性的部分，因為我們可以針對運算的返回值再進一步的操作。

而Applicative `<*>`的運算是不相依(Independency)。因為少了相依性，對比起來使用Applicative可以寫出更乾淨的代碼。


---

# 4 
前面有個例子`(+) <$> (Just 1) <*> (Just 2)`，不過`Control.Applicative`有提供我們一些方便的工具來做同樣的事情
```haskell
liftA  :: (a -> b)           -> f a -> f b
liftA2 :: (a -> b -> c)      -> f a -> f b -> f c
liftA3 :: (a -> b -> c -> d) -> f a -> f b -> f c -> f d
```
因此可改寫成
```haskell
liftA2 (+) (Just 1) (Just 2)
```
視覺上看起來有比較簡潔了
而`liftA` `liftA2` `lift3`的差別只在其所要lift的函數的參數個數而已


此外，使用`<$>..<*>..<*>`，在閱讀上或許對一部分人來說比較不直覺的。如`f <$> expr1 <*> expr2 <*> expr3`，執行順序並不是由左往右的。

這部分可以使用ApplicativeDo，使得可以跟以往使用do notationu那般由上到下的書寫方式
```haskell
{-# LANGUAGE ApplicativeDo #-}

run = do
 x <- expr1
 y <- expr1
 z <- expr1
 return (f x y z)
```
因為expre並不會相互依賴，因此會被轉換成`f <$> expr1 <*> expr2 <*> expr3`，其實就還是一樣的東西。

---

# 5 參考資料及更詳盡的內容

[GHC.Base#Applicative](https://hackage.haskell.org/package/base-4.11.0.0/docs/src/GHC.Base.html#Applicative)
[Haskell/Applicative functors - Wikibooks, open books for an open world](https://en.wikibooks.org/wiki/Haskell/Applicative_functors)
[ApplicativeDo – GHC](https://ghc.haskell.org/trac/ghc/wiki/ApplicativeDo)
