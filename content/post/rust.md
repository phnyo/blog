---
title: "Rust覚え書き"
date: 2021-12-23T16:42:30+09:00
tags:
 - rust
 - memo
---

### 入出力

`String`型の`buf`への入力

```
use std::io::Read;

fn main() {
    let mut buf = String::new();
    std::io::stdin().read_to_string(&mut buf).unwrap();
}
```

`i64`型の配列への入力

```
let mut iter = buf.split("\n");
let v1:Vec<i64> = iter.next().unwrap().split_whitespace().map(|x| x.parse::<i64>().unwrap()).collect();
```
もしくは可読性を上げて
```
let mut iter = buf.split("\n");
let v1:Vec<i64> = iter.next().unwrap().split_whitespace().map(|x| {
                    x.parse::<i64>().unwrap()
                  }).collect();
```

### 配列

VectorでもArrayでもいい。`Collect()`すると`Vec<T>`になる。

`Array`の初期化

```
分かっている状態での初期化
let arr: [T; num of elements] = [直書き]
繰り返したいとき
let arr: [T; num of elements] = [elem; num of elements]
```

### debug

type check

```
use std::any::type_name;

fn type_of<T>(_: T) -> &'static str {
    type_name::<T>()
}
```
