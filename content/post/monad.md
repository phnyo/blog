---
title: "The Road to Monads: Monadについて"
date: 2021-12-06T11:20:40+09:00
tags:
  - Haskell
  - 翻訳
  - Adv2021
author: sabanium
---

この記事は[The Road to Monads](https://oliverbalfour.github.io/haskell/2020/08/08/understanding-monads-haskell.html)を意訳したものであり、[RICORA Advent Calendar](https://adventar.org/calendars/6389)の11日目の記事です。

## Monad

今までの流れを振り返ると、[Functor](/post/functor)は何らかの値のWrapperであり、`fmap`によってその要素に関数適用ができた。
[Applicative Functor](/post/applicative)はそれをさらに発展させて、関数自体をFunctorで包むことによって、複数の引数を取る関数を実装できるようにしていたのだった。

さて、何が足りないのだろう？
実は、Applicative Functorだけでは関数を複数組み合わせることが難しい。
例えば、`f :: a -> b`と`g :: b -> c`があったときに、Applicative Functorだと`a -> f b`、`b -> f c`に写すことはできても`a -> f c`に写すことができない。
こういう関数の使い方をする機会は割と多い気がするし、こんな感じで書きたくない？という話。

簡単にいえば、MonadはFunctorの中での関数合成を可能にする構造。
しかし、これだけだとMonadが実際に何をするかいまいちわからないので以下に書いていく。
導入として、大体FunctorみたいなものだとしてMonad`m`を使うことにしよう(とはいっても、実装上は拡張したApplicative型なんだからFunctorの拡張として捉えるほうが適切かもしれない）。

じつはMonadは他の必要っぽいことも包摂してくれている。
1. Monad上の関数の繰り返し適用に対して、Monadを圧縮する。`a -> m b`に対してまたfmapをつかって`m`を適用すると、`m a -> m (m -> b)`になってしまう。
例えば、`a -> Maybe b`なら`Maybe a -> Maybe ( Maybe b )`的な。
これを圧縮して、繰り返し適用しても`a -> Maybe b`の型を作れるようにする。
2. さっきの例。
`a -> m b`と`b -> m c`から`a -> m c`の関数を作りだせる。
3. 逆に`m a`が与えられたとき、`a -> m b`の関数を使えるようにする。

### 型の圧縮

まあこれだけだとよくわからないので、具体例を考えよう。
`1 / sqrt (x)`を`Maybe`を返すとして考える。
この関数を二つの関数`1 / x`と`sqrt(x)`の合成で考えるとすると、例えば`1/0`はundefinedなので、`1 / x`で`x = 0`なら`Nothing`を返したい。
これは`sqrt(x)`でも一緒。`x < 0`なら`Nothing`を返したい。
そこで、これらを実際に書いてみるとこんな感じ。
```
safeInverse :: Double -> Maybe Double
safeInverse 0 = Nothing
safeInverse _ = Just(1/x)

safeSqrt :: Double -> Maybe Double
safeSqrt x = case signum x of
  (-1) -> Nothing
  _ -> Just (sqrt x)
```
ここで`signum`は符号を返す関数。

まあこれだけで完結するんだったらいいんだけど、もっとでかいプログラムの一部に組み込むなら`Maybe`の場合分けをし続けることになる。
めんどくさい犬の散歩みたいだ。
`fmap`をつかってもいいかもしれないけど、`fmap`を使うと`1 / sqrt(x)`の関数の型が`a -> Maybe (Maybe b)`になってしまう。
これをもっとでかい関数の一部として使うとすると...大変なことになりそうだ。

それを避けるために、`join`という関数を考える。
`Maybe`に対してはこんな感じで書ける。
```
join :: Maybe (Maybe a) -> Maybe a
join (Just x) = x
join Nothing = Nothing
```
これは圧縮そのものだから、実際に`1 / sqrt(x)`を書くと
```
sqrtInverse :: Double -> Maybe Double
sqrtInverse = join ( fmap safeInverse (safeSqrt x))
```
こんな感じ。
確認だけど、`fmap SafeInverse (safeSqrt x)`が`Double -> Maybe (Maybe Double)`型の関数なので、`join`を使うと`Double -> Maybe Double`になる。

この`join`は実際にこんな感じに一般化できるらしい。
```
join :: Monad m => m (m a) -> m a
```

### 関数の合成

この`sqrtInverse`を関数の合成の観点から見てみよう。
さっきと同じように普通の値を取ってMonad内部の値を返すような関数を二つ考えて、それらを普通の値を取ってMonadの値として返す関数を想定するとき、`>=>`という演算子を使うことができる。
例えば`f :: a -> m b`と`g :: b -> m c`があるとき、`f >=> g`は`a -> m c`という型になる。
この`>=>`演算子は魚に似ている（？）のでfish演算子と呼ばれ、この合成自体はKleisli(クライスリ)合成というらしい。

実際にこのfish演算子が使われている所を見てみよう。
```
(>=>) :: (a -> Maybe b) -> (b -> Maybe c) -> (a -> Maybe c)
f >=> g = \x -> case f x of
  Just x -> g x
  Nothing -> Nothing
```
うん。
まず一行目はfish演算子を定義した通りの定義になっている。
そこで、二行目以降ではMonadがMaybeなので、`f x`は`Just x`か`Nothing`を返し、`Just x`ならunwrapして`g x`を返すようにパターンマッチングしてあげている。

このfish演算子を実際にさっきの`sqrtInverse`に使おう。
```
sqrtInverse :: Double -> Maybe Double
sqrtInverse = safeSqrt >=> safeInverse
```
だいぶ短くなったし、これで明示的にパターンマッチングの処理をしなくてよくなったのでだいぶ見通しが良くなるはずだ。

でもこれにはまだ問題がある。
Monadの中の値を使うときに恒等射的な関数を考える必要が出てくるのだ。
例えば`m a`という値に対して`m a -> a`という関数を作り、fish演算子を使う必要がある。
これだとまた型に合わせるための関数を作って見通しがまた良くなくなってしまう。

実際にそういう例を見てみよう。
`const a b = a`という関数を考えれば、`Just 5`を計算するために
```
(const (Just 4) >=> (\x -> Just (x+1))) ()
```
とする必要がある。

ごちゃごちゃしててわかりにくいから型を確かめよう。 
`\x -> Just(x + 1)`は`Int -> Maybe Int`という型で、`const Just 4`は`Num a => a -> Maybe b`という型だ。
あれ、`const`って引数2つ取るんじゃないの?と思うんだけど、それはカリー化されたものを見ているだけで、実際には1変数だけ取ることもできる（もちろん、`const a`は`const a b`とは違う関数）。

それで`>=>`が使えるが、型チェックをすると`(const Just 4 >=> (\x -> x + 1)) :: Num c => b -> Maybe c`なので、もう一つなにかの引数が必要っぽいことがわかるので、入れてあげると`Just 5`が出る。
これはなんでもいい。
例えば`'c'`でも`()`でもいい。

### 連続的に計算させる

では、別の演算子`(>>=) :: m a -> (a -> m b) -> m b`を考えよう。
これをbind(演算子?)と呼ぶ。

Maybeに対して実装するとこんな感じ。
```
(>>=) :: Maybe a -> (a -> Maybe b) -> Maybe b
x (>>=) f = case x of
  (Just x') -> f x'
  Nothing -> Nothing
```
ここで`x`は`Maybe x'`という型だとした。

さっきの`sqrtInverse`をまた拾ってこよう。
```
sqrtInverse :: Double -> Maybe Double
sqrtInverse x = safeSqrt x >>= safeInverse
```
他の例としてこういうのもある。
```
Just 3 >>= (\x -> Just (x + 1))
```
これはもちろん`Just 4`を返す。

実際、bindはfish演算子や`join`より使いやすい。
でも、結局bindでやっていることはfish演算子や`join`を使って遠回りしてできることと同じだ。

bind、fishと`join`からお互いを作れることを示そう。
ここで`const`と`id`という関数を使っているが、それぞれ`const a b = a`と`id a = a`と定義される関数である。

思い返せば、それぞれの型はこんな感じであった。
```
join :: m (m a) -> m a
(>=>) :: (a -> m b) -> (b -> m c) -> (a -> m c)
(>>=) :: m a -> (a -> m b) -> m b
```
そこで、まず`join`を作ってみよう。
```
join x = x >>= id
join x = const x >=> id
```
上は`m (m a)`型の変数を取っている。
`id`は`a -> a`の関数なので、


