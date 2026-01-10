---
title: ログイン中のAWSアカウントが持つIAMロールを一覧表示する方法
tags:
  - 'AWS'
  - IAM
  - CLI
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに
AWSコンソール上でリソースの作成・更新をしようと思ったら、権限がない！

こんな経験はないでしょうか。私は数えきれないほどあります。
その度、ログイン中のアカウントにアタッチされたIAMロールを確認し、社内のAWSインフラ部隊に「権限ないんですけど」と問い合わせを送ります。

企業などの組織におけるそれぞれのAWSアカウントの権限は基本的に最小構成で設計していることが多いと思います。これはセキュリティの観点からはベストプラクティスであり、過剰な権限をユーザーに渡すリスクを抑えることができます。ただ一方で、組織内の各人のユースケースに沿った適切なロールを渡すこともまた難しいです。

組織の人数が増えるほど「権限ないんですけど」の発生頻度も増加します。そんな時、自分のアカウントの所持ロールをサッと確認する方法が欲しくなります。

# IAMロールを一覧で出力するCLIツールを作りました
毎回IAMを調べるのも大変なので、アタッチされたロールを一発で表示するコマンドラインツールを作りました。canidoという名前のツールです。

https://github.com/tttol/canido

canidoを使うことで、ログイン中のAWSアカウントの所持ロールを確認することが可能です。
例えばこんな感じ。
```bash
% canido

--- Checking AWS credentials ---
Target role: AWSReservedSSO_CanidoInlinePolicy_f1d7ab46757a3473

==================================================
  1. Managed Policies
==================================================
[Policy ARN]: arn:aws:iam::aws:policy/IAMFullAccess
{
  "Statement": [
    {
      "Action": [
        "iam:*",
        "organizations:DescribeAccount",
        "organizations:DescribeOrganization",
        "organizations:DescribeOrganizationalUnit",
        "organizations:DescribePolicy",
        "organizations:ListChildren",
        "organizations:ListParents",
        "organizations:ListPoliciesForTarget",
        "organizations:ListRoots",
        "organizations:ListPolicies",
        "organizations:ListTargetsForPolicy"
      ],
      "Effect": "Allow",
      "Resource": "*"
    }
  ],
  "Version": "2012-10-17"
}
--------------------------------------------------

==================================================
  2. Inline Policies
==================================================
[Policy Name]: AwsSSOInlinePolicy
{
  "Statement": [
    {
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Effect": "Allow",
      "Resource": [
        "*"
      ],
      "Sid": "Statement1"
    },
    {
      "Action": [
        "secretsmanager:DescribeSecret",
        "secretsmanager:GetRandomPassword",
        "secretsmanager:GetResourcePolicy",
        "secretsmanager:GetSecretValue",
        "secretsmanager:ListSecretVersionIds",
        "secretsmanager:ListSecrets",
        "secretsmanager:BatchGetSecretValue"
      ],
      "Effect": "Allow",
      "Resource": [
        "*"
      ],
      "Sid": "Statement2"
    }
  ],
  "Version": "2012-10-17"
}
--------------------------------------------------
```

仕組みとしては単純で、以下のコマンドを順番に呼び出した結果を成形して出力しています。
```bash
% aws sts get-caller-identity
% aws iam list-attached-role-policies --role-name MyRole
% aws iam get-policy-version --policy-arn ... --version-id ...
```


インストール方法はREADMEに書いてある通りですが、macOSの場合はHomebrewから取得してください。ただし、先にHomebrew tapのマウントが必要です。
```bash
brew tap tttol/tap # tapをマウント
brew install canido # canidoをインストール
```

Linuxの場合は以下の手順でバイナリからインストールしてください。Macでもこの手順は適用可能です。
```bash
# For x86_64 (Intel/AMD)
curl -LO https://github.com/tttol/canido/releases/latest/download/canido-x86_64-unknown-linux-gnu.tar.gz
tar xzf canido-x86_64-unknown-linux-gnu.tar.gz
sudo mv canido /usr/local/bin/
canido --version
```

Windowsの方はWSL2からLinuxと同じ手順を踏んでください。すみませんがコマンドプロンプトとPowerShell向けのバイナリは用意できていません。（私の手元にWindows環境がないので…）

# 工夫した点
### 必要最低限の機能に絞った
「ログイン中のAWSアカウントにアタッチされたIAMロールを出力する」だけの機能に特化し、それ以外の機能はドロップしました。アカウントIDを受け取ってそのアカウントのIAMロールを出力する、IAMロールのARNを受け取ってIAMポリシーを出力する、などの機能もその気になれば実装できそうでしたが、やめました。
canido作成のモチベーションは「今ログインしているアカウントのロールは何かを知りたい」という点なので、現在ログインしていないアカウントや他のロールのことはcanidoスコープ外としました。ツール名のcanidoの元ネタのCan I do?も、「私には何ができるのか？」という自分自身のみに焦点を当てた表現からとってたりします。

### --shortオプション
所持するIAMロールが多かったり、一つのIAMロールに含まれるIAMポリシーが多数存在する場合は出力が長くなって可読性が下がります。対策として--shortオプションを追加し、アカウントにアタッチされたIAMロール名のみを一覧で出力するオプションを追加しました。

### Homebrewで配布した
自分はMacユーザーなのでcanidoはHomebrewで配布しました。簡単に言ってますがHomebrewでのツール配布は初めてだったので、調べながら手探りでやっていきました。

Homebrewでパッケージを配布したい場合、homebrew/coreという公式のリポジトリに追加する必要があります。追加には一定の審査基準があり、オープンソースであることや一定の知名度（GitHub Starなど）が求められます。

canidoのような知名度の低いツールをcoreに追加するのはハードルが高いので、代わりにtapという機能を使います。これは個人利用や小規模なプロジェクト向けに用意された審査なしで配布できる方法です。ただし、tapを使った場合はインストール時にtapをマウントするコマンドが1行必要になります。
```bash
brew tap tttol/tap # tapをマウント
brew install canido # canidoをインストール
```


# さいごに
canidoがAWSユーザーに広まってくれると嬉しいです。ぜひGitHubにStarをつけてください！
