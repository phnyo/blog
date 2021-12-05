---
author: "sabanium"
title: "Haskell覚え書き"
date: "2021-12-04"
description: "構文等"
tags:
- dev
- Haskell
---

構文解析のためのモナド等についてまとめる。完全に自分用。

### Haskellの文法について

よく見るやつ

- 型拘束
```
(Ord a) => a -> a
```

- 要素の存在
```
(x:xs)
```

