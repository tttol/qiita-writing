---
title: MySQL慣れしたエンジニアがOracle DBを初めて触った時に戸惑ったこと
tags:
  - 'mysql'
  - 'oracle'
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---
私はエンジニアになって7年ほど経ちますが、今までOracle DBというものに触れる機会がなく、RDBMSといえば大体MySQLでした。（たまにポスグレも）そんな私が最近ついにOracle DBを利用する現場に配属になり、満を持してOracle DBデビューを果たしました。Oracle DBはMySQLやにはない独特の概念・仕様が多く、MySQLと同じ感覚でいると痛い目をみることがあります。本番環境でやらかしてしまう前に一度調べてみましょう。

# スキーマ
Oracle DBにはスキーマという概念があります。（MySQLにもスキーマはありますが）スキーマとはデータオブジェクトの論理的な集合を指します。テーブルは全て任意のスキーマに属することになります。

例えばHOGE_SCHEMAというスキーマのなかにUSERSというテーブルがあった場合、SELECT文はこうなります。
```sql
SELECT * FROM HOGE_SCHEMA.USERS;
```

`スキーマ.テーブル`のフォーマットでテーブルを指定します。スキーマが一つしかない場合はスキーマ部分を省略してもOKです。複数のスキーマを持つ環境では明示的にスキーマを指定すべきシーンが発生します。

### スキーマはユーザーごとに作成される
Oracle DBではユーザーを作成すると、ユーザー名と同名のスキーマが自動的に作成されます。ユーザーとスキーマは一対一の関係で対応しています。
```sql
-- ユーザー作成 = スキーマ作成
CREATE USER sales_user IDENTIFIED BY password;

-- このユーザーがテーブルを作成すると、sales_userスキーマに属する
CREATE TABLE sales_user.customers (...);
```

### スキーマを跨いでテーブルをJOINしたい時
スキーマAにいる状態からスキーマBにあるテーブルにアクセスしたい場合、シノニムを使うことでスムーズにアクセスが可能になります。Oracle DBにおけるシノニム(synonym)とは別名・類義語という意味で、テーブルやビューなどに別名をつけることを指します。

例えばSALES_USERスキーマからPRODUCT_USERスキーマのテーブルにアクセスする例は以下の通りです。
```sql
-- SALES_USERスキーマで実行
-- PRODUCT_USERスキーマのproductsテーブルへのシノニムを作成
CREATE SYNONYM products FOR product_user.products;

-- これで、あたかも自分のスキーマにあるかのようにアクセス可能
SELECT 
    o.order_id,
    o.order_date,
    p.product_name,
    p.price
FROM 
    orders o
INNER JOIN 
    products p ON o.product_id = p.product_id;
```


なお、シノニムを使わずに直接参照することも可能です。ただし、これはスキーマ名に依存するので、環境ごとにスキーマ名が異なる場合（SALES_USER_[DEV|STG|PROD]など）やスキーマ名の変更が発生した際にSQLエラーになるリスクがあります。
```sql
-- SALES_USERスキーマにログイン中
-- PRODUCT_USERスキーマのテーブルを直接参照
SELECT 
    o.order_id,
    o.order_date,
    p.product_name,
    p.price
FROM 
    sales_user.orders o
INNER JOIN 
    product_user.products p ON o.product_id = p.product_id;
```

# 空文字はNULLとして扱われる
```sql
SELECT '' IS NULL FROM DUAL; -- TRUE
```

初めて聞いた時にまあまあ驚いたんですが、空文字とNULLは同じものとして扱われるようです。
知らなかったら絶対事故ってる自信があります。

ちなみにMySQLだとこうです。
```sql
SELECT '' IS NULL;  -- FALSE
SELECT '' = '';     -- TRUE
```

DBMSごとに挙動変えるのはやめていただきたいですね。

# 雑感
これら以外にも、Oracle DBにはLIMIT句がない・クォーテーションの扱いが異なる・文字列連結の方法が異なる、など色々あるようです。疲れたのでここまでにしますが、ちょっとずつキャッチアップしていこうと思います。
