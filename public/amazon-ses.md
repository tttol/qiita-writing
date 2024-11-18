---
title: Amazon SESのメール配信について深堀り/Deep dive into deliverability of Amazon SES!
tags:
  - 'AWS'
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに
最近Amazon SESをよく利用しています。SESでは送信したメールが正しく配信されたかどうかを確認する機能が豊富に用意されているのですが、メール配信に対する自分の理解が乏しいためなんとなくふわっとした状態でSESのコンソールを眺めていることが多いです。エンジニアとして健全な状態ではないので、今一度学習し直そうと思います。
# 送信と配信の違い
SESの仕様以前に、メールの世界には「送信」と「配信」という似た言葉があります。同じ意味のように見えますが意味が異なります。
### 送信
送信とは「メールが送信者の手元から離れ受信者に向けて送られたこと」を指します。
メールが手元から離れるというのは具体的にどういうことでしょうか。これは視点によって解釈が異なってきます。

例えばアプリサーバーの視点から見た場合、アプリからメールの送信リクエストを送りそのリクエストがSMTPサーバに問題なく到達した時点で「送信された」と判断することが多いです。一方でSMTPサーバの視点から見た場合、メール送信リクエストを受信しただけではメールを送信したとは言い難いです。SMTPサーバは宛先の受信サーバであるPOP3/IMAPサーバに送る義務があるので、受信サーバへリクエストを問題なく送信した時点で「送信した」と言えます。
### 配信
配信とは「メールが受信者のメールボックスに届いたこと」を指します。受信トレイだろうと迷惑メールフォルダだろうと、受信者のメールボックスに届いていればそれは配信されたと言えます。バウンスにより受信者に届かなかった場合は配信に失敗したことになります。

参考：

https://docs.aws.amazon.com/ses/latest/dg/send-email-concepts-deliverability.html

# 配信におけるイベント
メールが正しく配信された後、そのメールがどう扱われるかによって名付けられる事象が変化します。

### ハードバウンス/Hard Bounce
メール配信が永続的に失敗する状況にある時、SESはハードバウンスとして処理します。具体的には、存在しないメールアドレスであったり、メールアドレスドメインのDNSの名前解決に失敗したりなどが考えられます。ハードバウンスは基本的に何度リトライしてもメールが配信されることは望めないので、宛先メールアドレスの見直しなどの調査を行う必要があります。SESの公式ドキュメントにも以下の記載があります。

> We strongly recommend that you do not make repeated delivery attempts to email addresses that hard bounce.

https://docs.aws.amazon.com/ses/latest/dg/send-email-concepts-deliverability.html#:~:text=We%20strongly%20recommend%20that%20you%20do%20not%20make%20repeated%20delivery%20attempts%20to%20email%20addresses%20that%20hard%20bounce.

### ソフトバウンス/Soft Bounce
一時的な配信失敗はソフトバウンスとして処理されます。宛先のメールボックスが満杯だったり、何らかの理由で宛先とのコネクションが確立できなかったりなどが考えられます。ソフトバウンスの場合SESは何度かリトライを試みます。（リトライの試行上限回数まで）

:::note info
bounceとは英語で「跳ね返る」という意味です。メールを送ったが跳ね返ってきた、みたいなイメージでしょうか。
:::

### 苦情/Complaint
受信者がメールを迷惑メールフォルダに移動したりスパムとして設定したりした場合、Eメールプロバイダーは「苦情」をSESに送ります。SESコンソールで「苦情」と表示されるメールアドレスは受信者側でスパム扱いされていることになります。

### サプレッションリスト/Suppression list
サプレッションリストとはメール送信を抑制するメールアドレスのリストです。リストに登録されているメールアドレスにSESから送信を実施しようとした場合、送信処理としては成功しますがメール配信は実施されず、自動的にハードバウンスとして処理されます。何をしたらリストに登録されるのか？という具体的なロジックを見つけることはできませんでしたが、ハードバウンスされたメールアドレスはリストに登録される可能性が高いと思われます。

サプレッションリストにはGlobal suppression list/Account-level suppression listの2種類が存在します。Globalの方はSESが管理するマネージドなリストであり、Account-levelはユーザーがAWSアカウント単位で自由に管理できるリストです。
https://docs.aws.amazon.com/ses/latest/dg/sending-email-suppression-list.html