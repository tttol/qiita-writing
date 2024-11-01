---
title: S3のコンソールからAmplify Hostingを使えるようになったぞ！
tags:
  - S3
  - amplify
private: false
updated_at: '2024-11-01T08:29:22+09:00'
id: 6d890fa96c873b229f1f
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに
2024/10/30に「Simplify and enhance Amazon S3 static website hosting with AWS Amplify Hosting」というタイトルのアナウンスが発表されました。

https://aws.amazon.com/jp/blogs/aws/simplify-and-enhance-amazon-s3-static-website-hosting-with-aws-amplify/

S3のコンソールから静的ウェブサイトホスティングを実行しようとすると、Amplify Hostingを推奨する旨の誘導が追記されました。
![スクリーンショット 2024-11-01 7.21.26.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/159675/2d4c5bdb-da0e-d5c5-ecab-5081fb7d46b5.png)

Amplify好きの私としては、うれしいアプデです。試しに色々触ってみて感じたことを記事にまとめます！

# 事前知識
まず、本記事を読むうえで知っておくべき知識を2つ紹介します。

### S3静的ウェブサイトホスティングとは
S3の静的ウェブサイトホスティングは、HTMLやCSS、JavaScriptなどの静的ファイルをウェブサイトとして公開するための機能です。簡単にウェブサイトを構築でき、カスタムドメインやエラーページの設定も可能です。HTTPS対応はCloudFrontと組み合わせることで実現でき、S3のスケーラビリティを活かして、安価に高可用性なサイトを運営できます。

例えば、任意のS3バケットにindex.htmlを配置して静的ウェブサイトの設定をONにするだけで、index.htmlが静的コンテンツとして配信されます。デフォルトではhttp:// から始まるURLで配信されますが、CloudFrontの設定を追加で行うことでHTTPS対応も可能です。また、Route 53と連携することでカスタムドメインも割り当て可能です。

### Amplify Hostingとは
Amplify Hostingは、フロントエンドウェブアプリケーションのホスティングサービスで、特に静的サイトやSPAに最適です。Gitリポジトリと連携し、自動デプロイやCI/CDパイプラインの構築が可能です。Amplify Consoleでブランチごとに異なる環境を設定でき、バックエンドとの連携や認証、ストレージ機能なども簡単に追加できます。スケーラブルかつセキュアなウェブアプリを迅速にデプロイし、管理するのに適したサービスです。

例えばGitHubにプッシュされたReact.js製のSPAに対して、Amplifyとリポジトリ連携をさせることで簡単にアプリケーションをWeb上にホスティングすることができます。S3の静的ウェブサイトホスティングとの違いは、静的コンテンツ以外の配信も自由に行える点です。

# 手順
本題です。S3のコンソールから静的ウェブサイトホスティングを実行しに行きます。
まずはindex.htmlとstyles.cssという2つのファイルを用意し、hosting-migration-sampleというS3バケットにアップロードしておきます。styles.cssはassetsディレクトリの中に入れています。
![スクリーンショット 2024-11-01 8.03.40.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/159675/61d2ec35-624b-b5c2-7a29-50ba6976e4ed.png)

バケットのプロパティから静的ウェブサイトホスティングを選択肢に行きます。すると冒頭で示した通り、`静的ウェブサイトホスティングには AWS Amplify ホスティングを使用することをお勧めします`という文言と、Amplifyへのリンクが記載されていますね。
![スクリーンショット 2024-11-01 7.21.26.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/159675/2d4c5bdb-da0e-d5c5-ecab-5081fb7d46b5.png)

リンクを踏むとAmplifyの画面が別タブで開かれます。アプリケーションの名前・ブランチ名・配信元の入力を促されます。配信元は既に先程のS3バケットがフィルインされていますね。
![スクリーンショット 2024-11-01 7.26.09.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/159675/0ba2a155-758e-d169-d435-57726968559b.png)

アプリケーション名はこのままでもいいですが、今回はhosting-migration-sampleとしておきました。ブランチ名はそのままstagingにしました。ここでデプロイするモジュールが本番稼働ブランチとして認識されるので、productionやmainなどの名称に変更しておいてもいいですね。

画面に沿って次に進むとデプロイが始まります。今回は軽量なファイルだったので、数秒でデプロイが完了しました。デプロイ後は↓のような画面にリダイレクトします。
![スクリーンショット 2024-11-01 7.27.05.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/159675/da659a21-6b2b-884a-b522-d4ad0e9a8699.png)

記載されたURLにアクセスし、S3にアップロードしたコンテンツが正しく表示されることを確認できました。
![スクリーンショット 2024-11-01 7.27.51.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/159675/348aeb07-8b99-78a7-e9ad-674a35f124fe.png)

# Amplify Hohtingのうれしいところ
### デフォルトでHTTPS対応
S3で配信する場合、HTTPS対応を自分でCloudFrontから設定する必要があります。一方、Amplify Hostingの場合は最初からHTTPSになっているためここの手間が省けます。ただし、Amplify Hostingが裏で設定しているCloudFrontのディストリビューションは**こちらから確認することができない**ため、注意が必要です。

### S3静的ウェブサイトホスティングでできることはAmplify Hostingでも可能
S3静的ウェブサイトホスティングでできることは、Amplify Hostingでも実現可能です。具体的には以下です。
- リダイレクトルールの設定
- エラーページの設定（404, 500など）
- カスタムドメインの設定
- CloudFrontによるキャッシュ制御

実際に試せていませんが、上記はすべてAmplifyのコンソールに設定項目がありました。

# さいごに
個人的に、今後S3のホスティング機能はサ終するのかなあと思います。AmplifyはS3ほど知名度の高いサービスではないので、これを機にAmplifyを知る機会になればいいなと思います。
