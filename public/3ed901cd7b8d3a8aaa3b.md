---
title: テストコード素人がJUnitを1か月触って学んだこと
tags:
  - JUnit
  - SpringBoot
private: false
updated_at: '2023-01-03T11:51:02+09:00'
id: 3ed901cd7b8d3a8aaa3b
organization_url_name: null
slide: false
ignorePublish: false
---
先日、JUnitを1月程触る機会があり、色々と勉強したので忘れないように備忘録を残しておくことにした。
JUnit歴1か月弱なので、おかしなことを書いているかもしれない。おかしなところを見つけた場合はスルーするかそっと優しくご指摘ください。

# 環境
JDK1.8
Spring-boot 2.4.2
JUnit 5

# テストコードの書き方
## ビジネスロジックのテスト
まずはソースコードをドン！


```テスト対象.java
    public class Calc {
        public int add(int num1, int num2) {
            return num1 + num2;
        }
    }
```


```テストコード.java
import static org.hamcrest.CoreMatchers.is;
import static org.junit.Assert.assertThat;

import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;

public class CalcTest {
    Calc c = new Calc();

    @Test
    @DisplayName("add test")
    public void addTest() {
        // 実際の値
        int actual = c.add(1, 1);
        // 期待値
        int expected = 2;

        assertThat(actual, is(expected));
    }
}
```

足し算を行うCalc#addと、そのテストコードCalcTest#addTest。

### 命名
色々な宗派があると思うが、`テスト対象のメソッド名＋Test` が多い気がする。

### @Testアノテーション
このメソッドはテストコードですよ、の印。
お作法的に書いてしまってるのが本音なので、もう少し深堀して理解したいところ。

### @DisplayNameアノテーション
Eclipse等でJUnit実行したときに、ここで指定した名前がテスト結果に表示される。
省略可能で、省略時はメソッド名がそのまま表示される。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/159675/f3926917-b1c6-9971-1c28-00a37ef5aa86.png)

### assertThat

`assertThat(T actual, Matcher<? super T> matcher)` でテストを実行する。
actualが実際値で、matherが期待値。ここが一致すればテスト成功、不一致ならテスト失敗。

assertThatのほかに、assertEquals, assertTrue, assertFalseなど様々あるが、個人的にはassertThatが一番使いやすく感じた。色々調べた中での個人的assertThatのメリットとしては

- assert that actual is expected と英文として自然な形でコーディングできるため、可読性が高い
- 第二引数 `Matcher<? super T> matcher` で使えるメソッドが豊富 → https://qiita.com/opengl-8080/items/e57dab6e1fa5940850a3

です。
参考：https://qiita.com/YNYS2020/items/8090048d3f540455b699

## Controllerのテスト
SpringのControllerのテストはassertThatではなく、MockMvcを使う。

```テスト対象.java
@Controller
public class HogeController {
    @RequestMapping(value = "/top")
    public String top() {
        return "top";
    }
}
```

```テストコード.java
@SpringBootTest(classes = HogeController.class)
public class HogeControllerTest {
    /** MockMvc */
    private MockMvc mockMvc;

    /** WebApplicationContext */
    @Autowired
    private WebApplicationContext webApplicationContext;

    /**
     * 前処理
     */
    @BeforeEach
    public void  setup() {
        mockMvc = MockMvcBuilders.webAppContextSetup(this.webApplicationContext).build();
    }

    @Test
    public void topTest() throws Exception {
        mockMvc.perform(get("/top"))
            .andExpect(status().isOk())
            .andExpect(view().name("top"));
    }
}
```

`mockMvc.perform` の後ろにテストしたい条件をつらつらと記載していく寸法。
わりと直感的にできてわかりやすい。
`get("/top")`の後ろにもいろいろメソッドチェーンで条件を付与できる（Cookieやリクエストパラメタ/ヘッダなど）

## Formバリデーションのテスト
以下がとても参考になった。
これ以上の説明を書くことは私には無理なので、リンクを引用しておく。
https://qiita.com/akane_kato/items/8bfc6fa9bf1f7518e515

## staticメソッドのmock化
staticメソッドのmock化は少し工夫が必要です。

```staticのmock化.java
        try (final var localDateTime = Mockito.mockStatic(LocalDateTime.class)) {
                localDateTime.when(LocalDateTime::now).thenReturn(LocalDateTime.of(1970, 1, 1, 0, 0, 0)));
            // write test code ...
        }
```

例えば`LocalDateTime.now()`の値を固定したい場合、上記のように記述する。
`Mockito.mockStatic`で対象メソッドのクラスをモック化し、try-with-resourceで宣言する。


引数を持つstaticメソッドの場合は以下のようにメソッド参照を使う。

```staticメソッドのmock化.java
    @MockBean
    private WebApplicationContext mockWac;

    @Test
    public void test() throws Exception {
        try (var webapplicationContextUtilsMock = Mockito.mockStatic(WebApplicationContextUtils.class)) {
            webapplicationContextUtilsMock.when(() -> WebApplicationContextUtils.getWebApplicationContext(Mockito.any())).thenReturn(mockWac);
            doReturn(new Hoge()).when(mockWac).getBean("hoge");
        }
        // write test code ...
    }
```
