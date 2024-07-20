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