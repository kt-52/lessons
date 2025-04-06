# 演習課題解説 その3

## カプセル化

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

前章では「listから逆順で要素を取り出す」方法について考えました。
他方、逆順でソートされたリストの実装方法としては「リストに要素を追加する際に逆順で追加する」方法も考えられます。

* そのまま追加し逆順で返す方法

```mermaid
flowchart TD

in1[1]
in2[2]
in3[3]
in4[4]
in5[5]
out1[5]
out2[4]
out3[3]
out4[2]
out5[1]
list[1 2 3 4 5]
iterator[reverse iterator]

in1 --"add to tail"--> list
in2 --"add to tail"--> list
in3 --"add to tail"--> list
in4 --"add to tail"--> list
in5 --"add to tail"--> list

list --> iterator

iterator --"get"--> out1
iterator --"get"--> out2
iterator --"get"--> out3
iterator --"get"--> out4
iterator --"get"--> out5
```

* 逆順で追加しそのまま返す方法

```mermaid
flowchart TD

in1[1]
in2[2]
in3[3]
in4[4]
in5[5]
out1[5]
out2[4]
out3[3]
out4[2]
out5[1]
list[5 4 3 2 1]
iterator[natural iterator]

in1 --"add to head"--> list
in2 --"add to head"--> list
in3 --"add to head"--> list
in4 --"add to head"--> list
in5 --"add to head"--> list

list --> iterator

iterator --"get"--> out1
iterator --"get"--> out2
iterator --"get"--> out3
iterator --"get"--> out4
iterator --"get"--> out5
```

これらは内部実装が異なりますが振る舞いは同じです。

## 情報隠蔽

外部仕様を公開し、内部実装を隠蔽する設計手法を情報隠蔽といい、カプセル化と並ぶもの・またはカプセル化の一部として捉えられています。
内部実装が隠蔽されることで、利用者にとっては必要な知識が減るという、実装者にとっては仕様を変えなければ内部実装を変更できるというメリットがあります。

---

構造や原理を知らないけれども使用方法を知っていれば使えるものには何があるでしょうか？

---
