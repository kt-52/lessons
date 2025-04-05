# Java Stream API 特講 第五章 ~ Stream API 中間操作

プリミティブ特化型StreamについてはInt・Double・Longの3つがありますが、内容が重複するためここでは `IntStream` のみを例に挙げます。


## 中間操作

### 重要度 高

- `Stream#limit(n)`
- `Stream#skip(n)`

`limit` は前からn個以降の要素を捨て、 `skip` は前からn個の要素を捨てた残りのStreamを返します。

特に無限Streamでは大抵 `limit` で上限を定める必要があるので、よく使います。

- `Stream#filter(predicate)`

Streamの各要素にpredicateを適用し、falseが返った要素を捨てた残りのStreamを返します。

mapと並んで最もよく使うメソッドです。

- `Stream#map(mapper)`
- `Stream#mapToInt(mapper)`
- `IntStream#mapToObj(mapper)`

Streamの各要素にmapper関数を適用し、その変換結果のStreamを返します。

プリミティブ型と参照型というJava特有の事情により、 `map` ・ `mapToInt` 等多数のmapメソッドがありますがやっていること自体は同じです。

filterと並んで最もよく使うメソッドです。

- `Stream#distinct()`

Streamの重複する要素を捨てた残りのStreamを返します。

重複は `Object#equals(obj)` で判定されるとドキュメントにありますが、nullが重複する場合も `NullPointerException` が投げられずに重複したnullが捨てられます。

実際には `Objects.equals(a, b)` で判定しているものと思われます。

### 重要度 中

- `Stream#takeWhile(predicate)`
- `Stream#dropWhile(predicate)`

`takeWhile` は各要素に対してfalseが返るまで順にpredicateを適用していき、trueの返った要素のみを含むStreamを返します。

`dropWhile` は各要素に対してfalseが返るまで順にpredicateを適用していき、trueの返った要素を捨てた残りのStreamを返します。

内容的には `limit` ・ `skip` に判定を渡せるバージョンなので、 `limitWhile` ・ `skipWhile` という名前の方が良かったように思います。

```java
public static void main(String[] args) {
  List.of(0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
    .stream()
    .dropWhile(number -> number < 8)
    .forEach(System.out::println);
  /*
  * 8
  * 9
  * 10
  */

  List.of(0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
    .stream()
    .takeWhile(number -> number < 3)
    .forEach(System.out::println);
  /*
  * 0
  * 1
  * 2
  */
}
```

- `Stream#flatMap(mapper)`
- `Stream#flatMapToInt(mapper)`

Streamの各要素にmapper関数を適用し、その変換結果を平坦化したStreamを返します。

平坦化とは、入れ子になったコレクションの次元を下げることを言います。（例えば2次元配列→1次元配列

インスタンスフィールドでコレクションを持つオブジェクトのコレクションがあって、そのフィールド内容を集計したいような場合に利用されます。

```java
public class Main {
  public static void main(String[] args) {
    var team = new ArrayList<User>() {{
      this.add(new User("Java"));
      this.add(new User("Swift", "Kotlin"));
      this.add(new User("Java", "Javascript"));
      this.add(new User("Javascript", "Python"));
      this.add(new User("C#", "Java"));
    }};

    var teamSkills = team.stream()
      .flatMap(user -> user.skills().stream()) // 各要素をStreamに変換する、これらのStreamが全て連結されたものがflatMapの返り値となる
      .distinct()
      .toList();

    System.out.println(teamSkills); // [Java, Swift, Kotlin, Javascript, Python, C#]
  }
}

class User {
  private final List<String> skills = new ArrayList<>();

  public User(String... skills) {
    this.skills.addAll(Arrays.asList(skills));
  }

  public List<String> skills() {
    return List.copyOf(this.skills);
  }
}
```

なお、プリミティブ特化型Streamに `flatMapToObj` はなぜかありません。

これをやりたい場合、後述の `boxed` を挟む必要があります。

- `IntStream#sorted()`
- `Stream#sorted(comparator)`

Streamの要素を並び替えたStreamを返します。

Stream APIの追加と共に、Comparatorにも多数のstatic・defaultメソッドが追加されています。

もはや `compare(T o1, T o2) { return o1 - o2; }` のような謎の呪文を書く必要は無くなりました。

```java
public static void main(String[] args) {
  var languages = List.of("Java", "C#", "Python", "Kotlin", "Swift", "Elixir");

  // 自然順序、TがComparable<T>を実装していることが条件でComparable#compareToでソートされる
  // Stringクラスの自然順序は辞書順
  languages.stream()
    .sorted(Comparator.naturalOrder())
    .forEach(System.out::println);
  // [C#, Elixir, Java, Kotlin, Python, Swift]

  // 上の逆順
  languages.stream()
    .sorted(Comparator.reverseOrder())
    .forEach(System.out::println);
  // [Swift, Python, Kotlin, Java, Elixir, C#]

  // int値でソート、ここでは文字数順
  languages.stream()
    .sorted(Comparator.comparingInt(String::length))
    .forEach(System.out::println);
  // [C#, Java, Swift, Python, Kotlin, Elixir]

  // 上の逆順
  languages.stream()
    .sorted(Comparator.comparingInt(String::length).reversed())
    .forEach(System.out::println);
  // [Python, Kotlin, Elixir, Swift, Java, C#]

  // 文字数の逆順で、同一順序内は辞書順
  languages.stream()
    .sorted(Comparator.comparingInt(String::length).reversed().thenComparing(Comparator.naturalOrder()))
    .forEach(System.out::println);
  // [Elixir, Kotlin, Python, Swift, Java, C#]
}
```

なお、プリミティブ特化型Streamに引数有りの `sorted(comparator)` はなぜかありません。

これをやりたい場合、後述の `boxed` を挟む必要があります。

**また、引数無しの `Stream#sorted()` はTの型によってClassCastExceptionを投げるので使ってはいけません。**

その代わりに、型安全な同義の呼び出しとして `sorted(Comparator.naturalOrder())` を使用します。（ `Stream#sorted()` が実行時例外を投げる状況でコンパイルエラーとなります。

- `IntStream#boxed()`

各要素をボクシングしたStream（ `IntStream` → `Stream<Integer>` ）を返します、これは `mapToObj(Integer::valueOf)` と同義です。

前述の `flatMapToObj` や `sorted(comparator)` をプリミティブ特化型Streamで呼びたい場合に仕方なく挟みます。

### 重要度 低

- `Stream#peek(action)`

Streamの各要素にactionを適用した上で、そのままStreamを返します。

完成したコード上に現れることは少ないので重要度は低としましたが、開発やデバッグにはよく使います。

特に、Streamは遅延実行される特徴からデバッガで追いにくいという欠点があるため、デバッガを使用していてもprintlnで表示したい場面は多いです。

- `Stream#mapMulti(mapper)`
- `Stream#mapMultiToInt(mapper)`

`Stream#flatMap` の変形型です。

できること自体は同じですが、flatMapは複数の返り値をStreamで返すのに対し、mapMultiは返したい要素をconsumerに渡します。

返す各要素が少ない場合はmapMultiの方がstream化しない分パフォーマンス向上が見込めるらしいですが、手元ではflatMapの方が早かったので環境次第ではありますがflatMapで良いと思います。（そこまで厳密にパフォーマンスが必要なら、C++でfor文書けばいいんじゃないですかね・・・

また、 `Consumer#accept` を複数回呼ぼうとするとワンライナーにならないので、lambda式の `{}` とreturnが省略できなくなるというのもStream APIの趣旨にそぐわないです。

```java
// flatMapのサンプルコードの抜粋
var teamSkills = team.stream()
  // .flatMap(user -> user.skills().stream())
  .mapMulti((user, consumer) -> user.skills().forEach(consumer))
  .distinct()
  .toList();
```

- `IntStream#asDoubleStream()`
- `IntStream#asLongStream()`

プリミティブ特化型Streamを変換します。

あまり使うことはないでしょう。
