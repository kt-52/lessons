# Java Stream API 特講 第二章 ~ 関数型interface

## Strategyパターンの弱点

前章では、Javaではメソッドそのものの受渡しができないこと、Strategyパターンを用いることでメソッドをインスタンスで包んで受渡しすることができることを説明しました。

しかし、Strategyパターンには弱点もあります、それは記述量が多いことです。

Strategyパターンは、前章で解説した通り

1. interfaceを定義する
1. そのinterfaceを実装したclassを定義し、それに渡したいメソッドを実装する
1. そのclassをnewして引数に渡す

の手順が必要です。

しかし、FizzBuzzの `i % 3 == 0` や `i % 5 == 0` 程度の情報を受渡しするのに、interface1つと実装classを実装数分定義するのはあまりに冗長です。

そこで、これらを簡単に記載するための仕組みが関数型interfaceとlambda式・メソッド参照です。


## 関数型interface

関数型interfaceとは、interfaceであって抽象メソッドが1つだけのものをいいます。

defaultメソッドやstaticメソッドはいくつ持っていても構いません。

次章で説明するlambda式やメソッド参照の対象の型になることができること、 `@FunctionalInterface` のアノテーションをつけられること、の2点以外は普通のinterfaceと変わりません。

なお、抽象メソッドを持たないか2個以上持つinterfaceに `@FunctionalInterface` のアノテーションをつけるとコンパイルエラーになります。

ちなみに、前章で出てきた `Condition` は抽象メソッドが1つだけなので関数型interfaceです。

```java
// アノテーションがつけられる
@FunctionalInterface
interface Condition {
  boolean judge(int i);
}
```


## 標準ライブラリ `java.util.function`

関数型interfaceは `java.util.function` のパッケージに汎用的なものが定義されています。

これらを利用することにより、Strategyパターンの手順「1. interfaceを定義する」が不要になります。

- 以下を参照
https://docs.oracle.com/javase/jp/8/docs/api/java/util/function/package-summary.html

この `java.util.function` には大量のinterfaceが定義されていて何やら難しそうに見えます。

しかし、これらは全て「定義されている抽象メソッドの引数の数や引数・返り値の型が異なるだけ」のinterface群です。

* 引数なし

`Supplier<T>`（引数を受け取らずT型の返り値を返す）のみです。

* 引数が1つ

`Function<T, R>`（T型の引数を受け取ってR型の返り値を返す）が最も汎用的な型になります。

| interface | 名前の意味 | Functionでいうと |
|---|---|---|
| Function\<T, R\> | 関数 | Function\<T, R\> |
| Consumer\<T\> | 消費 | Function<T, Void> |
| Predicate\<T\> | 断定 | Function\<T, boolean\> |
| UnaryOperator\<T\> | 操作 | Function\<T, T\> |
| （参考）Supplier\<T\> | 供給 | Function\<Void, T\> |

* 引数が2つ

`BiFunction<T, U, R>`（T型・U型の引数を受け取ってR型の返り値を返す）が最も汎用的な型になります。

| interface | 名前の意味 | Functionでいうと |
|---|---|---|
| BiFunction\<T, U, R\> | 関数 | Function\<T, U, R\> |
| BiConsumer\<T, U\> | 消費 | Function<T, U, Void> |
| BiPredicate\<T, U\> | 断定 | Function\<T, U, boolean\> |
| BinaryOperator\<T\> | 操作 | Function\<T, T, T\> |
| （参考）Supplier\<T\> | 供給 | Function\<Void, Void, T\> |

* 引数が3つ以上

`java.util.function` には定義されていません。

自作する必要があります。

* プリミティブ型の特化型

また、これらに加えてプリミティブ型用のinterfaceがあります。

例) `IntFunction<R>` は `Function<Integer, R>` と同等です。

ただし、 `Integer` と `int` 間の変換はboxing・unboxingがかかるので `IntFunction<R>` の方が効率的です。

**これらを暗記する必要は全くありません**

**必要に応じて一覧から適切なものを選択できれば十分です**
