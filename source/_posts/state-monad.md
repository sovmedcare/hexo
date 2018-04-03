---
title: state_monad
date: 2018-04-02 15:28:10
tags: [Haskell]
author: tan
---

許多的程式應用情境，是要依賴隨著時間或是運算而不停變換的狀態而進行。Haskell 中沒有變數，數值都是不可變的。

Haskell 要達到這種需求，同樣也是靠函數。藉由傳入 state 作為輸入參數，經過運算後回傳最後的結果以及更新後的 mewState。

原本的 state 並沒有被修改到，而我們也同樣達到了根據 state 做計算並且變換 state 的目的了

而在 Haskell 中，這種需要處理狀態性問題時，常用到的工具就是 State monad。

---

# 範例
一般會用躑骰子當範例

```haskell
import System.Random

rollDie5Times :: StdGen -> (Int, Int, Int, Int, Int)
rollDie5Times g =
  let
    (one, g1) = randomR (1, 6) g
    (two, g2) = randomR (1, 6) g1
    (three, g3) = randomR (1, 6) g2
    (four, g4) = randomR (1, 6) g3
    (five, _) = randomR (1, 6) g4
  in
    (one, two, three, four, five)
```

目前這種模擬躑五次骰子的方法看起來很不舒服，原因在於 generator 他必須手動取出和傳入，多了很多重複的步驟。

如果使用了 State monad，我們就可以將 generator 作為 state 來改善目前的範例。

---

# 定義 State monad

```haskell
newtype State s a = State {runState :: s -> (a, s)}
```
`State` 包裹著一個函數型別為`s -> (a, s)`。其意義就是如一開始所說的，傳入 state，經過運算回傳結果以及新的 state。

State monad 的`Functor` `Applicative` `Monad` instance 我們可以如此定義

```haskell
instance Functor (State s) where
  fmap f (State g) = State $ \s -> let (a, s1) = g s
                                   in  (f a, s1)

instance Applicative (State s) where
  pure a = State $ \s -> (a, s)
  (State fab) <*> (State fa) = State $ \s -> let (a, s1) = fa s
                                                 (ab, s2) = fab s1
                                             in  (ab a, s2)

instance Monad (State s) where
  return = pure
  State f >>= k = State $ \s -> let (a, s1) = f s
                                in runState (k a) s1
```

從實作中可以看到 state 會不斷的更新然後傳遞下去。


---

# Methods
一般的 State monad 還會提供幾個方便的 method

```haskell
-- 將函數包進 State 中，變為 State monad
state :: (s -> (a, s)) -> State s a
state = State

-- 取的當前的 state
get :: State s s
get = State $ \s -> (s, s)

-- 更新 state
put :: s -> State s ()
put x = State $ const ((), x)

-- 傳入 state 執行 State monad 並回傳最後的 state
execState :: State s a -> s -> s
execState (State sa) s = snd $ sa s

-- 傳入 state 執行 State monad 並回傳最後的運算結果 a
evalState :: State s a -> s -> a
evalState (State sa) s = fst $ sa s

-- 對 state 做轉換
modify :: (s -> s) -> State s ()
modify f = State $ \s -> ((), f s)
```

---

# 範例改寫
```haskell
rollDie :: State StdGen Int
rollDie = state $ randomR (1, 6)

rollDieNTimes :: Int -> State StdGen [Int]
rollDieNTimes n = replicateM n rollDie

> evalState (rollDieNTimes 5) (mkStdGen 0)
[5, 1, 4, 6, 6]
```

如此一來可以定義“躑 n 次骰子”的函數，而且看起來還比原先範例更為精簡。

---

# 結語

如同 Reader monad，一般在使用時，同樣會直接使用現成的 library 像 [`mtl`](https://hackage.haskell.org/package/mtl)或[`transformers`](https://hackage.haskell.org/package/transformers)。

可以去看看這些 library 是如何實作的，藉此來學習。

此外通常看到或是使用時，因為會需搭配其他的 Monad 一起使用，並不會用`State`而是使用`StateT`。而去看原始碼可以發現 `State s a` 其實也就是`StateT s Identity a`的type alias。

---

# 參考資料
1. [Haskell/Understanding monads/State - Wikibooks, open books for an open world](https://en.wikibooks.org/wiki/Haskell/Understanding_monads/State)
2. [Learn You a Haskell for Great Good!](http://learnyouahaskell.com/)