---
title: java-static
tags:
  - ''
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---
# staticブロックとは
Javaにおいて`static{ }`で表されるブロックのことを指します。具体例は以下。
```java
  static {
      value = 100;
  }
```

staticブロックはJVMにクラスがロードされる前に実行されるという特徴があります。つまり、クラスのコンストラクタやインスタンスメソッドよりも前に実行されることになります。


また、staticブロックは一度しか実行されない仕様となっています。同じクラスを複数回宣言しても、staticブロックは最初の1回目にだけ実行され、2回目以降は実行されないようにJVMが制御しています。

```java
public class StaticBlockSample {
    public static void main(String[] args) {
        var s = new Static();
    }

    static class Static {
        static {
            System.out.println("1. Static block");
        }

        static int value = initValue();

        static int initValue() {
            System.out.println("2. Static field initialization");
            return 10;
        }

        {
            System.out.println("3. Instance initializer");
        }

        public Static() {
            System.out.println("4. Constructor");
        }
    }

}
  ```

```bash
❯ java StaticBlockSample
1. Static block
2. Static field initialization
3. Instance initializer
4. Constructor
```

# staticブロックの使い所の例
staticオブジェクトの初期化
```java
  static {
      settings = new HashMap<>();
      settings.put("host", "localhost");
      settings.put("port", "8080");
      settings.put("timeout", "30");
  }
```

DBなどのリソースの初期化
```java
      static Properties dbProperties;

      static {
          dbProperties = new Properties();
          try (InputStream input = DatabaseConfig.class
                  .getClassLoader()
                  .getResourceAsStream("db.properties")) {
              dbProperties.load(input);
          } catch (IOException e) {
              throw new ExceptionInInitializerError(e);
          }
      }
```


# 区別が必要なものたち
## static変数
```java

static int value = initValue();

static int initValue() {
    System.out.println("2. Static field initialization");
    return 10;
}
```
上記のvalueのようなstatic変数は、書かれた順に上から初期化されます。
例えば下記のようにstaticブロックより上に記載すると、`2. Static field initialization`が先に出力されます。

```java
public class StaticBlockSample {
    public static void main(String[] args) {
        var s = new Static();
    }

    static class Static {
        static int value = initValue();

        static int initValue() {
            System.out.println("2. Static field initialization");
            return 10;
        }

        static {
            System.out.println("1. Static block");
        }


        {
            System.out.println("3. Instance initializer");
        }

        public Static() {
            System.out.println("4. Constructor");
        }
    }

}
```

```bash
❯ java StaticBlockSample
2. Static field initialization
1. Static block
3. Instance initializer
4. Constructor
```

## インスタンスブロック
```java
{
    System.out.println("3. Instance initializer");
}

```
インスタンスブロックは、コンストラクタより先に実行されます。
複数のコンストラクタの間で共通の前処理がある際などに有用そうですが、正直実務ではあまり見かけないかもしれません。


