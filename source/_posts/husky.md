---
title: husky + git hook
date: 2017-10-24 10:44:02
tags: [javascript]
author: tancc
---

git中，許多的操作指令都有所謂的`hook`，提供給我們先做一些前置作業再去真正執行git的指令。較常用`hook`的像是`pre-commit`, `pre-push`等等，通常會用這些`hook`搭配其他套件來做程式碼的規範檢查和自動化測試，來避免不好的程式碼推上repository。

git所提供的`hook`放在`<project>/.git/hooks`裡面，可以自行寫`shell`或是其他腳本語言去修改。不過這樣的方式比較麻煩，所以我們是透過`husky`，將指令定義在`package.json`來達成的。

# husky

[husky](https://github.com/typicode/husky)的這套件的描述就是

> Git hooks made easy

有多簡單呢？假設現在我們要在commit之前先通過lint檢查，push之前通過測試，只要兩個步驟

1. 安裝 `husky`

`yarn add husky --dev`

2. 修改`package.json`，定義`precommit` hook

```json
{
"scripts": {
"test": "jest",
"lint": "standard",
"precommit": "yarn lint",
"prepush": "yarn test"
},
}
```

# `lint-staged`

不過通常我們不會希望每次commit之前，都對所有的js檔案做lint，只要對這次修改的檔案去檢查即可。

這時候就可以搭配[`lint-staged`](https://github.com/okonet/lint-staged)來使用。

1. 安裝 `lint-staged`

`yarn add lint-staged --dev`

2. 修改`package.json`，加入`lint-staged`設定

```diff
{
"scripts": {
"test": "jest",
"lint": "standard",
+   "precommit": "lint-staged",
"prepush": "yarn test"
},
+ "lint-staged": {
+   "*.js": ["standard --fix", "git add"]
+ }
}
```

`lint-staded`可以不只做一件事，我們可以再順便加上css檔案的lint, format

```diff
{
"lint-staged": {
+   "*.js": ["standard --fix", "git add"],
+   "*.{css,less,scss,sss}": [
+     "stylefmt",
+     "stylelint",
+     "git add"
+   ]
}
}
```

# 參考資料

- [husky](https://github.com/typicode/husky)
- [`lint-staged`](https://github.com/okonet/lint-staged)

