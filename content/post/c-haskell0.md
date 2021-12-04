---
author: "sabanium"
title: "HaskellでCコンパイラもどきを作る #0"
date: "2021-12-03"
description: "環境構築"
tags:
- dev
- Haskell
---

まず環境構築をする。

### Haskellのインストール

[これ](https://www.haskell.org/downloads/)を読んで、GHCupとHaskell Tool Stackの一番上の方にのってるcurlコマンドを叩く。

```
curl --proto '=https' --tlsv1.2 -sSf https://get-ghcup.haskell.org | sh
curl -sSL https://get.haskellstack.org/ | sh
```

LSPは必要なので入れて、それ以外のものにも大体Yesと回答した気がする。なんかPとか押させるのが厄介。

### Haskell Language Serverの導入

vim-lsp-settingsはHaskell Language Server(以下HLS)に対応していないがvim-lsp-settingsの[issue](https://github.com/mattn/vim-lsp-settings/issues/309)を読むとそういうことが書いてあるので.vimrcを適当にいじるとうまくHLSが動く。

これでGHC+HLSが入った環境が準備できた。
