# 演習課題解説 その1

## 型の適合性

```Java
import java.util.List;

public class Main {
  public static void main(String[] args) {
    List<String> reverseList = new ReverseList<String>();
    reverseList.add("1");
    reverseList.add("2");
    reverseList.add("3");
    reverseList.add("4");
    reverseList.add("5");

    for (String element : reverseList) {
      System.out.println(element);
    }
  }
}
```

`ReverseList` を定義するにあたって、型の適合性を考える必要があります。

```Java
List<String> reverseList = new ReverseList<>();
```

まず、この行に注目しましょう。
`=` の右辺は、new演算子を使用しているので型が `ReverseList<String>` になります。
`=` の左辺は、 `List<String>` と変数宣言しているので変数 `reverseList` の型は `List<String>` になります。
代入演算子の左右の型が異なるため、 `ReverseList<String>` と `List<String>` に代入可能な関係性がないといけません。

Javaにおいては、親クラス型の変数に子クラス型のインスタンスを、またインターフェース型の変数に実装クラス型のインスタンスをそのまま（明示的なキャストが不要という意味）代入することができます。
これをアップキャストと言います。

本問においては、左辺の変数が `List<String>` 型です。
これはimportにより `java.util.List` であることがわかります、そしてこれはインターフェースです。（https://docs.oracle.com/javase/jp/8/docs/api/java/util/List.html
そうすると、右辺の `ReverseList<String>` は `java.util.List` の実装クラスでなくてはなりません。
また、 `<String>` と型引数を取っているので、ジェネリックなクラスである必要もあります。

`ReverseList` を `java.util.List` の実装クラスにする方法は2通りあります。

1. `implements java.util.List` する
1. `implements java.util.List` しているクラスを `extends` する

`implements` する場合は全ての抽象メソッドを実装しないとコンパイルエラーになるのが大変ですが、今回は `add` と `iterator` しか使わないので、あとは全て `return null;` とかの適当な実装でも構いません。
実装クラスを `extends` する場合は、これら抽象メソッドが（実装クラスがabstractでなければ）実装されているので楽です。

以上より、 `ReverseList` のクラス定義は以下のようになります。

* `implements java.util.List` しているクラスを `extends` する場合

例として `ArrayList` を使用していますが、他のList実装クラスでも可能です。

```Java
import java.util.ArrayList;

public class ReverseList<T> extends ArrayList<T> {

}
```

* `implements java.util.List` する場合

返り値は全て既定値で埋めています。

```Java
import java.util.List;

public class ReverseList<T> implements List<T> {
  @Override
  public int size() {
    return 0;
  }

  @Override
  public boolean isEmpty() {
    return false;
  }

  @Override
  public boolean contains(Object o) {
    return false;
  }

  @Override
  public Iterator<T> iterator() {
    return null;
  }

  @Override
  public Object[] toArray() {
    return null;
  }

  @Override
  public <T1> T1[] toArray(T1[] a) {
    return null;
  }

  @Override
  public boolean add(T t) {
    return false;
  }

  @Override
  public boolean remove(Object o) {
    return false;
  }

  @Override
  public boolean containsAll(Collection<?> c) {
    return false;
  }

  @Override
  public boolean addAll(Collection<? extends T> c) {
    return false;
  }

  @Override
  public boolean addAll(int index, Collection<? extends T> c) {
    return false;
  }

  @Override
  public boolean removeAll(Collection<?> c) {
    return false;
  }

  @Override
  public boolean retainAll(Collection<?> c) {
    return false;
  }

  @Override
  public void clear() {

  }

  @Override
  public T get(int index) {
    return null;
  }

  @Override
  public T set(int index, T element) {
    return null;
  }

  @Override
  public void add(int index, T element) {

  }

  @Override
  public T remove(int index) {
    return null;
  }

  @Override
  public int indexOf(Object o) {
    return 0;
  }

  @Override
  public int lastIndexOf(Object o) {
    return 0;
  }

  @Override
  public ListIterator<T> listIterator() {
    return null;
  }

  @Override
  public ListIterator<T> listIterator(int index) {
    return null;
  }

  @Override
  public List<T> subList(int fromIndex, int toIndex) {
    return null;
  }
}
```
