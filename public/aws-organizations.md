---
title: おひとりさまAWS Organizationsに入門する
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
「おひとりさまAWS Organizations」とは、その名の通りAWS Organizationsを一人で使うことです。
AWS Organizationsは複数のAWSアカウントを一元管理する機能なので、主に企業等が利用することを想定されたサービスです。
ですが、これをあえて一人で使います。

ハンズオン用アカウントなどで一人で複数アカウント発行することもあるのと、Organizationsについて学んでおきたかった、というのが主な理由です。

# 用語
まずは用語のお勉強です。

![aws-organizations.drawio2.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/159675/a7e80343-efc7-93a8-fdfd-8cb1048422c5.png)

### AWS Organizations(以下Organizationsと呼ぶ)
- 複数のAWSアカウントを一元管理することのできるサービス
- 利用料金無料
### 管理アカウント
- Organizationsの管理を行うアカウント
- 後述するOUの作成・削除、メンバーアカウントをOUに移動させたりなどを行う
### Organizational Unit(以下OUと呼ぶ)
- 組織単位とも呼ばれる
- Organizationsを親とする子単位
- 管理アカウントと1対Nの関係
### メンバーアカウント
- Organizationsに属するアカウント
- OUと1対Nの関係
### サービスコントロールポリシー(以下SCPと呼ぶ)
- OUやアカウントに対して適用するポリシー
- 複数のアカウントに対して一括で適用できる点がメリット
- ポリシーはJSONで定義する
- 例：下記は`FullAWSAccess ポリシー`のJSON
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "*",
      "Resource": "*"
    }
  ]
}
```
### IAM Identity Center(以下IdCと呼ぶ)
- 利用料金無料
- IdC内でユーザー・グループを作成して、AWSアカウントに紐づける事ができる
- 作成したユーザーで、AWSへのSSOログインを可能にする
### 許可セット
- 1つ以上のIAMポリシーを複数のIdCユーザー・グループに適用するテンプレート

# SCPと許可セットの違い
SCPと許可セットはどちらもポリシーを設定するが、両者には違いがあります。
SCPは組織アカウントやOUに対して適用され、許可セットはIdCの個々のユーザーやグループに対して適用されます。
マネコンで操作するときも、SCPはOrganizationsの画面から設定し、許可セットはIdCの画面から設定します。

↓SCPの適用範囲
![SCP適用範囲のスクリーンショット](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/159675/d720f5d9-44c9-7775-fae9-b98c676aedb0.png)

↓許可セットの適用範囲
![許可セット適用範囲のスクリーンショット](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/159675/561900b7-473b-ea3d-4a8c-a63b1cc26f99.png)


SCPと許可セットの違いを表にまとめたものが以下です。
|| SCP| 許可セット|
|-|-|-|
| **適用範囲**    | 組織全体のアカウントやOUに対して適用され、全体のアクセス制御を強化する。| 個々のユーザーやグループに対して適用され、特定のアカウント内での権限を管理する。 |
| **目的と使用方法**    | 組織全体のセキュリティポリシーを強制し、各アカウントのIAMポリシーに対する上位の制約として機能する。 | ユーザーやグループに特定のアクセス権限を割り当て、日常の操作やリソース管理を簡単にする。 |

# できたもの

![スクリーンショット 2024-07-22 7.12.12.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/159675/94cd9bf7-a1f6-2ced-ba6f-7bc28e16bd57.png)

Organizations開始前に使っていたメインのアカウントをRoot配下に管理アカウントとして配置しました。
また、SandboxというOUを作りその配下にsandbox-accountというアカウントを追加しました。

画像には写ってませんが、IdCで作ったユーザー・グループが各AWSアカウントにぶら下がっています。

# Organizations, IdCでの操作
Organizations, IdCで行う操作について説明します。

### AWS Organizationsの開始
Organizationsはデフォルト状態では無効になっているため、有効化する必要があります。
マネコンから「Organizations」と検索し、Organizationsの画面から有効化を行いましょう。

### AWSアカウント新規追加とスイッチロール
AWSアカウントを新規に追加する際は、Organizationsの画面から行います。

↓右上のオレンジ色ボタンから
![スクリーンショット 2024-07-22 7.30.09.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/159675/740e0d9a-db08-0a6b-5a55-fd2946b2c8fb.png)


アカウント名メールアドレスなどを入力して作成することができます。
![スクリーンショット 2024-07-22 7.31.24.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/159675/69d60198-afe1-e8b1-c80a-0ad50349fb0d.png)

:::note warn
メールアドレスの注意点
一度AWSアカウントに利用したメールアドレスは、そのアカウントを削除したとしてもメールアドレスの再利用ができません。

公式ドキュメントにも明記されています。
https://docs.aws.amazon.com/accounts/latest/reference/manage-acct-closing.html
> You can't use the same email address that was registered to your AWS account at the time of its closure as the primary email of another AWS account. If you want to use the same email address for a different AWS account, we recommend updating it before closure. See Update the AWS account name, email address, or password for the root user for instructions on updating your email address.

sandbox用アカウントやハンズオンアカウントを作成する際は拡張メールアドレスやエイリアスアドレスなどを使ってテスト用のアドレスを取得することをおすすめします。
私はGmailのエイリアスアドレスを使いました。

https://dekiru.net/article/16418/#google_vignette

:::

アカウントが作成されたら、スイッチロールでそのアカウントにサインインします。

### IdCユーザー・グループ・許可セットの追加
AWSアカウントが作成できたら、その配下にIdCのユーザー・グループを紐づけていきます。
こちらの記事を参考に実施しました。

https://qiita.com/kyooooonaka/items/af3b36d5e946b3152021

UIに沿って操作すれば、詰まるところはなかったです。
ユーザー作成時に指定するメールアドレスは、アカウント作成時と同じくエイリアスアドレスを使いました。
許可セットは任意のポリシーを選びましょう。自分はAdministratorAccessとPowerUserAccessを作成しました。

ユーザー作成するとサインインURLがメールで送られてきます。アクセスすると以下のような画面になります。
![スクリーンショット 2024-07-22 7.47.57.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/159675/0b4e1677-14de-7e2f-7e58-0522ab274cba.png)

サインイン後は以下のように、自身の親のアカウント名と許可セット名が表示されます。
![スクリーンショット 2024-07-22 7.50.53.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/159675/b7d58e1c-ad64-ca9d-fa97-fb191cf1f819.png)

# さいごに
最後に、おひとりさまAWS Organizationsを実際に行ってみて感じたメリット・デメリットを挙げます。
### メリット
- ハンズオン・個人開発など用途に応じたAWSアカウントを一元管理できる
- Organizationsの勉強を一人でできる
- Organizations, IAM Identity Centerはどちらも追加料金無しのためコスパ良し
### デメリット
- 漢のシングルアカウント運用と比べるとどうしても複雑さは増す

# 参考

https://dev.classmethod.jp/articles/overview-introduction-to-organizations-enabled-configuration-procedure/

https://www.slideshare.net/slideshow/embed_code/key/kuxZbeoTWPfVLJ

https://speakerdeck.com/htan/awsgakao-eruzui-di-quan-xian-shi-xian-henoapurotigai-lue-aws-iam-identity-center-noquan-xian-she-ji-nituitemokao-etemiru

https://dekiru.net/article/16418/#google_vignette

https://qiita.com/kyooooonaka/items/af3b36d5e946b3152021