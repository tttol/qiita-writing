---
title: 【String型 vs StringBuilder】文字列結合における処理速度の違い
tags:
  - Java
  - string
  - StringBuilder
private: false
updated_at: '2022-01-19T10:59:28+09:00'
id: 2455ee3a8bbbfb447abb
organization_url_name: null
slide: false
ignorePublish: false
---
#はじめに
Javaで文字列結合をする際、String型で`+=`で繋いでいくか、StringBuilderの`append()`でつなぐか、２通りの手段があります。


結果はどちらも同じですが、処理速度に差が出ます。

#Stringでやったとき
例えばこんなコード

```java
String stringResult = null;
    // Stringオブジェクトによる文字列結合
    for (int i = 0; i < 10; i++) {
      String str = Integer.toString(i);
      stringResult += str;
    }
```
String型とはStringオブジェクトのことなので、String型の変数が宣言される＝Stringオブジェクトが生まれる、ということになります。


なので、文字列結合をしている`stringResult += str;`のタイミングでStringオブジェクトを生成しています。


つまり、for文のループの回数分だけStringオブジェクトが生まれるわけです。


#StringBuilderでやったとき
例えばこんなコード

```java
StringBuilder sb = new StringBuilder();
    // StringBuilderによる文字列結合
    for (int j = 0; j < 10; j++) {
      sb.append(Integer.toString(j));
    }
```

結果はStringの例と同じです。


しかし、`append()`はStringオブジェクトを作りません。末尾に追加していくだけです。


Stringオブジェクトを生成しない分、処理が速い。

#どれくらい速度が違う？
下記のようなプログラムで、StringとStringBuilderとの処理時間の違いを測ってみました。

```java
public class StringBuilderSample{
  public static void main(String[] args) {
    final int MAX_COUNT = 100;
    System.out.println("結合回数：" + MAX_COUNT + "回のとき");
    /********************* String型の場合 *********************/
    String stringResult = null;

    // Stringでの処理にかかった時間
    long sTime;

    // 開始時間
    long startTime = System.nanoTime();

    // String型による文字列結合
    for (int i = 0; i < MAX_COUNT; i++) {
      String str = Integer.toString(i);
      stringResult += str;
    }

    // 終了時間
    long endTime = System.nanoTime();

    // 処理にかかった時間を算出
    sTime = endTime - startTime;

    System.out.println("Stringでの処理時間：" + sTime + "ナノ秒");

    /********************* StringBuilderの場合 *********************/
    StringBuilder sb = new StringBuilder();

    // StringBuilderでの処理にかかった時間
    long sbTime;

    // 開始時間
    startTime = System.nanoTime();

    // StringBuilderによる文字列結合
    for (int j = 0; j < MAX_COUNT; j++) {
      sb.append(Integer.toString(j));
    }

    // 終了時間
    endTime = System.nanoTime();

    // 処理にかかった時間を算出
    sbTime = endTime - startTime;

    System.out.println("StringBuilderでの処理時間：" + sbTime + "ナノ秒");
    System.out.println("StringBuilderの方が " + (sTime - sbTime) + " ナノ秒速い");
  }
}
```

文字列結合を`MAX_COUNT`の回数だけ行ったときの処理速度を表示するプログラムです。

以下、`MAX_COUNT`の値を100, 1000, 1万, 10万と増やしていきながら処理時間の違いを見ていきます。

ちなみに、ナノ秒とは10 ^ (-9) 秒（＝0.000000001秒）のことです。


##100回実行したとき
![スクリーンショット 2018-07-19 23.11.48.png](https://qiita-image-store.s3.amazonaws.com/0/159675/bbbec012-acb9-1b4c-f65a-f834e4ee68e7.png)


##1000回実行したとき
![スクリーンショット 2018-07-19 23.12.23.png](https://qiita-image-store.s3.amazonaws.com/0/159675/e804c83c-50ad-3e7b-4bd5-e5cc47a4dfbc.png)

##1万回実行したとき
![スクリーンショット 2018-07-19 23.13.53.png](https://qiita-image-store.s3.amazonaws.com/0/159675/45f1be05-6b27-102e-02d8-b1fbd74182d2.png)
約0.57秒速い

##10万回実行したとき
![スクリーンショット 2018-07-19 23.14.49.png](https://qiita-image-store.s3.amazonaws.com/0/159675/dcca2fa2-06ec-1b81-a5bf-40a2bda0802f.png)

約27秒速い

#StringBuilderのほうが速かった！
1000回ぐらいまでならせいぜいマイクロ秒（10 ^(-6)）単位での違いなので、そんなに気にはならない。


1万回超えたあたりから処理時間の差が気になるレベルになってくる。


まあ何回実行するにしてもStringBuilderの方が速いことは確かなので、特別な理由がない限り文字列結合はStringBuilderでやったほうがいいですね。

2018 07/21 追記：続編書きました → [【java】appendの中で"+"は使わない！](https://qiita.com/tor_2085/items/754180f91a8c1554287d)
