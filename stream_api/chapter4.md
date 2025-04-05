# Java Stream API 特講 第四章 ~ Stream API 概観・生成操作

## Stream APIとは

Stream APIとは、コレクション等に対する操作を宣言的に記述できるAPI群を言います。

従来、コレクションはIteratorによって全要素に対して反復することは出来ましたが、その要素に対して何をするかを手続的に書く必要がありました。

これに対しStream APIはコレクションに対する操作をメソッドとして提供することによって、何をしたいのかをコード上に記述できるようになりました。

```java
// 従来
// 変換もフィルタリングも判定も全て拡張forで行うので、中身の処理を見ないと何をしているのかがわからない
public static List<String> toStringAll(List<Integer> list) {
  ArrayList<String> result = new ArrayList<>();
  for (Integer element : list) {
    result.add(Integer.toString(element));
  }
  return result;
}

// Stream API
// 変換はmap、フィルタリングはfilterと各処理がメソッドに分かれているので、何をしているのかがわかりやすい
public static List<String> toStringAll(List<Integer> list) {
  return list.stream()
    .map(element -> Integer.toString(element))
    .toList();
}
```

## Stream APIの特徴

現段階ではイメージできない点もあると思います。

中間操作・終端操作の章の読了後に読み返すと良いでしょう。

### 生成・中間操作・終端操作

Stream APIは生成・中間操作・終端操作の三段階で構成されます。

生成はコレクションをStreamインスタンスに変換したり、Streamインスタンスを生成するstaticメソッドを呼び出したりすることで、操作の起点となります。

中間操作はStreamに対する操作で、0回以上の任意の回数行うことができます。

終端操作はStreamを集計したり変換したりすることで結果を得ます。

例えば、

1. 文字列のlistをstreamに変換し（生成）
1. Jから始まるものだけを抜き出し（中間操作）
1. 重複するものを取り除き（中間操作）
1. 文字数に変換し（中間操作）
1. 累計を取る（終端操作）

のような形になります。

```java
public static void main(String[] args) {
  // 現段階では理解できなくとも、メソッド名から何をしているかの雰囲気は伝わるはず
  int result = List.of("Java", "C++", "Python", "Java", "Ruby", "Javascript", "Haskell", "Java")
    .stream()
    .filter(language -> language.startsWith("J"))
    .distinct()
    .mapToInt(String::length)
    .sum();
  System.out.println(result);
}
```

### 遅延実行

Stream APIの中間操作メソッドは、その操作をすぐには行わず終端操作のメソッドが呼び出された際に必要な分だけ行います。

```java
public static void main(String[] args) {
  // 終端操作前に一旦変数に取る
  Stream<String> stream = List.of("Java", "C++", "Python", "Java", "Ruby", "Javascript", "Haskell", "Java")
    .stream()
    .filter(language -> {
        System.out.println(language);
        return language.length() == 4;
    });

  // lambdaのprintよりもこちらが先に実行される
  System.out.println("Languages!");

  // 終端操作
  long result = stream.count();
  System.out.println(result);

  /*
  * Languages!
  * Java
  * C++
  * Python
  * ・・・
  * 4
  */
}
```

よって、中間操作メソッドに渡したlambda式が必ずしも実行されるとは限りません。

```java
public static void main(String[] args) {
  long result = List.of("Java", "C++", "Python", "Java", "Ruby", "Javascript", "Haskell", "Java")
    .stream()
    .map(language -> {
      // countを取るだけなら変換処理は無視できるので、このlambdaは実行されない
      System.out.println(language);
      return language + "!!";
    })
    .map(language -> {
      // 同上
      System.out.println(language);
      return "[" + language + "]";
    })
    .count();
    // countの前にdistinctやfilterを挟むと、変換の有無によってcountが変わる可能性が出るのでmapのlambdaが実行されるようになる
  System.out.println(result);
}
```

### メソッドチェーン

Stream APIの中間操作メソッドは、その操作を受け付けたStreamを返します。

そのため、中間操作を連続して呼び出すことができます。（既に上記のサンプルコードがメソッドチェーンになっています

### 並列処理

Stream APIは操作を並列で処理することができます。

CollectionをStreamに変換する時に `parallelStream()` を呼ぶか、Streamの持つ `parallel()` を呼ぶことで並列処理用のStreamを得られます。

**ただし、並列化することでのパフォーマンスは使用できるスレッド数などの環境と実行する処理内容に大きく依存します。**

**手元で試すと、処理にもよりますが並列化することで大体2~10倍程度遅くなります**

**これは並列化のためにStreamを分割し、並列処理後に結合するという処理が増えるためです**

**よほど並列化の恩恵が大きい環境と処理でない限り、パフォーマンスが悪化するので注意してください**

### 非破壊的

コレクションからStreamを生成した場合、Stream APIの各操作は元のコレクションの内容を変更せずに新たな結果を返します。

これは、関数を適用して返り値として結果を得るという関数型的な思想に合致します。

```java
public static void main(String[] args) {
  List<String> original = List.of("Java", "C++", "Python", "Java", "Ruby", "Javascript", "Haskell", "Java");
  List<String> filtered = original
    .stream()
    .filter(language -> language.length() == 4)
    .toList();
  System.out.println(original.size()); // 8
  System.out.println(filtered.size()); // 4
}
```

### プリミティブ型用の特化型

Streamには、オブジェクトを対象とする `Stream<T>` の他に、プリミティブ型を対象とする特化型の `IntStream` ・ `LongStream` ・ `DoubleStream` があります。

他のプリミティブ型については、クラス数を増やしすぎないため、booleanは `Stream<Boolean>` を用いること、byte・char・shortは `IntStream` で、floatは `DoubleStream` で代用することとされています。

これらの違いとして、プリミティブ特化型の方がラッパークラスのStreamよりメモリ上のパフォーマンスが良い点と、オブジェクトを対象とする `Stream<T>` に対し、プリミティブ特化のStreamは算術演算子を使えることが保証されているため、累計を計算するsumや平均を計算するaverageなどのメソッドが用意されている点が挙げられます。

なお、 `Stream<T>` にも集計メソッドはありますが、算術演算子が使用できないためどうやって計算するかをlambda式で渡してやる必要があります。


## Streamの生成

（重要度は私の独断と偏見に基づくものです。）

### 重要度 高

- `Collection#stream()`

コレクションをStreamに変換します、Stream生成の基本です。

- `Arrays.stream(array)`

配列をStreamに変換します。

これはオーバーロードされており、プリミティブ型の配列を渡すとプリミティブ特化型のStreamを返します。

（発展問題）
`List<Integer>` の `Collection#stream()` メソッドを呼ぶと `Stream<Integer>` が返りますが、 `IntStream` を返すことができないのはなぜでしょうか？（なお、List interfaceはCollection interfaceを継承しています

### 重要度 中

- `IntStream.range(start, end)`
- `IntStream.rangeClosed(start, end)`

`range` ・ `rangeClosed` はstartからendまでのint値を持つStreamを生成します。

`range` はendの値を含まず、 `rangeClosed` はendの値を含む点が異なります。

for文を置き換えられる点が有用です。

```java
public static void main(String[] args) {
  // 古代の文法
  // for (int i = 0; i < 3; i++) {
  //   System.out.println(i);
  // }

  // 変数が要らないワンライナー、スマート
  IntStream.range(0, 3).forEach(System.out::println);
}
```

- `Stream.generate(supplier)`
- `Stream.iterate(seed, operator)`

`generate` はsupplierが生成する値を返し続ける無限Streamを生成します。

`iterate` は `seed` を初期値に、重畳的にoperatorを適用した値を返し続ける無限Streamを生成します。

無限Streamは主に `limit` と組み合わせて使います。

```java
public static void main(String[] args) {
  Stream.generate(() -> "hoge")
    .limit(3)
    .forEach(System.out::println);
  /*
  * hoge
  * hoge
  * hoge
  */

  Stream.iterate("hoge", str -> str + "!")
    .limit(3)
    .forEach(System.out::println);
  /*
  * hoge
  * hoge!
  * hoge!!
  */
}
```

### 重要度 低

- `Stream.empty()`
- `Stream.of(elements)`
- `Stream.builder()`

`empty` は値を持たないStreamを、 `of` は可変長引数で渡された値を、 `builder` は返されたbuilderにaddされた値を要素に持つStreamを生成します。

有効な場面はオブジェクトを `flatMap` するケース（次章参照）くらいでしょうか。
