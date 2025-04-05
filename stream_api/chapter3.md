# Java Stream API 特講 第三章 ~ lambda式・メソッド参照

前章の関数型インターフェースにより、Strategyパターンの手順の

1. interfaceを定義する
1. そのinterfaceを実装したclassを定義し、それに渡したいメソッドを実装する
1. そのclassをnewして引数に渡す

のうち「1. interfaceを定義する」が解決しました。

本章では残り2つを解決する言語仕様であるlambda式とメソッド参照について解説します。


## 型推論

lambda式とメソッド参照に入る前に、重要な前提知識として型推論を説明します。

型推論とは、コード上で型を明示せずとも文脈から型を推定してくれる機能を言います。

Javaは静的型付け言語なので、変数やメソッド引数・返り値の型は全てコンパイル時に定まります。

この時型をコード上で明示するのですが、分かりきっている型を何度も書かなくてはならない場合があり冗長です。

そのため、文脈から型が確定する場合にはコード上で明示しなくてもコンパイラが型を特定して補う機能が追加されています。

まずJava7で導入されたのがダイアモンド演算子です。

```java
// 同じとわかりきっているのにジェネリクスの型名を2回書かなくてはならなかった
List<Supercalifragilisticexpialidocious> list1 = new ArrayList<Supercalifragilisticexpialidocious>();

// 左辺からジェネリクスの型は明らかだから、右辺を省略していいのがダイアモンド演算子
List<Supercalifragilisticexpialidocious> list2 = new ArrayList<>();
```

さらにJava10でローカル変数の型推論が追加されました。

```java
// これも同じことを2回書かなくてはならなかった
Supercalifragilisticexpialidocious s1 = new Supercalifragilisticexpialidocious();

// 右辺の型から変数の型が定まるので、左辺は省略していいのがローカル変数の型推論
var s2 = new Supercalifragilisticexpialidocious();

// 型が合わずcompile error
s1 = "hoge";

// これも型が合わずcompile error
// s2は推論によりSupercalifragilisticexpialidocious型で宣言されている
s2 = "hoge";

// ただし、親クラスやインターフェースの型で宣言したい場合は明示的に書かなくてはならない
Object o = new Supercalifragilisticexpialidocious();
```

いずれも「型を明示しなくて良い」だけであって、「型が無くなる」わけではないことに注意してください。


## lambda式

プログラミングにおけるlambda式は匿名関数・無名関数とも呼ばれ、小さな関数を簡単に書く言語機能を言います。

Javaにおいては、「関数型インターフェースの抽象メソッドを実装したインスタンスを生成する」ことの省略記法となっています。

ここでは、関数型インターフェースの実装クラスを定義してそのクラスをnewすることとの対比で説明します。

Integerを受け取って、前後を `<>`で装飾した文字列を返すメソッドをlambda式を使わずに定義すると以下のようになります。

```java
class IntegerToStringConverterWithDecoration implements Function<Integer, String> {
  @Override
  public String apply(Integer number) {
    return "<" + Integer.toString(number) + ">";
  }
}

public class Main {
  public static void main(String[] args) {
    Function<Integer, String> converter = new IntegerToStringConverterWithDecoration();
    String result = converter.apply(20);

    System.out.println(result); // "<20>"
  }
}
```

これをlambda式を使うと以下のようになります。

```java
public class Main {
  public static void main(String[] args) {
    Function<Integer, String> converter = number -> "<" + Integer.toString(number) + ">";
    String result = converter.apply(20);

    System.out.println(result); // "<20>"
  }
}
```

省略された部分を解説します。

まず重要なのは、代入しようとしている変数の型が `Function<Integer, String>` であることから以下のことが推論できることです。

1. これは関数型インターフェースであるので、実装すべき抽象メソッドは一つだけである
1. これは `Function` なので、実装すべき抽象メソッドは `apply` である
1. この抽象メソッドは `Integer` 型の引数が1つで、返り値の型は `String` である

これにより、 `Integer` を受け取り `String` を返すメソッドの実装だけが記載されれば十分ということがわかります。

よって、`Function` 実装クラスはここまで記述を減らすことができます。

```java
// class名はなんでもいい
// implementsは推論により不要
class IntegerToStringConverterWithDecoration implements Function<Integer, String> {
  // アノテーションはそもそも不要
  @Override
  // public  -> アクセス修飾子はinterfaceの抽象メソッドなのでpublicで確定するため不要
  // String  -> 返り値の型は推論により不要
  // apply   -> メソッド名は推論により不要
  // Integer -> 引数の型は推論により不要
  public String apply(Integer number) {
    return "<" + Integer.toString(number) + ">";
  }
}

// よって最低限必要な記載は以下の通り
(number) {
  return "<" + Integer.toString(number) + ">";
}
```

これに、lambda式を生成する演算子である `->` （アロー演算子）を加えるとlambda式になります。（この時点でコンパイルが通ります

```java
// 引数とメソッド本体の間にアロー演算子を置くのがJavaにおけるlambda式の文法
(number) -> {
  return "<" + Integer.toString(number) + ">";
}
```

ここからさらに、lambda式専用の省略記法があります。

1. 引数が1つのみの場合、引数を囲む `()` を省略できる
1. 返り値を返すメソッド本体が `return` の1文のみの場合、 `{}` と `return` と `;` が省略できる
1. 返り値を返さないメソッド本体が1文のみの場合、 `{}` と `;` が省略できる

よって最終的に以下のようになります。

```java
// 引数の()と本体の{}・return・;を省略
number -> "<" + Integer.toString(number) + ">"
```

このように、lambda式は型推論から成り立っているため、推論の効かない場合lambda式はコンパイルエラーになります。

```java
public static void main(String[] args) {
  // 推論が効かないためNG
  Object obj = () -> 1;

  // これはOK
  // 推論により型が確定してインスタンス化された後は関数型インターフェース型以外にダウンキャストできる
  IntSupplier supplier = () -> 1;
  obj = supplier;
}
```

なお、変数に代入せずメソッドの引数に直接渡す場合はメソッド定義から推論されます。


## メソッド参照

lambda式を使っていると、「lambda式では特定のメソッドを呼ぶだけ」という場面がよくあります。

それを簡潔に書く記法がメソッド参照です。

* 【class名】::【staticメソッド名】

そのstaticメソッドを呼び出すlambda式と同等です。

```java
// 引数なし
Supplier<Long> supplier = System::nanoTime;
Supplier<Long> supplier = () -> System.nanoTime();

// 引数あり
UnaryOperator<Double> operator = Math::ceil;
UnaryOperator<Double> operator = number -> Math.ceil(number);
```

* 【class名】::【インスタンスメソッド名】

第一引数で与えられたインスタンスのインスタンスメソッドを呼び出すlambda式と同等です。

```java
// 引数なし
ToIntFunction<Random> generator = Random::nextInt;
ToIntFunction<Random> generator = random -> random.nextInt();

// 引数あり
BiFunction<LocalDateTime, DateTimeFormatter, String> function = LocalDateTime::format;
BiFunction<LocalDateTime, DateTimeFormatter, String> function = (localDateTime, formatter) -> localDateTime.format(formatter);
```

* 【インスタンス】::【インスタンスメソッド名】

そのインスタンスのインスタンスメソッドを呼び出すlambda式と同等です。

```java
// 引数なし
// 注）System.outはSystemクラスの持つPrintStream型のインスタンスフィールド
Consumer<Object> consumer = System.out::println;
Consumer<Object> consumer = obj -> System.out.println(obj);

// 引数あり
String message = "Hello World";
Predicate<String> predicate = message::startsWith;
Predicate<String> predicate = target -> message.startsWith(target);
```

* 【クラス名】::new

（コンストラクタは厳密にはメソッドではないのでこれはコンストラクタ参照と呼ばれますが、メソッド参照と機能としての違いはありません。）

そのクラスを `new` するlambda式と同等です。

```java
// 引数なし
Supplier<NullPointerException> supplier = NullPointerException::new;
Supplier<NullPointerException> supplier = () -> new NullPointerException();


// 引数あり
LongUnaryOperator operator = Long::new;
LongUnaryOperator operator = number -> new Long(number);
```
