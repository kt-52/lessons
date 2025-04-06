# 演習課題 回答編

## 回答例 その1

### コード

```Java
import java.util.ArrayList;

class ReverseList<T> extends ArrayList<T> {
  @Override
  public boolean add(T element) {
    this.add(0, element);
    return true;
  }
}
```

### 解説

`ArrayList#add(T)` はListの最後尾に要素を追加します。
`ReverseList<T>` ではそれをoverrideして先頭に追加するように変更します。
これにより、 `12345` の順にaddすると `54321` の順にListに追加されます。


## 回答例 その2

### コード

```Java
import java.util.ArrayList;
import java.util.Iterator;

public class ReverseList<T> extends ArrayList<T> {
  @Override
  public Iterator<T> iterator() {
    return new ReverseListIterator<T>(this);
  }
}

class ReverseListIterator<T> implements Iterator<T> {
  private final ReverseList<T> list;
  private int index;

  public ReverseListIterator(List<T> list) {
    this.list = list;
    this.index = list.size();
  }

  @Override
  public boolean hasNext() {
    return index > 0;
  }

  @Override
  public T next() {
    this.index--;
    return this.list.get(this.index);
  }
}
```

### 解説

`List#iterator()` は要素を順に取り出す方法を提供します。
`ReverseList<T>` ではそれをoverrideして要素を逆順で取り出すiteratorを返すように変更します。
ここでは `ReverseListIterator` を定義し、それをnewして返すようにしています。
`List#iterator()` をオーバーライドする方法の中では一番真っ当な方法です。
次の「回答 その3」のコード群はこれと同じ方針ですが、iteratorを自前で定義せずに既に定義されているiteratorを拝借しようという考え方になります。


## 回答例 その3

### コード 1

```Java
import java.util.ArrayList;

public class ReverseList<T> extends ArrayList<String> {
  @Override
  public Iterator<String> iterator() {
    return List.of("5", "4", "3", "2", "1").iterator();
  }
}
```

### コード 2

```Java
import java.util.ArrayList;

public class ReverseList<T> extends ArrayList<T> {
  @Override
  public Iterator<T> iterator() {
    return this.stream().sorted(Collections.reverseOrder()).iterator();
  }
}
```

### コード 3

```Java
import java.util.ArrayList;

public class ReverseList<T> extends ArrayList<T> {
  @Override
  public Iterator<T> iterator() {
    List<T> copied = new ArrayList<>(this);
    Collections.reverse(copied);
    return copied.iterator();
  }
}
```

### 解説

既存の `Iterable` が返す `Iterator` を流用して `ReverseList#iterator()` の返り値にしようというコードです。
この問題を通すだけなら `54321` の入ったListのiteratorを固定で返すコード1で十分です。

コード2はthisから逆順にソートしたstreamを作り、そのiteratorを返しています。
Stream APIの知識がないと何のことかわからないかもしれませんが、 `sorted reverseOrder` の単語から逆順ソートしていることが読み取れるかと思います。

コード3はthisをコピーし、それを逆順ソートしてiteratorを返しています。
`Collections#reverse(list)` は引数で与えられたlist逆順に変更します。
そのため、 `Collections.reverse(this)` とするとiteratorメソッド内でthisが逆順ソートされるのでこれを呼び出すたびに逆順・昇順・逆順・・・となってしまいます。

このように、引数で与えられたオブジェクトの内容を変更するメソッドを「破壊的なメソッド」と言います。
逆に引数で与えられたオブジェクトはそのままに、変更した内容の別のオブジェクトを作って返すメソッドを「非破壊的なメソッド」と言います。


## 回答例 番外編 その1

### コード

```Java
import java.util.ArrayList;

public class ReverseList<T> extends ArrayList<T> {
  public ReverseList() {
    System.out.println(5);
    System.out.println(4);
    System.out.println(3);
    System.out.println(2);
    System.out.println(1);
    System.exit(0);
  }
}
```

### 解説

`new ReverseList<String>()` した時点で `54321` を出力してアプリを終了します。
static初期化ブロックでも可能です。
また、アプリを終了せずとも `ReverseList#iterator()` を `hasNext` が常に `false` を返すイテレータオブジェクトを返すようにして拡張forループを回避する方法もあります。


## 回答例 番外編 その2

### コード

```Java
import java.util.ArrayList;

public class Main2 {
  public static void main(String[] args) {
    System.out.println("5");
    System.out.println("4");
    System.out.println("3");
    System.out.println("2");
    System.out.println("1");
  }
}

class ReverseList<T> extends ArrayList<T> {}
```

### 解説

Javaではmainメソッドを複数定義することができます。
`54321` を出力する別のmainメソッドを持つクラスを作り、そのクラスを指定して実行します。
`ReverseList` は使いませんが、定義が無いとコンパイルエラーになるので定義だけしてあります。
