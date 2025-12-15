---
title: AWSリソース消し忘れで実際に高額請求されないための対策
tags:
  - AWS
private: false
updated_at: '2025-12-15T06:06:28+09:00'
id: fa12c4a769e263a0c0be
organization_url_name: null
slide: false
ignorePublish: false
---
私は個人でAWSアカウントを発行し、個人開発やちょっとした技術検証に利用しています。
AWSリソースは従量課金のものが多いので、リソースを作った後に消し忘れてしまうとずっと課金されてしまい意図せず高額な料金が発生してしまうことが**稀によくあります。** Qiitaでもこの手の記事は定期的に上がってきますね。

https://qiita.com/riiiii/items/4eafbbbe885f314430e5

https://qiita.com/KNR109/items/01614b62eaf33f33bd78

自分も何度かやらかして意図しない請求をもらってAWSサポートに泣きついたことがあります。

自戒のために経緯と対処を記事にまとめます。

# ブロックチェーン消し忘れで約500ドル課金
Amazon Managed Blockchain(AMB)というブロックチェーンのサービスを使って、個人的に技術検証をしていた時期がありました。AMBは従量課金のサービスなので、稼働している間は課金がされ続けます。

AMBのリソース消し忘れにより、最終的に500ドル近い請求が発生してしまいました。日本円にして大体75,000円くらい。。。

当時、AWS Community Buildersの特典でクレジットをもらっており、470ドルほど余ってたのですがそのクレジットは全て使い切ってしまい、30ドルほどは自腹で払う必要がある状態でした。AWSサポートに問い合わせたところ、クレジット消費分は返金できないが現金で支払う分の30ドル分は返金可能とのことでした。クレジット分が本当にもったいないですが、自業自得なので泣く泣く受け入れました。

ちなみに返金の際に、再発防止策としてAWS Budgetsから予算アラートを設定してねとサポートの方から指示を受けました↓

```
///////////////
1．意図しない請求の予防策として以下の内、少なくとも 1 点の対策をお願いいたします。
※ご実施いただいた内容をご記載ください。
※設定が比較的容易なものに★を付けさせていただきました。
///////////////

▼シナリオ: CloudWatch で予想請求額をモニターリングする★
https://docs.aws.amazon.com/ja_jp/AmazonCloudWatch/latest/monitoring/gs_monitor_estimated_charges_with_cloudwatch.html 

CloudWatch の請求アラームが INSUFFICIENT_DATA の状態になっている場合、下記記事をご参照のうえ、ご対応お願いいたします。

▼CloudWatch アラームが INSUFFICIENT_DATA の状態になっている場合のトラブルシューティング方法を教えてください。
https://repost.aws/ja/knowledge-center/cloudwatch-alarm-insufficient-data-state 

▼AWS Budgets を用いてコストを管理する★
https://docs.aws.amazon.com/ja_jp/cost-management/latest/userguide/budgets-managing-costs.html 

▼AWS CloudTrail とは
https://docs.aws.amazon.com/ja_jp/awscloudtrail/latest/userguide/cloudtrail-user-guide.html 

▼AWS Trusted Advisor の開始方法
https://docs.aws.amazon.com/ja_jp/awssupport/latest/user/get-started-with-aws-trusted-advisor.html 

▼AWS Cost Anomaly Detection（コスト異常検出）
https://aws.amazon.com/jp/aws-cost-management/aws-cost-anomaly-detection/ 
```

個人アカウントなのでこの辺サボってました。。急いで設定しました。

# CodeBuild消し忘れで約160ドル課金
AMBでの500ドル事件のあと、またしてもやらかしました。

今度はCodeBuildで160ドル近く課金されていました。

CodeBuildなんて使ってないぞ？！と思いながらリソースを確認していたんですが、どうやらCDKのinteg-testを実行した時に作成されたCodeBuildプロジェクトがcfnからきちんと削除されておらず、ずっと裏で動き続けていたようです。。

AWS Budgetsの月次予算アラートは鳴らしていたんですが、閾値を1ドルのアラートと10ドルのアラートの2種類しか作成しておらず、10ドル以上を超過してからは特にアラートが来ないようになっていました。この10ドルという閾値も割と適当に設定した値なので、あまりアラートが機能していませんでした。

あと、そもそもAWSからメールをあまり見ていなかったというも大きな問題でした。。。
（メールって埋もれるんですよね）

幸い、この時もAWSクレジットが余っていたのでクレジット消費だけで賄えました。クレジット消費分は返金できないことは前回のAWSサポートの回答からわかっていたので、今回はもうサポートに問い合わせていません。

# AWS Budgetの設定見直し
さらなる再発防止策としてAWS Budgetsの設定を見直します。


### 月次予算アラートを細かく設定する
1ドルと10ドルの閾値しか設定していませんでしたが、ここをもっと細かく段階的に設定することにしました。
10ドル刻みで1, 10, 20, 30, ... , 100ドルまで設定しました。

### Cost Anomaly Detectionを設定する
AWS BudgetsにはCost Anomaly Detectionという機能があり、コストが不自然に跳ね上がった際にそれを異常現象とみなして検出してくれる機能があります。機械学習を使って過去の支出パターンを学習し、通常と異なる支出を自動検知してくれます。追加料金は完全無料。

月次予算アラートは絶対値としての閾値をベースにするのに対して、Cost Anomaly Detectionは過去の支出との相対的な変化をベースにしする点がポイントです。

AWSコンソール上で `コスト異常検出` と表示されているリンクをクリックすることで設定が可能です。
![スクリーンショット 2025-12-15 5.59.02.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/159675/e5691f38-771d-4e5c-abc2-2d63f9ce0fb8.png)

# まとめ
自分の過失で意図せぬ高額請求がきても、AWSサポートに問い合わせれば割となんとかしてくれることが多い印象です。（だからと言って甘えていいわけではないですが）

AWS Budgetsにはこうした事態にならないように様々な機能が実装されているので、個人アカウントであってもきちんと予算アラートの設定をすべきだなと痛感しました。
