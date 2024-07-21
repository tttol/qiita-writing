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

ここにイメージ図

## AWS Organizations
- 複数のAWSアカウントを一元管理することのできるサービス
- 利用料金無料
## 管理アカウント
- Organizationsの管理を行うアカウント
- 後述するOUの作成・削除、メンバーアカウントをOUに移動させたりなどを行う
## Organization Unit(OU)
- 組織単位とも呼ぶ
- Organizaionsを親とする子単位
- 管理アカウントと1対Nの関係
## メンバーアカウント
- Organizationsに属するアカウント
- OUと1対Nの関係
## サービスコントロールポリシー(SCP)
- 
## IAM Identity Center(IdC)
- IdC
- SSOログインを可能にする
- 利用料金無料
## 許可セット
- 1つ以上のIAMポリシーを複数のAWSアカウントに適用するテンプレート

# できたもの
# おひとりさまAWS　Organizationsのメリット・デメリット
## メリット
- ハンズオン・個人開発など用途に応じたAWSアカウントを一元管理できる
- Organizationsの勉強を一人でできる
## デメリット
- 漢のシングルアカウント運用と比べるとどうしても複雑さは増す

# さいごに
# 下書きメモ
- AWS Organizations
  - 無料
  - 複数のAWSアカウントを管理するサービス
  - 管理アカウント
    - Organizationsの管理者アカウント
  - Organization Unit
    - OU
    - Organizaionの単位
    - 管理アカウントと1対N
  - メンバーアカウント
    - Organizationsに属するアカウント
    - OUと1対N
- AWS IAM Identity Center
  - IdC
  - 無料
  - SSOログインを可能にする
  - 許可セット
    - 1つ以上のIAMポリシーを複数のAWSアカウントに適用するテンプレート

- 手順
  - Idcからユーザーを新規作成
  - 作成したユーザーをOrganizationsのRootに招待


- Root
  - Managemet Account
    - OU 1
      - AWS Account
        - IdC Group/User
        - IdC Group/User
      - AWS Account
        - IdC Group/User
    - OU 2
      - AWS Account
        - IdC Group/User

# 参考

https://dev.classmethod.jp/articles/overview-introduction-to-organizations-enabled-configuration-procedure/

https://www.slideshare.net/slideshow/embed_code/key/kuxZbeoTWPfVLJ

https://speakerdeck.com/htan/awsgakao-eruzui-di-quan-xian-shi-xian-henoapurotigai-lue-aws-iam-identity-center-noquan-xian-she-ji-nituitemokao-etemiru

https://dekiru.net/article/16418/#google_vignette