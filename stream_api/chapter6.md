# Java Stream API 特講 第六章 ~ Stream API 終端操作

プリミティブ特化型StreamについてはInt・Double・Longの3つがありますが、内容が重複するためここでは `IntStream` のみを例に挙げます。


## 終端操作

### 重要度 高

- `Stream#collect(collector)`

最もよく使われる集計メソッドだと思います。

集計方法を表す `Collector` インターフェースを実装したインスタンスを引数として渡すのですが、 `Collectors` クラスに主要な集計方法を実装したメソッドが用意されているため、事実上そこから選択することになります。

もちろん、 `Collector` インターフェースを自分で実装することもできますが、かなり面倒なのでおすすめしません。

`Collectors` には多数の集計方法が実装されていますが、 `counting` ・ `filtering` ・ `mapping` などのstream中間操作と内容が重複するものも多いです。

以下ではよく使うと思われるものを紹介します。

```java
public static void main(String[] args) {
  var languages = List.of("Java", "C#", "Python", "Kotlin", "Swift", "Elixir");

  // 平均値
  // mapToInt -> averageより1行減る
  languages.stream()
    .collect(Collectors.averagingInt(String::length));
  // 4.83333...

  // group化
  // ここでは文字数をキーとし、名前のリストが値であるMapになる
  // Mapが返るので、entrySet -> streamとすることでメソッドチェーンが継続できる
  languages.stream()
    .collect(Collectors.groupingBy(String::length));
  // {2=[C#], 4=[Java], 5=[Swift], 6=[Python, Kotlin, Elixir]}

  // 文字列連結
  // 区切りや先頭末尾につける文字列を指定できる
  languages.stream()
    .collect(Collectors.joining(" & ", "【", "】"));
  // 【Java & C# & Python & Kotlin & Swift & Elixir】

  // コレクション化
  // ここではArrayList<String>に詰めなおしている
  // Javaのコレクションは設計が悪いので、実装クラスを明示した方が良い
  languages.stream()
    .collect(Collectors.toCollection(ArrayList::new));
  // [Java, C#, Python, Kotlin, Swift, Elixir]

  // 不変リスト化
  languages.stream()
    .collect(Collectors.toUnmodifiableList());
  // [Java, C#, Python, Kotlin, Swift, Elixir]
}
```

- `Stream#allMatch(predicate)`
- `Stream#anyMatch(predicate)`
- `Stream#noneMatch(predicate)`

引数で与えられた `predicate` によってstreamの各要素を判定した上で、以下の通りboolean値を返します。

`allMatch` は全ての要素についてtrueが返ったときにtrueを返し、それ以外の場合falseを返します。

`anyMatch` はいずれかの要素についてtrueが返ったときにtrueを返し、それ以外の場合falseを返します。

`noneMatch` は全ての要素についてfalseが返ったときにtrueを返し、それ以外の場合falseを返します。

streamが空の場合には `allMatch` と `noneMatch` はtrueを、 `anyMatch` はfalseを返します。

また、これらは短絡評価されます。

例えば、 `allMatch` ではfalseが返った要素が現れた時点で判定を打ち切りfalseを返します。

```java
public static void main(String[] args) {
  var languages = List.of("Java", "C#", "Python", "Kotlin", "Swift", "Elixir");

  languages.stream().allMatch(language -> language.contains("a"));
  // false

  languages.stream().anyMatch(language -> language.length() == 4);
  // true

  languages.stream().noneMatch(String::isEmpty);
  // true
}
```

- `Stream#count()`

streamの要素数を返します。

filterやdistinctなどの要素を切り詰める中間操作と組み合わせて使うことが多いでしょう。

なお、int型ではなくlong型で返されます。

```java
public static void main(String[] args) {
  var languages = List.of("Java", "C#", "Python", "Kotlin", "Swift", "Elixir");

  languages.stream()
      .filter(language -> language.length() == 6)
      .count();
  // 3
}
```

- `Stream#reduce(accumulator)`
- `Stream#reduce(identity, accumulator)`

streamの各要素に `accumulator` 関数を適用して集計します。

引数1つのものは先頭要素を初期値として2つ目の要素から、引数2つのものは `identity` を初期値として先頭要素から `accumulator` 関数を適用します。

これらはstreamの要素の型でしか集計できません。

- `Stream#reduce(identity, accumulator, combiner)`
- `Stream#collect(supplier, accumulator, combiner)`

この2つは第一引数を初期値として先頭要素から `accumulator` 関数を適用し、並列実行時の各結果の結合に `combiner` 関数を使用します。

これらは以下の2点で異なりますが機能として大差はありません。

1. 第一引数について、 `reduce` は初期値そのものを渡すのに対し、 `collect` は初期値を生成する `Supplier` を渡す点
1. 第二引数について、 `reduce` は `accumulator` 関数の返り値を結果として集計するのに対し、 `collect` は初期値のインスタンスに結果を集計していく点

`accumulator` 関数が集計結果を都度返り値で返す場合は `reduce` 、集計結果を一つのインスタンスに追加していく場合は `collect` が合っています。

`reduce` も `collect` も「集計する」という汎用的なメソッドなので、メソッド名からどういう集計をするのかが読み取れません。

できる限りわかりやすい名前の付いた特化型の集計メソッド（ `max` ・ `count` ・ `anyMatch` などや、定義済み `Collector` を渡す `collect`）を使うべきです。

これは任意の型で集計することができます。

```java
public static void main(String[] args) {
  String numbers = Stream.iterate(0, number -> number + 1)
    .limit(10)
    // 初期値は空の文字列
    // 各要素を連結していき
    // 並列実行された場合はそれぞれの文字列を連結して完成
    .reduce(
      "",
      (str, number) -> str + number,
      (first, second) -> first + second
    )
    .toString();
  System.out.println(numbers);
  // 0123456789
}
```

```java
public static void main(String[] args) {
  String numbers = Stream.iterate(0, number -> number + 1)
    .limit(10)
    // 初期値は空のStringBuilder
    // 各要素をそのStringBuilderに追加していき
    // 並列実行された場合はそれぞれのStringBuilderを結合して完成
    .collect(
      StringBuilder::new,
      (stringBuilder, number) -> stringBuilder.append(number), // StringBuilder::appendとも書ける
      (first, second) -> first.append(second) // ここもStringBuilder::appendと書ける
    )
    .toString();
  System.out.println(numbers);
  // 0123456789
}
```

### 重要度 中

- `Stream#max(comparator)`
- `Stream#min(comparator)`

引数で与えられた `comparator` によってstreamの各要素を比較した上で、 `max` は最大値を `min` は最小値を持つOptionalを返します。

streamが空の場合、空のOptionalが返ります。

```java
public static void main(String[] args) {
  var languages = List.of("Java", "C#", "Kotlin", "Swift");

  languages.stream().max(Comparator.comparingInt(String::length));
  // Optional Kotlin

  languages.stream().min(Comparator.comparingInt(String::length));
  // Optional C#
}
```

- `IntStream#max()`
- `IntStream#min()`
- `IntStream#average()`
- `IntStream#sum()`

IntStreamの要素の要素における、 `max` は最大値・ `min` は最小値・ `average` は平均値を持つOptionalを、 `sum` は合計値を返します。

streamが空の場合、Optionalを返すものは空のOptionalが返り、 `sum` は0が返ります。

これら集計が必要となる場面はあると思います。

- `IntStream#summaryStatistics()`

要素数・最大値・最小値・平均値・合計値を持つ統計情報インスタンスを返します。

上記の `max` ・`min` 等の集計値を複数種類必要とする場合に役立ちます。

- `Stream#forEach(action)`
- `Stream#forEachOrdered(action)`

streamの各要素に引数で与えられた `action` を適用します。

`forEach` は実行される要素の順序が保証されず、 `forEachOrdered` は実行される要素の順序が保証されます。

入出力などの副作用を生じさせるメソッドを各要素で実行したい時に利用します。

### 重要度 低

- `Stream#toList()`

streamを不変リストに変換します。

メソッド名から不変であることが読み取れないこともあり、重要度が低いというよりは使って欲しくないメソッドです。

記述は長くなってしまいますが、可変リストなら `collect(Collectors.toCollection(ArrayList::new))` を、不変リストなら `collect(Collectors.toUnmodifiableList())` を使うことをお勧めします。

- `Stream#toArray()`
- `Stream#toArray(generator)`

streamを配列に変換します。

引数無しは `Object[]` が返ります。

引数有りには主に `stream<T>` における `T[]` のコンストラクタ参照を渡し、 その型の配列が返ります。

結果を `List<T>` で受け取る方法がある以上、古いメソッドから配列を要求されている場合くらいしか使い道がありません。

```java
public static void main(String[] args) {
  var languages = List.of("Java", "C#", "Kotlin", "Swift");
  String[] array = languages.stream().toArray(String[]::new);
}
```

- `Stream#findAny()`
- `Stream#findFirst()`

`findAny` はstreamのいずれかの要素を、 `findFirst` はstreamの最初の要素を持つOptionalを返します。

streamが空の場合、空のOptionalが返ります。

「どれか一つだけ」や「先頭の一つだけ」を欲しいことが少ないですし、sortedと組み合わせるならmin・maxで十分なのであまり使われません。

streamにはisEmptyがないのでそのかわりにはなります。（countを使うと要素数を全て数え上げるため非効率


## 最後に

filter・map等のコレクション操作は、現在主要言語にはほぼ間違いなく備わっています。

この解説にはJava特有の説明も含まれていますが、Javaに限らずどの言語にも通用する部分も多いので、是非マスターしてください。
