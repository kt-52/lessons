# 演習課題 問題編

## 問題

以下の通りのmainメソッドを持つJavaコンソールアプリケーションがあります。
このアプリケーションを実行すると以下の実行結果が出力される `ReverseList` クラスを実装してください。

* Javaコンソールアプリケーション

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

* 実行結果

```
5
4
3
2
1
```
