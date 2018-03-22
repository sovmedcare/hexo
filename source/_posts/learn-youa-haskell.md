---
title: 初識Haskell
date: 2018-03-19
tags: [Haskell,FP]
author: Tu-Szu-Chi
---
# Haskell

## 簡介

在之前我們介紹了些**Functional Programming**的概念*(ex. Currying/High-Order-Function...)*
**Haskell**就是最具有代表性的函數式語言，整個應用就是透過定義函數而組成，大致上函數皆沒有副作用*(Haskell用特定類型表達副作用，後續會提及)*
不再像命令式語言命令電腦要做什麼，而是用函數來描述出問題是什麼
其本身也有社群 - [Hackage](https://goo.gl/UiG8eQ) 持續發佈許多工具讓我們更方便建立應用
而多數理工人都碰過的**GHC**編譯器其實就是**Glasgow Haskell Compiler**的縮寫
接着我會從安裝到語法來介紹這門特殊的程式語言
![logo](https://upload.wikimedia.org/wikipedia/commons/thumb/1/1c/Haskell-Logo.svg/240px-Haskell-Logo.svg.png "Haskell以「λ」作為自己的標誌，具有「證明即程式、命題為類型」的特徵")

## 安裝

在[Haskell官方](https://www.haskell.org/downloads)頁面中有三種方式讓我們建設基本環境，在此我選擇功能較齊全的`Haskell-Platform`來安裝，並用Docker新建了純淨的Ubuntu環境來操作
有碰上*ERROR: Unable to locate package*，先下一個`apt-get update`再`apt-get install haskell-platform`就可以了
安裝後如果發現*ERROR: stack command not found*，可參考[stack 官方](https://github.com/commercialhaskell/stack/blob/master/doc/GUIDE.md#downloading-and-installation)的步驟

### ghci

`ghci`這指令可讓我們直接進入終端機來嘗試Haskell，就像Python的`python`指令一樣

```bash
root@758b279035c4:~# ghci
GHCi, version 7.10.3: http://www.haskell.org/ghc/  :? for help
Prelude> :q
Leaving GHCi.
root@758b279035c4:~#
```

### stack

`stack`指令主要是用來管理Haskell Project，`stack new`可以快速地幫我們建立一個基礎的專案架構
在此用的版本是`1.6.5`，截至今日，建議升級到1.6以上

```bash
# stack --version
# stack upgrade
stack new my-project
cd my-project
stack setup
stack build
stack exec my-project-exe
```

### cabal

`cabal`主要用來管理package，`stack`背後也就是cabal在運作

### 解釋

現在*/my-project*內會有`stack.yaml` & `my-project.cabal`兩個檔案，在此要稍微解釋一下
這和我們在寫Javascript專案時不一樣，不單單只有一個`package.json`幫我們管理第三方套件
首先，打開*my-project.cabal*，會看到cabal定義的package主要會有:

- Name & Version
- 0 or 1 libraries
- 0 or more executables
- A cabal file (在此是*my-project.cabal*)

而打開*stack.yaml*，stack定義的project會有:

- A resolver
- Extra dependencies (extra-deps)
- 0 or more local Cabal packages

`stack.yaml`中的**resolver**除了定義要用什麼來編譯外，也決定了*.cabal*中**build-depends**的package版本
各位可以試試看，當`resolver = lts-10.5` or `resolver = lts-9.4`兩種狀況時，當`stack build`過程，**network**的版本分別是多少

```yaml
executable my-project-exe
  hs-source-dirs:      app
  main-is:             Main.hs
  ghc-options:         -threaded -rtsopts -with-rtsopts=-N
  build-depends:       base >= 4.7 && < 5
                     , network
                     , my-project
  default-language:    Haskell2010
```

對於各個resolver對應的package version都可以在[這網站](https://www.stackage.org/lts-10.5)查到
那當你想用[Hackage](https://hackage.haskell.org/packages/browse)上的某個package，卻沒在當前的resolver找到呢？
這時只要在**stack.yaml**的`extra-deps`添加即可(版本號必備)

```yaml
extra-deps:
- accelerate-cublas-0.0
```

而專案的架構也可以從Github上發現，在package的`hs-source-dirs`資料夾，除了主要的Main檔外，不會有多餘的檔案
自行定義的諸多Function都整理爲**Library**，並在**.cabal**的`library: Exposed-Modules`載入
可以參考 [begriffs/postgrest](https://github.com/begriffs/postgrest) & [uber/queryparser](https://github.com/uber/queryparser)這兩個repo

## 語法

首先，有兩種方式可以讓我們進入**GHC**的互動模式

```bash
# 1
stack ghci
# 2
cd app/
ghci
:l Main
```

第一種方式會幫我們自動載入所有檔案，不需要一個個引入；而第二種就要個別引入檔案，在此預設使用第一種方式
當指令下完後，你的終端機會出現這些文字

```bash
...
Loaded GHCi configuration from /private/tmp/ghci94387/ghci-script
*Main Lib>
```

但前綴有點長(引入的檔名都顯示出來)，我們再直接下個指令更改

```ghci
:set prompt "ghci>"
```

然後跑stack project預設的程式試試

```bash
ghci> main
someFunc
ghci> 
```

### Operator

一般常用的數學運算符用法都差不多

```haskell
ghci> 1 + 1
2
ghci> 1 - 1
0
ghci> 2 * 2
4
ghci> 2 / 2
1.0
ghci> 1 == 1
True
ghci> 1 == 2
False
ghci> 1 /= 1
False
ghci> 1 /= 2
True
```

其實這每一個運算符也都是Function，這些被兩個參數夾在一起呼叫的我們稱爲**中綴函數**，其餘則是**前綴函數**(大部分皆這種)

```haskell
ghci> max 5 1
5
ghci> mod 10 5
0
```

Haskell的Function在傳遞參數並不用逗號、括號來分割，純粹用空格來表示；當然也有括號的優先順序

```haskell
ghci> max 5 (max 10 1)
10
ghci> (succ 9) + (max 5 4) + 1
16
```

但這些前綴函數有時會讓人覺得不易讀，所以我們也可以將它們改成用中綴的方式寫，只要這樣加上**`**即可

```haskell
ghci> 10 `mod` 5
0
ghci> 10 `div` 5
2
```

### 函數

接下來的Code會直接寫在Main.hs裡頭，每當存檔後，只要下`:r`這指令就會重新幫我們載入
來寫一個`addTwo`的Function

```haskell
addTwo :: Num a => a -> a
addTwo n = n + 2
```

第一行是定義這個Function的Type，代表它接收一個型別爲*Num*的參數並同樣回傳*Num*(你也可以先不定義)；第二行就是這Function主要做的事情
`:r`重新載入後，如果剛剛你沒定義這Function的Type，可以用`:t addTwo`看見Haskell自動幫你定義了，`:i addTwo`也可試試看

```haskell
addBoth :: Num a => a -> a -> a
addBoth x y = x + y
```

### if/else

用法也大同小異，只是在Haskell中，`else`不可省略(爲了確保一定會回傳一個值)
不論是`5`/`x+y`/`if...then...else`都只是個表達式(就是返回一個值的程式碼)，所以可以輕鬆的在函數中使用

```haskell
doubleSmallNumber x = if x > 100
                      then x
                      else x*2

doubleSmallAndAddOne x = (if x > 100 then x else x*2) + 1
```

### List

這是我們很熟悉的資料結構，許多資料都用List來操作，但在Haskell中List是一種單型別資料結構，List內的元素皆要是同型別
`GHCi模式中可以用 let a = [1,2,3] 定義常量`

```haskell
-- let 定義
ghci> let temp = [1,2,3]
ghci> temp
[1,2,3]

-- 合併List
ghci> [1,2,3] ++ [3,4,5]
[1,2,3,4,5]

-- String其實就是一組字元的List而已
ghci> ['h','a'] ++ ['O','O']
"haOO"
ghci> "ha" ++ "OO"
"haOO"
```

在用`++`要特別注意，Haskell會遍歷整個List(符號左邊那個)，如果List長度大時會跑一陣子
單單插入一個元素可以使用`:`往List前端插入，這樣較高效

```haskell
ghci> 1:[2,3,4]
[1,2,3,4]
ghci> 1:2:[3,4]
[1,2,3,4]
```

一些List的函數

```haskell
-- 取List的值
ghci> [1,2,3,4] !! 2
3
ghci> head [1,2,3,4]
1
ghci> tail [1,2,3,4]
[2,3,4]
ghci> last [1,2,3,4]
4
ghci> init [1,2,3,4]
[1,2,3]
ghci> length [1,2,3,4]
4
ghci> null [1,2,3,4]
False
ghci> null []
True
ghci> reverse [1,2,3,4]
[4,3,2,1]
ghci> take 3 [1,2,3,4]
[1,2,3]
ghci> drop 2 [1,2,3,4]
[3,4]
ghci> minium [1,2,3,4]
1
ghci> maximum [1,2,3,4]
4
ghci> sum [1,2,3,4]
10
ghci> product [1,2,3,4]
24
-- 檢查元素是否在List裡
ghci> 4 `elem` [1,2,3,4]
True
ghci> 5 `elem` [1,2,3,4]
```

再來介紹List中一個好用的功能 - **Range**
欲表達一個1~20的List我們可以這樣做

```haskell
ghci> [1..20]
[1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20]
-- 字元也行
ghci> ['a'..'z']
"abcdefghijklmnopqrstuvwxyz"
-- 也可以給它間距
ghci> [1,3..20]
[1,3,5,7,9,11,13,15,17,19]
-- 但間距只能給一次
ghci> [1,3,5..20]
error: parse error on input ‘..’
```

我們也可以不標註界限，定義個無限長度的List - `[1,3..]`
由於Haskell是惰性的，所以並不會去取一個無限長度的值，它會等到你真的要取的時候，看你要多少才給多少

```haskell
ghci> take 10 [1,3..]
[1,3,5,7,9,11,13,15,17,19]
ghci>  take 14 (cycle "SOV! ")
"SOV! SOV! SOV!"
ghci>  take 10 (repeat 5)
[5,5,5,5,5,5,5,5,5,5]
```

接着也是個厲害的語法 - **List Comprehension**
這其實就是數學中*Set Comprehension*的概念，從既有的集合按照規則產生一個新集合

```haskell
ghci> [x*2 | x <- [1..10]]
[2,4,6,8,10,12,14,16,18,20]
ghci> [x + y | x <- [1..5], y <- [2,3]]
[3,4,4,5,5,6,6,7,7,8]
```

如果我們自己動手實現一個`length'`函數可以這樣

```haskell
length' xs = sum [1 | _ <- xs]
```

`_`代表我們不在乎它是什麼值

### Tuple

**Tuple**很像**List** - 將多個值存入一個容器中，我們剛剛有提到，List內的元素皆是同型別，且它的Type不會因爲內部的元素數目而有異
可是**Tuple**要求明確的內部數據數目，它的Type會取決於內部數據的數目&Type

```haskell
-- 可以發現Type的異處 --
ghci> :t [1,2,3]
[1,2,3] :: Num a => [a]
ghci> :t (1,2,3)
(1,2,3) :: (Num c, Num b, Num a) => (a, b, c)
ghci> :t ([1,2], 3)
([1,2], 3) :: (Num b, Num a) => ([a], b)
-- 不會有單元素的Tuple --
ghci> :t (1)
(1) :: Num p => p
```

我們先介紹針對*序對*Tuple的Function(後續會提及其他長度的Tuple)

```haskell
ghci> fst (1,2)
1
ghci> snd (1,2)
2
```

用List產生Tuple

```haskell
ghci> zip [1,2,3,4,5] [5,5,5,5,5]
[(1,5),(2,5),(3,5),(4,5),(5,5)]
ghci> zip [1 .. 5] ["one", "two", "three", "four", "five"]
[(1,"one"),(2,"two"),(3,"three"),(4,"four"),(5,"five")]
-- 如果兩個List長度不一，取小的
ghci> zip [1,2,3] [4,5,6,7,8]
[(1,4),(2,5),(3,6)]
-- 所以也可以處理無限的List
ghci>zip [1,3..] ["one", "two"]
[(1,"one"),(3,"two")]
```

實際的應用上，我們以三角形爲例

```haskell
let threeEdge = [ (a,b,c) | c <- [1..10], b <- [1..10], a <- [1..10] ]
```

我們必須再多給些條件才能讓它是三角形的集合(假設我要直角三角形)

```haskell
-- c是斜邊, b是長邊, a是短邊
let triangles = [ (a,b,c) | c <- [1..10], b <- [1..c], a <- [1..b], a^2 + b^2 == c^2 ]
```

最後我只想要周長爲24的三角形，所以再修改一下

```haskell
let finalTriangle = [ (a,b,c) | c <- [1..10], b <- [1..c], a <- [1..b], a^2 + b^2 == c^2, a+b+c == 24 ]
```

## 進階函數

語法提及的差不多，接着要持續深入Haskell中Function的精髓

### 模式匹配 (Pattern matching)

這是一個很實用的功能，~~真心推薦每個語言都帶有~~除了能幫我們Code寫的更簡潔外，取參數也變得更加容易
來個簡單的範例

```haskell
lucky :: (Integral a) => a -> String
lucky 7 = "LUCKY SEVEN!"
lucky x = "Sorry, you're out of luck..."
```

**Pattern Matching**會從上到下開始匹配，如果把最後一個匹配一切的模式移到最前面，那永遠會回傳*Sorry, you're out of luck...*
我們來實作個階乘函數，可以計算`10!`,`7!`之類的，而條件就是`0!`回傳**1**

```haskell
-- 還結合了遞迴在裏頭~
factorial :: (Integral a) => a -> a
factorial 0 = 1
factorial n = n * factorial (n - 1)
```

**Pattern Matching**的順序很重要，如果將兩個模式對調，那就是無窮盡的一直跑了...
我們再來把Tuple融合到模式匹配裡，以`相加兩個Vector向量`爲例

```haskell
--- 一般Function寫法
addVectors :: (Num a) => (a, a) -> (a, a) -> (a, a)
addVectors a b = (fst a + fst b, snd a + snd b)

-- Pattern Matching寫法
addVectors' :: (Num a) => (a, a) -> (a, a) -> (a, a)
addVectors' (x1, y1) (x2, y2) = (x1 + x2, y1 + y2)
```

這跟Javascript的*解構賦值*有點像，我們再舉個三元組的Tuple來試試

```haskell
first :: (a, b, c) -> a
first (x, _, _) = x

second :: (a, b, c) -> b
second (_, y, _) = y

third :: (a, b, c) -> c
third (_, _, z) = z
```

`_`就跟剛剛一樣，我們不在乎它是什麼~
再來要以**List**爲例子，這也可能是往後很常用到的技巧
`[1,2]`就是`1:2:[]`的結果，也可以是`1:[2]`，用變數表達來看即爲`a:b:c` or `x:xs`，這也是我們在模式匹配中針對List的寫法

```haskell
-- 改寫head，List的匹配至少要有一個元素，所以空的就拋出錯誤
head' :: [a] -> a
head' [] = error "Empty !"
head' (x:_) = x
```

再來改寫`length`試試

```haskell
-- 遞迴又用上了
length' :: (Num b) => [a] -> b
length' [] = 0
length' (_:xs) = 1 + length' xs
```

總之在用Pattern Matching搭配遞迴的時後要記得給予邊界條件，就像我們針對空List或是值等於**0**那樣，避免無窮下去

### Guards

**Pattern Matching**用來檢查一個值是否適合從中取值，而**guards**是用來檢查一個值的某項屬性是否爲真~~就是比較簡潔的if/else啦~~
先用個簡單的Bmi來試試

```haskell
bmiTell :: (RealFloat a) => a -> String
bmiTell bmi
    | bmi <= 18.5 = "You're underweight"
    | bmi <= 25.0 = "You're supposedly normal"
    | bmi <= 30.0 = "You're fat!"
    | otherwise   = "You're a whale, congratulations!"
```

guard利用豎線來表示，並且要記得縮排(不然會報錯)
他跟Pattern Matching一樣，由上到下開始撮合，最後要給一個`otherwise`來處理未列舉的狀況
Haskell定義Function後面不需要加上`=`，且對縮排有要求，當然也可以用之前提過的**中綴**方式來寫

```haskell
bmiTell' :: (RealFloat a) => a -> a -> String
weight `bmiTell'` height
    | weight / height ^ 2 <= 18.5 = "You're underweight"
    | weight / height ^ 2 <= 25.0 = "You're supposedly normal"
    | weight / height ^ 2 <= 30.0 = "You're fat!"
    | otherwise   = "You're a whale, congratulations!"
```

### Where

在`bmiTell'`裡我們重複了三次`weight / height ^ 2`，照理說應該要讓它有個常量可以暫存著，所以我們將它改成

```haskell
bmiTell' :: (RealFloat a) => a -> a -> String
weight `bmiTell'` height
    | bmi <= 18.5 = "You're underweight"
    | bmi <= 25.0 = "You're supposedly normal"
    | bmi <= 30.0 = "You're fat!"
    | otherwise   = "You're a whale, congratulations!"
    where bmi = weight / height ^ 2
```

要定義多個常量只要縮進就可以了

```haskell
weight `bmiTell'` height
    | bmi <= skinny = "You're underweight"
    | bmi <= normal = "You're supposedly normal"
    | bmi <= fat = "You're fat!"
    | otherwise   = "You're a whale, congratulations!"
    where bmi = weight / height ^ 2
          skinny = 18.5
          normal = 25.0
          fat = 30.0
-- 模式匹配版

...省略
    where bmi = weight / height ^ 2
          (skinny, normal, fat) = (18.5, 25.0, 30.0)
```

### let

let & where很相似，差在於`where`綁定是在函數底部，綁定`語法結構`，且所有guard在內的整個函數皆可看見
`let`綁定則是個`表達式`，像是定義局部變數，所以對不同guard不可見
但我們也可以想成，一個是放在前面，一個後面這樣~
既然let是綁定表達式，所以可以這樣寫

```haskell
ghci> 4 * (let a = 9 in a + 1)
40

-- 在其他結構裡
ghci> [let square x = x * x in (square 5, square 3, square 2)]
[(25,9,4)]

-- 用在List Comprehension
calcBmis :: (RealFloat a) => [(a, a)] -> [a]
calcBmis xs = [bmi | (w, h) <- xs, let bmi = w / h ^ 2, bmi >= 25.0]
```

### case of

在剛剛介紹了Pattern Matching後，你一定覺得莫名其妙，爲何還要有switch的功能？
其實**case**才是正宗的! **Pattern Matching**是語法糖來着~
給個範例看看就好

```haskell
head' :: [a] -> a
head' [] = error "Empty !"
head' (x:_) = x

head'' :: [a] -> a
head'' xs = case xs of [] -> error "Empty !"
                       (x:_) -> x
```

### lamda function

就是匿名函數，那些一次性的Function通常不會特地去定義它，我們可以這樣寫

```haskell
mapAndPlusOne :: Num a => [a] -> [a]
mapAndPlusOne arr = map (\x -> x+1) arr
```

### $ 函數呼叫符

```haskell
ghci> :t ($)
($) :: (a -> b) -> a -> b
```

這只是個呼叫函數的符號而已，以往都是用`空格`直接呼叫函數，了不起再加個括號決定優先權
用空格呼叫的函數是`左結合`，`f a b c`等於`(((f a) b) c)`，**$**是右結合的
假設今天有個`sum (map sqrt [1..10])`，我們可以改寫成`sum $ map sqrt [1..10]`，少了括號也清楚多了
附上個需要想想的例子，可以用`:t`來看看位什麼

```haskell
ghci> map ($ 3) [(4+),(10*),(^2)]
[7.0,30.0,9.0]
```

## 實例

到入門的最後一關總要練習實例，這邊主要以簡單的leetcode、Hackrank題型來講

### 快速排序(quick sort)

因爲我們要做排序，所以這邊要規定a爲`Ord Typeclass`的成員
主要邏輯就是：先取得比`頭部`小的數，然後**quick sort**，比`頭部`大的數，也是要**quick sort**
所以要怎取得比頭部大&小的數呢？ **List Comprehension**!

```haskell
quicksort :: (Ord a) => [a] -> [a]
quicksort [] = []
quicksort (x:xs) =
  let smallerSorted = quicksort [a | a <- xs, a <= x]
      biggerSorted = quicksort [a | a <- xs, a > x]
  in  smallerSorted ++ [x] ++ biggerSorted
```

簡潔有力，這就是*Functional Programming*的厲害之處

### 回文 (Palindrome Number)

給予一個**Int**並判斷它是不是回文的格式，例如, `1`,`121`,`4444`,`12321`
先別想著用`[Char]`來做，單純用數值去操作
首先，我們要判斷是不是回文，至少要比較**數字位數 除以 2**次 - `length / 2`
假設數字是*12321*，在知道總位數&數值的前提下，我只要遍歷個/十位數，就可以知道千/萬位數是否相等
但因爲**Int**並沒有實作**length**，所以我們要用Function算 - `calLength`
並且再實作一個比較的Function，這Function已知我們的數值n & 總位數len，我們傳入位數i讓它比較相對的位數是否相等 - `compare`

- **calLength**: 初始長度爲1，利用遞迴將長度算出來
- **compare**: 已知的數值n & 長度len(這裡利用Currying簡化)，取相對應的位數來比較，並回傳*Bool*
- **all**: 可以利用`:t`來看它的Type，在此它會將`檢查是否爲True`的Function，map到一個List上，全爲True即True，否則False

```haskell
palindrome :: Integer -> Bool
palindrome n
    | n < 0             = False -- 不考慮負數
    | otherwise         = all (== True) $ map (compare n len') [1..(len' `div` 2)]
    where len'          = calLength 1 n
          calLength len n
            | n >= 10   = calLength (len+1) (n `div` 10)
            | otherwise = len
          compare n len i =
            let high    = (n `div` 10 ^ (len - i)) `mod` 10
                low     = (n `div` 10 ^ (i - 1)) `mod` 10
            in  high == low
```

再來是~~偷工減料~~轉爲字串版...

```haskell
palindrome' :: Integer -> Bool
palindrome' n
    | n < 0         = False
    | str_n == re_n = True
    | otherwise     = False
    where str_n     = show n
          re_n      = reverse str_n
```

### 移動0 (Move Zeroes)

最後來題比較輕鬆的，給予一個`[Int]`，將數字0都移到List最後面
例如: `[1,4,0,3,0,1]`轉成`[1,4,3,1,0,0]`
用**Pattern Matching**加上**遞迴**能讓我們寫的很精簡(記得給予遞迴邊界)

```haskell
moveZeroes :: [Integer] -> [Integer]
moveZeroes (x:xs)
    | null xs = [x]
    | x == 0  = moveZeroes xs ++ [x]
    | otherwise = x : moveZeroes xs
```

## 總結

以上提到的只是Haskell最初的入門知識點，還有很多像是**Module**、**TypeClass**、**Monad**之類的學問，會在之後的文章有個說明
Haskell可以利用其他第三方Module、pakage建構出一個完整的服務*(I/O, Network...)*，並不單只是解些運算題而已
也推薦大家可以從[Haskell趣學指南](https://www.gitbook.com/book/mno2/learnyouahaskell-zh/details)當做入門點
未來還有很多驚喜~~挫折~~等著你呢~

## Reference

[Haskell趣學指南](https://www.gitbook.com/book/mno2/learnyouahaskell-zh/details)
[Hackage](https://hackage.haskell.org/)