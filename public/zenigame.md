---
title: AWS Amplifyで家計管理アプリを開発して個人的に運用している話
tags:
  - AWS
  - amplify
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに
こんにちは。

この記事では、私がAWS Amplify（以下Amplifyと呼びます）を利用して開発・運用している家計管理アプリを紹介します。
実は過去にとある勉強会でもこのアプリについて登壇したことがあります。

https://speakerdeck.com/tttol/amplifytekai-fa-yun-yong-siteiru-ge-ren-kai-fa-ahurishao-jie

本記事は、この登壇資料をより詳細に記述したものになります。

:::note warn
このアプリは個人開発アプリであり、商用利用は想定していません。あくまで私と私の家族だけが利用することを想定しています。
:::


# 想定読者
- Webアプリケーションを使って課題解決を行いたい方
- AWS Amplifyについて聞いたことはあるが詳細まで知らない方
- AWSリソースを利用してWebアプリケーションを手軽に開発したい方
- アプリの知識はあるがインフラの知識に自信がない方
# 我が家の家計管理の課題
「我が家の課題：家族間における立替精算のやりとりをなくしたい」

私は妻とよく立替精算の会話をします。例えば妻がスーパー等で買物に行ってくれたとき、その代金は一時的に妻が立て替えて、後日代金の半額分を私が妻に支払います。

```
妻「買い物行ってきたよ。立て替えたのでお金あとでちょうだい。」

自分「ありがとう。そういえば昨日薬局でティッシュ買ったから、この分と相殺したいな。」

妻「あーでも明日スーパーも行きたいから、それも一緒に・・・」

自分「んあーーー」←ここで考えるのがめんどくさくなる。
```

日々、こうした精算作業を行うのはつらいので、どうにかして立替状況を管理したいところです。
私が最初に思いついた管理方法は、Googleのスプレッドシートで管理する方法です。

![スクリーンショット 2024-08-18 10.30.40.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/159675/4e3080f0-8f51-01b6-33bb-ba8524791ed0.png)

夫・妻がそれぞれ支払った額と品目を記録していき、最下行のSUMIF関数で各自の最終支払額を計算するものです。
SUMIFで算出された負債の支払いは毎日行う必要はなく、週に一回・月に一回などまとまったタイミングで精算します。

このスプレッドシートを妻と私で共有し、互いに記録していくことで立替精算を楽にしようと試みました。

# スプレッドシート管理の問題点
スプレッドシート管理によりそこそこ楽になったのですが、同時に問題点も浮かんできました。

- スマホからスプレッドシート入力するのが大変
  - 外出先などで入力するときはスマホアプリ版のスプレッドシートから入力することになる
  - 画面が小さい・・・
  - 入力が億劫になり、レシートがどんどん溜まっていく
- 表がいっぱいになったら行追加が必要。これもスマホからだと大変。
  - 行が増えるとスクロールが必要になる
  - ぱっと見で自分に何円分の負債があるのかわからない

といった具合で、**結果的にPCから入力することが多くなり、家で暇な時に一気に入力することが増えます。**

これらの問題点を解決すべく、スプレッドシートの持つ機能をWebアプリケーションに落とし込みます。

# Amplifyでアプリケーション化
Next.jsでアプリケーションを作成し、Amplify Hostingで公開しました。
![スクリーンショット 2024-08-18 11.02.07.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/159675/fef6b1fb-9c51-8ce9-c2e1-3672a9c9ee7e.png)

# アプリアーキテクチャ
アプリのアーキテクチャは以下の通りです。
![スクリーンショット 2024-08-18 11.03.24.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/159675/45adc200-d966-e2b8-20e9-ebb7eb2ee90c.png)

Amplifyに習熟している方にとってはおなじみの構成だと思います。

### Hosting
Amplify Hostingを利用しました。
裏ではS3とCloudFrontが動いており、こちらでHTMLの配信を行います。
![スクリーンショット 2024-08-18 11.06.08.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/159675/393b1795-239c-08c4-3d3d-df3a535a4377.png)

ここで利用されているS3バケットとCloudFrontディストリビューションは閲覧・編集することができません。
AWSの内部で管理されているリソースのようです。

### Database & API
DynamoDBを利用しました。各人の支払った品目をここに保存します。
CRUD操作はApp Syncを利用してGraphQLで実行します。

Amplifyではamplify/data/resource.tsというファイルが自動で生成され、ここにDynamoDBのアーキテクチャをIaCで記述することができます。

```typescript:amplify/data/resource/ts
import { a, defineData, type ClientSchema } from '@aws-amplify/backend';

const schema = a.schema({
  Todo: a.model({
      content: a.string(),
      isDone: a.boolean()
    })
    .authorization(allow => [allow.group('Admin')]), // User-group based data access
});

// Used for code completion / highlighting when making requests from frontend
export type Schema = ClientSchema<typeof schema>;

// defines the data resource to be deployed
export const data = defineData({
  schema,
  authorizationModes: {
    defaultAuthorizationMode: 'apiKey',
    apiKeyAuthorizationMode: { expiresInDays: 30 }
  }
});
```

参考
https://docs.amplify.aws/react/build-a-backend/data/set-up-data/
https://docs.amplify.aws/react/build-a-backend/data/customize-authz/user-group-based-data-access/

### Auth
Cognitoを利用しました。

DBと同様に、認証についてもamplify/auth/resource.tsというファイルが自動で生成され、ここに認証のアーキテクチャを記述することができます。
```typescript:amplify/auth/resource.ts
import { defineAuth } from "@aws-amplify/backend"

/**
 * Define and configure your auth resource
 * @see https://docs.amplify.aws/gen2/build-a-backend/auth
 */
export const auth = defineAuth({
  loginWith: {
    email: true,
  },
})
```

参考
https://docs.amplify.aws/react/build-a-backend/auth/set-up-auth/

# アプリ特徴
アプリケーションの特徴について解説します。

### 上段にサマリーを表示
### プルダウン・デートピッカー・ラジオボダンで手入力作業を極力排除
### 私と家族だけがアクセスできるように権限を制御
### 開発環境・本番環境を用意
# 運用コスト
# さいごに