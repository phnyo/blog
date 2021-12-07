---
title: "The Road to Monads: Applicativeについて"
date: 2021-12-05T11:22:10+09:00
tags:
- Haskell
- 翻訳
author: sabanium
---

この記事は[The Road to Monads](https://oliverbalfour.github.io/haskell/2020/08/06/applicative-functors.html)を意訳したものであり、[RICORA Advent Calendar](https://adventar.org/calendars/6389)の9日目の記事です。

## Applicative Functor

[Functor](/post/functor)はWrapperで、fmapを使えば`(a -> b)->(f a -> f b)`という風にFunctorの要素に対して関数を適用できたのだった。

でもこれだと問題があって、例えば
```
f :: Int -> Int -> Int
f a b = a * b
```
みたいな関数を`Maybe Int`で置き換えて使おうと思ったときに使えない。
というか単純に一変数関数にしか使えん。
これはけっこうだるい。

そこで、その代替策がApplicativeなのだがこれもこれとして理解し難い。
まずは関数定義から見てみよう。
```
class functor f => Applicative f where
  pure :: a -> f a
  (<*>) :: f (a -> b) -> f a -> f b
```
ここで()に囲まれているのは演算子であることを思い出そう。
すると、まず`pure`はある型`a`から`Functor a`型に移すようなものだとわかる。
`pure`はfunctorのデフォルトであるべき（らしい）なので、簡単な例だと`Maybe`に対して
```
pure a -> Just a
```
である。どこが`pure`なのかはいまいちよくわからない。

Haskellでは関数も型なので、こういうこともできる。ある項+1を取れば、
```
pure (+1) -> Just (+1)
```
この`Just(+1)`の型は`Maybe(a -> a)`である。

うん。まあなんとなくはわかった感がある。

次は`(<*>)`について。
なにこいつ、なまこっぽいみたいな感想が先に立つけどまあうん。
実はよく見ると`fmap`に似ていることがわかる。
```
fmap :: (a -> b) -> f a -> f b
<*> :: f (a -> b) -> f a -> f b
```
ここでの違いは、もう関数がFunctorに埋め込まれているかどうかだけ。

だから実は
```
pure a -> f a
```
を思い出せば、型システムから
```
fmap f x = pure f <*> x
```
であることがわかる。

具体例を作ってみよう。
```
fn :: a -> b
```
に対して
```
pure fn :: f (a -> b) 
```
なので、
```
pure fn <*> :: a -> f b
fmap fn :: a -> f b
```
がいえる。ふう。もちろん、ここで`f`はFunctorの型で`a`は関数の引数である。

いや、なんの役に立つねん。
うーんと、実はなまこ`<*>`の重要性は関数を取ることにある。

`<*>`の引数は`f(a -> b)`といってるけど、この`b`が`c -> d -> e`型だった場合について考えよう。
一回`<*>`を適用すると、返り値は`f b`だけどこれは実際には`f( c -> d -> e )`の型。
あれ？これってさっきと同じ形じゃない？
そう、そのとおりで、`c`を`a`、`d -> e`を`b`と置き換えると、`<*>`をもう一回適用することで`f(d -> e)`がでて、結局`f e`の値が出ることになる。
その過程で、入力に`f a`、`f b`、`f c`、`f d`の値が必要になる。
これって引数が多い関数じゃない？そう。
だから再帰的なアプローチを関数に対して取ることで（！？）うまく引数の多い関数を実装できるような仕組みができているのだった。

### 具体例

具体的な例を見よう。
```
pure (+) <*> Just a <*> Just b
== (Just (+) <*> Just a) <*> Just b
== Just (+a) <*> Just b
== Just (a+b)
```
これは別に`fmap`でもいい気もする。

こんな関数を考えると
```
quadratic :: Int -> Int -> Int -> Int -> Int
quadratic a b c x = a * x ^ 2 + b * x + c
```
`pure`と`fmap`で書いたこれらが等価であることがわかる。
`<$>`は`fmap`の略記。
```
quadratic <$> (Just 3) <*> (Just 4) <*> (Just (-2)) <*> (Just 9)
pure quadratic <*> (Just 3) <*> (Just 4) <*> (Just (-2)) <*> (Just 9)
== Just(quadratic 3 4 (-2) 9)
```
他に、`Applicative Maybe`についても考えることができる。
`Nothing`が片方の項にあるなら`Nothing`、それ以外は`Just`を返せばいいので
```
instance Applicative Maybe where
  pure = Just
  (Just f) <*> (Just x) = Just (f x)
  _ <*> _ = Nothing
```
はい。いい感じだね。

具体例についてはまたどっかで追記するかもしれない。

### 法則について

守らなきゃいけない法則がある。
1. 恒等的な関数の存在
```
  pure id <*> x == x
```
これはFunctor側の`fmap`で確かめられていても、こっちで確かめなきゃいけない。

2. 合成則
```
  pure (.) <*> u <*> v <*> w == u <*> (v <*> w)
```
うーんと、何が言いたいかいまいちよくわからない感じがするけど`(u . v) w == u (v w)`とFunctorなしの表記にするとわかりやすいのかな。
ここで`u`と`v`はapplicative functorの中の関数なので、先に関数を合成して計算しても、関数の結果に関数を適用しても値が同じことを保証しなければならないことを言っている。
参照透過性とかと関係あるかは不明。

3. 準同型
```
  pure f <*> pure x == pure (f x)
```
気持ち的には自明っていいたい！が証明は必要。
`pure f <*> y`とすれば、`f y`から`f (f x)`より`pure (f x)`っぽい。

4. 交換則
```
  f <*> pure x == pure ($ x) <*> f
```
割と意味わからん。ここはあとで追記。

### 補足

要はApplicative Functorは一つの国みたいなもので、`pure`はそこに連れて行くためのコンストラクタ、`<*>`はその中で関数を適用するための演算子として捉えるとわかりやすいのかな。

おわり。
