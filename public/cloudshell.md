---
title: AWS CloudShellは便利なのか？
tags:
  - 'AWS'
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---
AWS CloudShellとは、AWSコンソール上から操作できる簡易のシェルです。CloudShellはAWSアカウントへの認証が既に済んだ状態で起動するためこちらでログイン操作などをする必要がなく、ブラウザで開いているAWSアカウントに対してすぐにCLI操作を実行することが可能です（`aws configure`などが不要）。

AWSコンソールの左下にある`CloudShell`を押下すると以下のようにシェルがひょこっと表示されます。

![スクリーンショット 2025-12-16 5.10.29.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/159675/c3188037-fe3d-4a37-8b13-ca2eaf65d48e.png)
ちょっとCLIコマンドからリソースにアクセスしたいなあって時に地味に便利な機能なんですが、実務でガッツリCloudShellを使うことってあまりないような気がします。企業で運用しているAWSアカウントではそもそもOrganizationsのSCPなどでCloudShellの利用自体を制限していることもありますし。そんなCloudShellを最近実務で少し触る機会があり、細かい仕様がどうなっているかが気になるシーンがあったので、少しまとめてみます。

# スペック・Linuxディストリビューション
CloudShellのスペックは以下の通りです。
- 1 vCPU (virtual central processing unit)
- 2-GiB RAM
- 1-GB persistent storage (storage persists after the session ends)

最低限のスペックって感じですね。メモリ・CPUをガッツリ消費して何か作業するような想定はされてなさそうなスペックです。

また、CloudShellに利用されているLinuxディストリビューションはAmazon Linux 2023です。ここはユーザーから変更できないようです。

# プリインストールソフトウェア
プリインストールされているソフトウェア以下です。

AWS関連ツール:
- AWS CLI v2
- AWS SAM CLI
- AWS CDK
- EB CLI（Elastic Beanstalk）
- ECS CLI

開発ツール・ランタイム:
- Git
- Python 3（pip含む）
- Node.js & npm
- PowerShell

その他のツール:
- Docker（利用は制限あり）
- vim, nano などのエディタ
- bash, zsh
- curl, wget
- tar, zip, unzip


割と色々入ってます。SAMやCDKが元から入っているのは嬉しいところですね。
ここにないソフトウェアも必要に応じてapt-getでインストール可能です。ただし、セッションが切れると追加インストールしたものは全て削除されてしまいます。ホームディレクトリは永続化されているようなので、~/.bashrcにインストールスクリプトを書くなどしてユーザー側でケアする必要があります。

# 権限制御
CloudShellからAWS CLIを使うと、その気になればAWSリソースに対して作成・更新・削除などなんでもできてしまいます。実務ではAWS Organizationsなどで開発メンバーのロールを一括管理することが多いので、CloudShellもSCPやRCPなどで制御できると助かります。

調べたところ、AWS CLIレベルでの制限は可能だがUNIXコマンドレベルでの制御は不可能、という感じでした。

例えばCloudShell自体は利用を許可するがEC2へのアクセスは許可しない、は以下のように表現できます。
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowCloudShell",
      "Effect": "Allow",
      "Action": [
        "cloudshell:*"
      ],
      "Resource": "*"
    },
    {
      "Sid": "DenyEC2Operations",
      "Effect": "Deny",
      "Action": [
        "ec2:TerminateInstances",
        "ec2:DeleteVolume"
      ],
      "Resource": "*"
    }
  ]
}
```

「curl, wgetコマンドの実行を禁止する」などのUNIXコマンド単位での制御はできません。
なので、悪意を持ったユーザーがcurlでよからぬソフトをDLしたり、wgetでマイニングソフトウェアをインストールしたりなどの行為を防ぐことは難しいです。

# 雑感
プリインストールソフトウェアがそこそこあるので、ちょっとCLIからAWSリソースを操作したいなって時には便利ですが、スペックが貧弱なのと権限制御がざっくりしているところが残念ですね。特に権限のところは、企業などの大きな組織で使う場合はどうしてもリスクがあるので、もうCloudShellは一括で利用禁止にしよっかってなりがちな気がします。この辺がもう少しアップデートされると良さそうです。
