---
title: Reader monad
date: 2018-03-30 14:54:11
tags: [Haskell]
author: tancc
---

程式中時常有許多函數是需要去使用共用的常數(shared environment)，常見的像是設定檔(config)之類的東西。

此時函數的參數就是這些使用到的如 config，我們必須在 type signature 寫出需要的參數型別，此外在使用時必須顯式的寫出要傳入的東西
```haskell
-- 假設此時的 configs 中有 name age weight height 等等
myFunc1 :: Name -> Age -> ...
myFunc2 :: Name ->...
myFunc3 :: Weight -> Height -> ...
-- ...
myFunc1 name age
myFunc2 name 
myFunc3 weight height
-- ...
```

這樣的壞處很明顯就是我們必須重複寫很多東西。
要解決這問題的一個直覺想法就是，讓這些函數所在的 scope，可以自由地取得共用的資料，如此一來就不需由我們手動傳入了。
而在 functional programming 中，就是使用函數去達成。只要將這些函數做良好的定義，並塞到一個更大的函數裡，將共享資源傳入這個大函數中，裡頭的小函數們就都可以去取得了。

Reader monad 就是可以幫我們做這件事情的工具，因此他又有另外的名稱，Environment monad。
而最簡單的 Reader monad 其實就是一個函數的新名稱罷了。

---

# 範例
這裡用一段沒什麼用處的程式碼來舉例。
這裡的函數用返回執行別用`Maybe`，只是為了要用`do notation`而已
```haskell
  type Name = String
  type Age = Int
  type Height = Double
  type Weight = Double
  type Result = (String, String, String, Double)

  revName :: Name -> Maybe String
  revName n = Just $ "Reverse Name: " ++ reverse n

  nameAndAge :: Name -> Age -> Maybe String
  nameAndAge n a = Just $ "Name & Age: " ++ n ++ " " ++ show a

  weightAndHeight :: Weight -> Height -> Maybe String
  weightAndHeight w h = Just $ "Weight & Height: " ++ show w ++ " " ++ show h

  calBmi :: Height -> Weight -> Maybe Double
  calBmi h w = Just $ w / (h / 100)^2

  go = 
    let
      name   = "Tom"
      age    = 20
      height = 180
      weight = 100
    in
      do
        na <- nameAndAge name age
        n  <- revName name
        wh <- weightAndHeight weight height
        b  <- calBmi height weight
        return (na, n, wh, b)
```

可以發現 `nameAndAge`, `revName`, `weightAndHeight`, `calBmi`等函數，我們需要定義、傳地重複的東西進去，使程式碼看起來較為繁冗。
而他們用到的這些 input 就可以將其想像成 shared environment 這種在程式執行時並不會去改變且很多地方都會用到的東西。

---

# 定義 Reader monad
```haskell
newtype Reader r a = Reader { runReader :: r -> a }
```
也就是說，`Reader` 就是包裹了一個函數 `r -> a` ，其中 `r` 就是我們要傳入的共用資料，有些地方也會用`e`來表示(environment)；`a` 表示吃進 `r` 後，會回傳的東西是什麼。

再來我們可以自己定義他一系列的`Functor` `Applicative` `Monad` instance

```haskell
instance Functor (Reader r) where
  fmap :: (a -> b) -> Reader r a -> Reader r b
  fmap f (Reader ra) = Reader $ f . ra

instance Applicative (Reader r) where
  pure :: a -> Reader r a
  pure a = Reader $ const a
  (<*>) :: Reader r (a -> b) -> Reader r a -> Reader r b
  (Reader rab) <*> (Reader ra) =
    Reader $ \r -> rab r (ra r)

instance Monad (Reader r) where
  return = pure
  (>>=) :: Reader r a -> (a -> Reader r b) -> Reader r b
  (Reader ra) >>= aRb =
    Reader $ \r -> runReader (aRb (ra r)) r
```

可以看到這裡的 structure 是 `Reader r` 也就是 `(->) r` (function type) 這部分。而且在實作當中可以看到，`r`是保持著原來的值被傳遞，並沒有被做其他transform，因此可以確保 Reader monad 可以拿到相同的`r`

對照 `(->) r` 在[原始碼](https://hackage.haskell.org/package/base-4.11.0.0/docs/src/GHC.Base.html#line-794)中定義的`Functor` `Applicative` `Monad` instance，可以發現基本上是一樣的，只是多了 `Reader` 這一層包裝。

若將型別定義中的 `Reader r` 用 `(->) r` 替換，可以看到更明顯的結果
```haskell
  fmap :: (a -> b) -> Reader r a -> Reader r b
        = (a -> b) ->  (->)  r a ->  (->)  r b
        = (a -> b) ->  (r -> a)  ->  (r -> b)
        
  (<*>) :: Reader r (a -> b) -> Reader r a -> Reader r b
        =   (->)  r (a -> b) ->  (->)  r a ->  (->)  r b
        =      (r -> a -> b) ->  (r -> a)  ->  (r -> b)
        
  (>>=) :: Reader r a -> (a -> Reader r b) -> Reader r b
        =    (->) r a -> (a -> ((->) r b)) -> (->) r b
        =    (r -> a) -> (a -> (r -> b)    -> (r -> b)
```

所以其實 Reader monad 的概念，就是跟 function type `(->) r` 是一樣的。

--- 

# Methods
使用了`newtype Reader`將 function 包裝後，還沒有什麼用。
還需要一些小工具使我們可以方便地拿到想要的資料，`ask`與`asks`以及`local`
```haskell
-- 取得 shared environment
ask :: Reader a a
ask = Reader id
 -- = Reader $ \r -> r

asks :: (r -> a) -> Reader r a
asks = Reader
```

`ask`就是一個`id`，也就是傳什麼就吐一樣的東西回來。所以就是可以拿到`Reader r a`中的`r`。
```haskell
ex :: Reader Config String
ex = do 
  config <- ask
  -- ...
```

`asks`的參數是一個函數，這個函數的作用就類似 selector，用來塞選取得在 shared environment 中想要的資料。
```haskell
ex :: Reader Config String
ex = do 
  n <- asks $ lookup "name"
  -- ...
```

`local`是用來修改 Reader content，但他不是修改全域的內容，而是只有在`local` scope 裡面的 Reader Monad 才會被影響。
```haskell
local :: (r -> r) -> Reader r a -> Reader r a
local f (Reader ra) = Reader $ \r -> ra (f r)

-- example
ex :: Reader Int (Int, Int)
ex = do
  i <- local (+1) $ do
    i <- ask
    return i
  j <- ask
  return (i, j)
  
> runReader ex 10
(11, 10)
```
從`ex`的執行範例可以看到，`i`也就是經過`local`後 Reader content 的確是被修改了，但是用`ask`拿到的 Reader content `j`，還是為 10 沒有改變。
所以`local`的作用是區域修改而不會影響到其 scope 外的地方，所以總的來說其實 Reader content並沒有發生改變。

---

# 範例改寫

原本的範例程式可以用 Reader monad 如此改寫
```haskell
  type Result = (String, String, String, Double)

  data Config = Config { name   :: String
                        , age    :: Int
                        , height :: Double
                        , weight :: Double
                        }

  revName :: Reader Config String
  revName = do
    n <- asks name
    return $ "Reverse Name: " ++ reverse n

  nameAndAge :: Reader Config String
  nameAndAge = do
    n <- asks name
    a <- asks age
    return $ "Name & Age: " ++ n ++ " " ++ show a

  weightAndHeight :: Reader Config String
  weightAndHeight = do
    w <- asks weight
    h <- asks height
    return $ "Weight & Height: " ++ show w ++ " " ++ show h

  calBmi :: Reader Config Double
  calBmi = do
    w <- asks weight
    h <- asks height
    return $ w / (h / 100)^2
  
  go :: Reader Config Result
  go = do
      na <- nameAndAge
      n  <- revName
      wh <- weightAndHeight
      b  <- calBmi
      return (na, n, wh, b)
  
  showResult = runReader go $ Config {name = "WOW", age = 20, height = 190, weight = 100}
```

主要的差別在於，每個函數的 type signature 是一致的 `Reader Config [回傳型別]`，大家是共享一份 config。
各函數在其內部自行定義要取得什麼資料，如此一來在使用的時候就不需要顯示的傳遞進去了。

---

# 結語
一般情況在使用 Reader Monad 時並不會自行去定義，可能會用一些 library 像 [`mtl`](https://hackage.haskell.org/package/mtl)或[`transformers`](https://hackage.haskell.org/package/transformers)，
``` haskell
import Control.Monad.Reader

data MyContext = MyContext
  { foo :: String
  , bar :: Int
  } deriving (Show)

computation :: Reader MyContext (Maybe String)
computation = do
  n <- asks bar
  x <- asks foo
  if n > 0
    then return (Just x)
    else return Nothing

ex1 :: Maybe String
ex1 = runReader computation $ MyContext "hello" 1

ex2 :: Maybe String
ex2 = runReader computation $ MyContext "haskell" 0
```

他們同樣也都提供了基本的像是`ask`, `asks`, `local`，可以去看看這些 library 是如何實作的，藉此來學習。

此外通常看到或是使用時，因為會需搭配其他的 Monad 一起使用，並不會用`Reader`而是使用`ReaderT`，`T`表示 transformer。

---

# 參考資料及更多資源

1. [mtl](https://hackage.haskell.org/package/mtl)
2. [transformers](https://hackage.haskell.org/package/transformers)
3. [Three Useful Monads - adit.io](http://adit.io/posts/2013-06-10-three-useful-monads.html)
4. [What I Wish I Knew When Learning Haskell 2.3 ( Stephen Diehl )](http://dev.stephendiehl.com/hask/#reader-monad)