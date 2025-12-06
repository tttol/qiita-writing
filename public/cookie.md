---
title: Cookieの属性の整理
tags:
  - 'cookie'
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに
ウェブアプリ開発者にとってCookieの理解は必須です。必須なんですが、いざ問われると忘れていたり、細かいところを分かってなかったりして、自分の理解がふわっとしているなあと思うことがよくあります。開発者としてよろしくないことなので、記事にすることで自分の知識を整理・強化することにします。

# そもそもCookieの役割
Cookieとは、ブラウザに保存されるデータのことです。

HTTP通信はステートレスな通信であるため、「状態」を保持できません。ですが、ウェブアプリを作っているとブラウザに状態を保持したいシーンがたくさんあります。ログイン中のユーザー情報だったり、ECサイトのカート内の商品データだったり、多岐に渡ります。こういった状態情報を保持するために利用されるのがCookieです。

Cookieはkey-value形式で保存されるテキストデータです。
```
sessionid=123456; theme=dark; ...
```

Cookieには様々な情報が保持されるので、万一Cookieの情報を悪意のある第三者に盗聴・奪取されるとセキュリティインシデントに繋がります。開発者はCookieの情報が外部に漏洩しないようにアプリケーションを設計・実装することが肝要です。そのために、Cookie側が属性という形で様々な防御機能を提供しています。代表的なものを以下に挙げていきます。

- HttpOnly
- Secure
- SameSite
- Expires
- Max-Age

例えば`HttpOnly`, `Secure`, `SameSite`が付与されたCookieは以下のようになります。
```
Set-Cookie: sessionid=abc123; HttpOnly; Secure; SameSite=Strict
```

# HttpOnly
HttpOnly属性が付与されたCookieはJavaScriptからCookieにアクセスできなくなります。例えば`document.cookie`プロパティからの利用が不可能になります。XSSなどでCookieを奪われるリスクを防ぐことが可能です。

例えばJavaのSpringBootでHttpOnlyを付与する場合はapplication.ymlで以下のように設定するか、
```yml:application.yml
# application.yml
server:
  servlet:
    session:
      cookie:
        http-only: true 
```

Java側で明示的にHTTPレスポンスヘッダに指定します。
```java
@GetMapping("/home")
public ResponseEntity<?> login(@RequestBody LoginRequest request, HttpServletResponse response) {
    String sessionId = generateSessionId();
    
    ResponseCookie cookie = ResponseCookie
        .from("sessionId", sessionId)
        .httpOnly(true)
        .secure(true)
        .path("/")
        .maxAge(Duration.ofHours(1))
        .sameSite("Strict")
        .build();
    
    response.addHeader("Set-Cookie", cookie.toString());
    
    return ResponseEntity.ok("Login successful");
}
```

# Secure
Secure属性が付与されたCookieはHTTPS通信でのみ送信が許可されます。つまりHTTP通信で送信することができなくなります。中間者攻撃の予防に繋がりますね。

# SameSite
Cookieを送受信する際にドメインに応じて制御をかけます。SameSiteは3つの値を持ち、デフォルトは`Lax`です。

- Strict
    - 同一ドメインからのリクエストのみ送信を許可
- Lax
    - 同一ドメインからのリクエスト かつ 以下の条件を満たすリクエストのみ許可
        - リンククリックなどの最上位のレベルナビゲーションは送信し、`<image>`, `<iframe>`, `<script>`, `fetch()`などのサブリソースからの送信は許可しない
- None
    - 制限なし
    - Noneを利用する場合はSecureの指定が必須となる

この「同一ドメイン」という表現も厳密には良くなくて、MDNの公式では以下のように記載されています。

> このより正確な定義では、サイトはドメイン名の登録可能なドメイン部分によって決定されます。登録可能なドメインは、[公開接尾辞リスト](https://publicsuffix.org/list/)の項目と、その直前のドメイン名の部分から構成されます。つまり、たとえば、theguardian.co.uk、sussex.ac.uk、bookshop.org はすべて登録可能なドメインということになります。
>
> この定義に従えば、support.mozilla.org と developer.mozilla.org は同じサイトの一部です。 mozilla.org が登録可能なドメインだからです。

参考元：https://developer.mozilla.org/ja/docs/Glossary/Site

[公開接尾辞リスト](https://publicsuffix.org/list/)の単位で同一のドメインであれば同一サイトと見なされるようで、support.mozilla.org と developer.mozilla.orgは同一サイトと判断されるようです。

# Expires
Cookieの有効期限を日時ベースで設定します。設定がない場合はブラウザ終了時にCookieが削除されます。

```
Expires=Wed, 09 Jun 2025 10:18:14 GMT
```

Expiresに設定できる最大値は 2038年1月19日 03:14:07 UTC とされています。これは32ビット符号付き整数で表現できるUNIXタイムスタンプの最大値（2,147,483,647秒）に対応しています。

では2038年1月19日以降に期限を設定したい時はどうするかというと、次に紹介するMax-Ageを使いましょう。

# Max-Age
Cookieの有効期限を秒数で指定することができます。Max-AgeはExpiresよりも優先されます。
```
Max-Age=3600
```

# まとめ
最後に表にまとめます。
| 属性          | 目的       | 認証Cookieでの推奨 |
|-------------|----------|--------------|
| HttpOnly    | XSS対策    | 必須           |
| Secure      | 盗聴対策     | 必須           |
| SameSite    | CSRF対策   | Strict推奨     |
| Expires, Max-Age     | 有効期限     | 適切に設定        |

上記以外にも様々なCookieの属性がありますが、挙げだすとキリがないので代表的なものだけに絞ります。
