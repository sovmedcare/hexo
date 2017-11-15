---
title: Docker基礎架設(以Python Scrapy為例)
date: 2017-11-15 17:00:00
tags: [Docker]
author: Tu-Szu-Chi
---

# 簡介

**Docker**的介紹和教學已有相當多的資源，但當我以為可以快速地建立環境、連上DB，卻不盡然
許多資源很棒，寫得很詳細，但正因對每一項解釋太詳細以致我無法很快找到我要的知識點，故在此提供這一回摸索後，我認為能較快建立環境的流程。

# 環境

因一直想寫寫爬蟲，較著名的Framework是基於Python的 - **Scrapy**，但其實Scrapy之外還要安裝蠻多東西的，故能用Docker幫我們整合當然再好不過

## 安裝

1. Docker
    * [Mac](https://store.docker.com/editions/community/docker-ce-desktop-mac)
    * [Windows](https://store.docker.com/editions/community/docker-ce-desktop-windows)
2. [Kitematic](https://kitematic.com/)
  這是一個Docker的GUI工具，個人很推薦剛入門者使用

都安裝好後記得確認下Docker是否正常運行著，並打開Kitematic，會要求輸入Docker帳號密碼，沒有的可以申請一組
> Mac使用者可看上方工具列小鯨魚點開，會亮綠燈且*Docker is running*

## 部署
總共會有三個Container來做示範，Scrapy/Postgres/Adminer
### Scrapy

接著要建立Scrapy的Container，在搜尋框輸入「Scrapy」第一個就會是了(*from scrapinghub*)
點一下三個點點的*More button*會看到可以選擇Tag/Network，先選擇`1.1-py3-latest`的Tag讓後續我們的版本一致，選好後就按下**Create**即可
> 因我在run流程第二遍時有發生HTTP 500 error在這步驟，可以參考[這篇](https://github.com/docker/kitematic/issues/3260)，登出Kitematic的Docker Account

載好後Kitematic就會自動run這個Container，可以在Setting看到些資訊
![Index](https://i.imgur.com/gDOrbxO.png)

接著點擊上方的**EXEC**會將Container啟動在我們的終端機上，輸入`scrapy`並按下Enter確認Container正常運作
> 可以先點左下方齒輪(設定)，將*Exec command shell*選擇bash會比較習慣

### Postgres

再來按**New**去搜尋`Postgres`並選擇第一個官方的Image(在此是用10.1版本)，一樣好了後利用**EXEC**在終端機上開啟並輸入`psql`
會有以下錯誤訊息

```shell
psql: FATAL:  role "root" does not exist
```

改成輸入預設的帳號就可以了

```shell
psql -h localhost -U postgres
```

要離開psql就輸入`\q`

### Adminer

接著**New**一個adminer的Container(官方的Image)，選擇latest版，方便我們之後檢查是否有連上DB
好了後可以到Settings -> Hostname/Ports 確認下這Container的Published IP (*有可能顯示的是localhost*)
![Adminer](https://i.imgur.com/pGDA7hC.png)
直接點藍色IP字段就會將Adminer打開在瀏覽器了
再來要輸入Server位置，我們要在Postgres的command line輸入

```shell
ip address
```

會得到類似以下資訊
![Ip address](https://i.imgur.com/w9GqGCC.png)
第二段的`172.17.0.3`就是Postgres在Docker內網的位置
為了要管理Container的Network，可以到Settings -> Network檢查該Container是屬於哪一組網路(*預設是bridge*)
要在同一組網路下連內網IP才有用
所以我們的三個Container都是在bridge情況下，要連對方用內網IP即可
故在Adminer介面只要輸入剛剛得到的Postgres內網IP & Username，按下Login就會連上了
![Adminer](https://i.imgur.com/RgZtEIa.png)
環境架設就先到這裡！

# 專案

再來就拿實際專案來做示範，可以先clone我個人抓證交所資料的[專案](https://github.com/Tu-Szu-Chi/stockParser/tree/docker-test)來測試
> branch 選擇 docker-test

要讓Container讀取本機端的檔案，要設定`Volumes`，而在Kitematic上不是每個Image都可以直接到Settings -> Volumes 去設定
> 本次的Scrapy就不行，Postgres可以

所以要教大家利用指令來啟動Container & 指定Volumes(原有的Scrapy Container不需要了)

## Step 1

先在終端機上輸入指令，找到我們的`scrapinghub/scrapinghub-stack-scrapy` Image，並記住Image ID

```shell
docker images
```

## Step 2

在終端機輸入

```shell
docker run -it -v ~/stockParser/twse/:/twse [Image ID] bash
```

`-i`是讓Container的標準輸入保持打開；`-t`是讓Docker分配一個虛擬終端(pseudo-tty)並綁定到Container標準輸入上
`-v`即是Volumes的設定，冒號左邊是本機端位置，要用絕對路徑，冒號右邊是Container內的路徑
按下Enter後我們就建立了一個新的Container，並可在Docker內讀取到我們的專案，於Kitematic中可以重整Containers List看我們剛剛建立的容器(Mac就按 **Cmd+R**)
> 這裡也有個插曲，當按重整沒有任何新的容器出現時，我要用管理員權限打開Kitematic才看得到剛剛建立的Container，但Postgres/Adminer就看不到了
![New Container](https://i.imgur.com/W4H8f0t.png)

## Step 3

接著在Scrapy的Container輸入`ls`就看得到twse資料夾，我們先`cd`進去，然後先安裝要用的Module
```shell
pip install -r requirements.txt
```

再到專案底下的/twse/twse/settings.py 底部的DATABASE填入我們`Postgres`的設定
> 如果發現你的DB和Scrapy是不同的Published IP
那你Scrapy要連DB就輸入Postgres的Published IP:Port
如果都一樣者只要先用`ip address`找Postgres內網ip再輸入即可
![IP:Port](https://i.imgur.com/hFO32iA.png)

記得到adminer去新建一個**database**，在此預設名字為*twse*

## Step 4

最後輸入以下指令，就會開始抓2017/10/18 電子工業的資料了
```shell
scrapy crawl stock
```

# 結語

以上是個人認為剛開始用Docker會想嘗試的地方，Docker還有很多強大的用途直得我們去學習
也歡迎有興趣的人可以一起完成這twse專案

# 參考連結

[Docker Gitbook](https://philipzheng.gitbooks.io/docker_practice/content/)