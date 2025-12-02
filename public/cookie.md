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
sessionid=123456;theme=dark; ...
```

Cookieには様々な情報が保持されるので、万一Cookieの情報を悪意のある第三者に盗聴・奪取されるとセキュリティインシデントに繋がります。開発者はCookieの情報が外部に漏洩しないようにアプリケーションを設計・実装することが肝要です。そのために、Cookie側が属性という形で様々な防御機能を提供しています。代表的なものを以下に挙げていきます。

- HttpOnly
- Secure
- SameSite
- Domain
- Path
- Expires
- Max-Age
- Partitioned

例えば`HttpOnly`, `SameSite`が付与されたCookieは以下のようになります。
```
Set-Cookie: sessionid=abc123; HttpOnly; Secure; SameSite=Strict
```

# HttpOnly
HttpOnly属性が付与されたCookieはJavascriptからCookieにアクセスできなくなります。例えば`document.cookie`プロパティからの利用が不可能になります。XSSなどでCookieを奪われるリスクを防ぐことが可能です。

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
Cookieを送受信する際にドメインに応じて制御をかけます。SameSiteは3つの値を持ちます。

- Strict
    - 同一ドメインからのリクエストのみ送信を許可
- Lax
    - 
- None
    - 制限なし
    - Noneを利用する場合はSecureの指定が必須となる
# Domain
# Path
# Expires
# Max-Age
# Partitioned
# まとめ
  | 属性          | 目的       | 認証Cookieでの推奨 |
  |-------------|----------|--------------|
  | HttpOnly    | XSS対策    | 必須           |
  | Secure      | 盗聴対策     | 必須           |
  | SameSite    | CSRF対策   | Strict推奨     |
  | Path        | スコープ制限   | /推奨          |
  | Max-Age     | 有効期限     | 適切に設定        |
  | Partitioned | プライバシー保護 | サードパーティで検討   |
