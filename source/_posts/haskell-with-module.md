---
title: 在Haskell使用Module
date: 2018-03-27 10:42:05
tags: [FP, Haskell]
author: Tu-Szu-Chi
---
# Module

大部分的專案我們都會需要引入其他第三方或是自己寫的`Module`、`Package`，目的就是爲了讓許多函數可以重複操作
而最基本的一些操作都是在`Prelude`這模組裡，預設會自動載入，下`stack ghci`指令就會發現會是`Prelude> ...`的prompt

## Data.List

`Data.List`讓我們可以更方便的對**List**做處理，**nub**這Function可以幫我們從List取出不重複的數字

```haskell
-- Main.hs
module Main (main) where

import Data.List

main :: IO ()
main = print "Main"

numUniques :: (Eq a) => [a] -> Int
numUniques = length . nub
```

在ghci中`:l Main`後，呼叫`numUniques [1,2,2,3]`就會是**3**了
但在此個人建議指針對要用到的Function做引入，例如這邊只用到**nub**那我們就改寫一下import部分

```haskell
import Data.List(nub)
```

## System.Random

都說Hakell是一個純函數語言，餵給一個函數相同的參數，不管怎樣都會回傳相同的結果，那要怎實作「隨機」的狀況呢？
就是利用這個`System.Random` Module，而要實做隨機的函數就叫做`random`

```haskell
random :: (RandomGen g, Random a) => g -> (a, g)
```

來分別講解兩個參數，**RandomGen** typeclass是指作爲**亂源**的type(Gen是Generator縮寫)
**Random** typeclass是指可以裝**亂數**的type(像是Int,Bool...)
首先我們要一個RandomGen的實例，在`System.Random`裡的**StdGen**就是了，剛好有個`mkStdGen`函數可以幫我們產生這實例，只要傳給他Int即可

```haskell
mkStdGen :: Int -> StdGen
```

跑跑看

```haskell
random (mkStdGen 100)
> (-3633736515773289454,693699796 2103410263)
```

剛有提到`random`會回傳的是一個Tuple`(Random, RandomGen)`，逗號右邊的確實是**RandomGen**(空格沒問題)，我們的值不一樣也很正常
在沒有指定回傳的type時，預設是Int，所以我們也可以指定成*Bool*、*Char*

```haskell
random (mkStdGen 100) :: (Bool, StdGen)
> (True,4041414 40692)
```

在這也可以發現兩次都有回傳不一樣的StdGen，如果要連續做好幾次random就可以接着用下去，所以剛好有個`randoms`函數，接收一個StdGen然後回傳無窮的List

```haskell
-- randoms :: (RandomGen g, Random a) => g -> [a]
take 5 $ randoms (mkStdGen 11) :: [Bool]
> [True,True,True,True,False]
```

## 自己的Module

該來自己寫個Moduel試試，在Main.hs當前目錄新增一個**Lib.hs**，並如下這樣寫，記得在Main裡`import Lib`

```haskell
module Lib
    ( someFunc
    ) where

someFunc :: IO ()
someFunc = putStrLn "someFunc"

someFunc' :: IO ()
somwFunc' = putStrLn "no no no..."
```

`:r`重新載入後，分別呼叫`someFunc`、`someFunc'`試試，會發現`someFunc'`會報錯，因爲我們只有export出`someFunc`而已
通常專案還會有許多子資料夾，我們新建一個`Module`的資料夾，將我們的`Lib.hs`移進去，這時候重新載入可以，但呼叫`someFunc`就報錯，因爲找不到了，我們要改寫一下module的名字就可以了

```haskell
-- /Module/Lib.hs
module Module.Lib
    ( someFunc
    ) where
-- /Main.hs
import Module.Lib
```

那會碰上不同Module可是Function name重複的問題，Haskell當然也有alias的方法，呼叫時只要用`Lib.someFunc`的形式就可以了

```haskell
import qualified Module.Lib as Lib
import qualified Module.XXX as XXX
```

## 參考資料

[Learn your haskell](https://learnyoua.haskell.sg/content/zh-tw/ch07/module.html)