---
title: "The Road to Monads: Functorについて"
date: 2021-12-04T22:45:30+09:00
tags:
  - Haskell
  - 翻訳
  - Adv2021
author: sabanium
toc: true
---

この記事は[The Road to Monads](https://oliverbalfour.github.io/haskell/2020/08/04/functors-in-haskell.html)を意訳したものです。
あんまり難しいことは言っていないと思うんだけど、あまり頭に入らなかったので簡単に翻訳してみた。

## Functor
### fmap

Functorって何? 内部の値にアクセスできるようにしたWrapper。具体例: IO、Maybeなど。
圏論でいうところの関手。

コードでの具体例：
```
addMaybe :: Num a => Maybe a -> Maybe a -> Maybe a
addMaybe (Just a) (Just b) = Just (a + b)
addMaybe _ _ = Nothing
```
確かに内部の値にアクセスしている。まあこれは恣意的な関数だけど。
ただし、これを一般化するといい感じになってくれる。
要は内部の値にアクセスすることが重要で、例えばf :: Int -> Intはこういう関数を使うことでMaybe Intにも使えるようになったりする。

次に、こんな関数を考える。
```
fmapList :: (a -> b) -> ([a] -> [b])
fmapList f [] = []
fmapList f (x:xs) = f x : fmapList f xs
```
これはmap関数と変わらない。
ある関数をリストのすべての要素に適用するだけだ。

では、これを一般化するとどうなるだろう？ 
つまり、こんな感じの関数たちを統一的に記述できるだろうか？
```
f :: (a -> b) -> ((Just a) -> (Just b))
f2 :: (a -> b) -> ([a] -> [b])
```
その関数が、functorと紐付けられたfmapである。
技術的にはfunctorはFunctor型クラスでfmapと紐付いたインスタンス。
ただし、満たすべき条件があり、それがこれ。

  1. fmap id == id (恒等的であること)
  2. fmap (f . g) == fmap f . fmap g (分配的であること)

じゃあlistとMaybeの実装を見てみよう。

PreludeのリストのFunctorの実装はこんな感じ。
```
instance Functor [] where
  fmap = map
```
上で見たとおり、fmapListはmapと同じなのだった。
mapはpythonとかの同名の関数と同じで、ある関数をリストの全要素に適用する関数。
いや、さすがにこれだとわからなくない？

次にMaybeを見よう。
```
instance Functor Maybe where
  fmap f (Just x) = Just (f x)
  fmap _ Nothing = Nothing
```
こっちの方が解釈はしやすい。fとJust xがあるなら、返り値（？）を(Just f x)として呼ぶことにしている。

まあまあわかるような気もする。
他には、
```
data BinaryTree a = Node a (BinaryTree a) (BinaryTree a) | Leaf a
```
こんな感じのBinaryTree型を使うとき、fmapは
```
instance Functor BinaryTree where
  fmap f (Node x left right) = Node (f x) (fmap f left) (fmap f right)
  fmap f (Leaf x) = Leaf (f x)
```
という感じになる。

Functorというかfmapのいいところは、Wrapperの存在を意識せずに書けるところっぽい。

