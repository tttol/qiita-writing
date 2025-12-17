---
title: テストコードにおけるヘルパークラスの立ち位置
tags:
  - テストコード
private: false
updated_at: ''
id: null
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
public class UserAssertions {
    
    public static void assertUserEquals(User expected, User actual) {
        assertNotNull(actual);
        assertEquals(expected.getId(), actual.getId());
        assertEquals(expected.getUsername(), actual.getUsername());
        assertEquals(expected.getEmail(), actual.getEmail());
    }
    
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

actualとexpectedがわかりづらくなる要因とは何か？

# ヘルパークラスの神クラス化
何でもかんでもヘルパーに入れない。必要最低限のものに限定し、クラス名で役割を絞る。TestHelperみたいな汎用的な名前にしてしまうと他の開発者がヘルパーメソッドを全てそこに実装してしまう恐れがある。

# ヘルパークラスに含めるもの・含めないもの
- 含める
    - 全テスト共通で利用するロジック
        - 認証情報のセット
        - モックのセットあっぷ
- 含めない
    - 一つのテストクラスでのみ利用するリクエスト・レスポンス生成処理
