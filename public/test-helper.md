---
title: テストコードにおけるヘルパークラスの立ち位置
tags:
  - テストコード
private: false
updated_at: '2025-12-18T06:31:45+09:00'
id: 10a1bf4aabc56d5bb9dc
organization_url_name: null
slide: false
ignorePublish: false
---
テストコードがそこそこ実装されているコードベースでは、いわゆるヘルパークラスと呼ばれるクラスが存在することがあります。ヘルパークラスとはテストコード向けの便利ツール的なポジションであり、テストの前処理や複雑な期待値オブジェクトの生成、アサーション処理の共通化など、テストケース自体とは関係のない処理をヘルパークラスに書くことでテストコードの可読性を上げる働きがあります。

データ生成ヘルパーの例
```java
public class UserTestHelper {
    
    public static User createDefaultUser() {
        return User.builder()
                .id(1L)
                .username("testuser")
                .email("test@example.com")
                .password("password123")
                .createdAt(LocalDateTime.now())
                .build();
    }
}
```

アサーション共通化ヘルパーの例
```java
public class UserAssertionsHelper {
    public static void assertValidUser(User user) {
        assertNotNull(user);
        assertNotNull(user.getId());
        assertNotNull(user.getUsername());
        assertTrue(user.getEmail().contains("@"));
        assertNotNull(user.getCreatedAt());
    }
}
```


そんなヘルパークラスの立ち位置について考察します。

# ヘルパークラスが欲しくなるタイミング
ヘルパークラスが欲しくなるタイミングはズバリ「テストコードの可読性が下がってきた時」だと自分は考えます。

テストコード内の各テストケースで一番重要なところは実行結果と期待値であり、ソースコードレベルではよくactual, expectedという名前の変数で表現されることが多いと思います。
- actual: テスト対象のメソッドを実行した際の結果
- expected: テストケースの期待値

この2つの値を読むことでテストケースがどういったことを検証しているのかを知ることができます。逆に言えば、actualとexpectedの内容が読みづらい・見つけづらいような実装がされているとテストケースの検証内容も読み取りづらくなります。

つまり「テストコードの可読性が低い＝actual, expectedがわかりづらい」ということになりますね。

テスト対象のメソッドを実行するまでの前処理が多かったり、あるいは実行後のassertionが複雑だったりするとactual, expectedがテストコードの中で埋もれてしまいがちです。このような、テストそのものとは関係のない記述をヘルパークラスに移動することでactual, expectedが埋もれる現象を防ぎ、テストコードの可読性を上げることが可能です。

# ヘルパークラスの神クラス化
ヘルパークラスはいわゆる神クラスになってしまわないように注意する必要があります。

※神クラス＝一つのクラスに多種多様な処理が実装され結果的に巨大で複雑な構造になってしまったクラスのこと。多数のクラスとの依存関係があるため機能改修の際にバグやデグレを起こしやすく、リファクタリングを困難にすることが多い。

例えばTestHelper.javaのような汎用的なクラス名にしてしまうと、チーム内の全ての開発者にTestHelperクラスへあらゆるヘルパーメソッドを実装することを促してしまいます。冒頭の例で示したようにUserTestHelper, UserAssertionHelperなど目的をクラス名に含めると、各ヘルパーの役割がわかりやすくなります。

また、ヘルパーに処理を寄せすぎるとテストコードの可読性が逆に下がるケースがあります。例えば以下のようにactualとexpectedの処理を全てヘルパーに任せてしまうと、actualとexpectedの中身が何なのかがテストクラスを見ただけではわからなくなってしまいます。
```java
var actual = helper.runTest(testTarget);
var expected = helper.createResponse();
```

テストコードを読むだけで実行結果(actual)と期待値(expected)を即座に読み取ることができテスト対象のコードの仕様がわかる、というのがテストコードのあるべき姿です。以下の例のようにactualとexpectedの中身はテストクラスに記述すべきだと自分は考えます。

```java
// テストクラスに直接記述する
var actual = testTarget.getUser(10);
var expected = User.builder()
            .id(10)
            .name("Tom")
            .age(20)
            .build();

// or ヘルパーを用いるが中身は可視化する
var actual = helper.runGetUser(10);
var expected = helper.createResponse(10, "Tom", 20);
```

# ヘルパークラスに含めるもの・含めないもの
一概には言えませんが、ヘルパーに含めるもの・含めないものはざっくり以下の印象です。
- 含める
    - 全テスト共通で利用するロジック
        - 認証情報のセット
        - 毎回書くのが面倒に感じる複雑なassertion
- 含めない
    - 単一のテストクラスでのみ利用する前処理・後処理
    - 各テストケースにおけるactual, expectedの生成

ヘルパークラスを正しく実装することで、テストコードの可読性向上に貢献したいところです。
