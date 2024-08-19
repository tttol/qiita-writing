---
title: AWS Amplifyで私用の家計管理アプリを開発して運用している話
tags:
  - AWS
  - amplify
  - "Next.js"
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
- Amplifyについて聞いたことはあるが詳細まで知らない方
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

画面上部の`SUMMARY`が各人の負債を表しています。

その下にあるエリアに表示されているのは、立て替えた品目のリストです。
`食材買い出し`と`ティッシュ補充`という2つの品目が登録されていることがわかります。
この2つの立て替え品目から計算した結果、夫の負債は0円・妻の負債は1,051円であるという状態です。
※負債の計算方法については後述します。

品目の追加は画面から可能です。
![スクリーンショット 2024-08-18 11.02.18.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/159675/06899d3b-44c9-2e90-75dc-e8728bf7abfe.png)


品目作成時の画面の操作性を良くすることで、先述したスマホ操作に関する問題点を解消しました。
ラベル選択にはプルダウンを、日付入力はデートピッカーを、支払者の選択はラジオボタンを採用することで、操作性を良くしました。

### 負債の計算方法について
> - ぱっと見で自分に何円分の負債があるのかわからない

先述した上記問題点の解決策として、アプリの画面最上部に各人の負債状況をサマリーで表示するようにしています。
これにより、アプリを表示したときに自身の立替状況がひと目でわかります。

サマリーでは「未払い差引合計」と「差引前」の2項目を表示しています。
これらはそれぞれ下記の意味を表しています。
- 「未払い差引合計」・・・互いの未払い金額を引き算して、最終的にどちらが何円払うべきかを差引精算した額を表示
- 「差引前」・・・各人の未払い額の合計を表示

具体例を挙げます。

画像では夫が2,500円立て替え済み、妻が398円立て替え済みの状態です。
夫は398円の半額である199円を妻に支払う必要があります。
妻は2500円の半額である1250円を支払う必要があります。
この2つの値が、「差引前」に表示される値です。

この2値を差し引きすると、1250 - 199 = 1,051となり、最終的に妻が夫に1,051円支払えばよいということになります。

よって、「未払い差引合計」は夫が0円で妻が1,051円となります。

| | 差引前 | 未払い差引合計 |
|:-----------|------------:|:------------:|
|**夫**|¥199|¥0|
| **妻**|¥1,250|¥1,051|


# アプリアーキテクチャ
アプリのアーキテクチャは下記の通りです。
Amplifyに習熟している方にとってはおなじみの構成だと思います。

![スクリーンショット 2024-08-18 11.03.24.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/159675/45adc200-d966-e2b8-20e9-ebb7eb2ee90c.png)


### Hosting
Amplify Hostingを利用しています。
裏ではS3とCloudFrontが動いており、こちらでHTMLの配信を行います。
![スクリーンショット 2024-08-18 11.06.08.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/159675/393b1795-239c-08c4-3d3d-df3a535a4377.png)

ここで利用されているS3バケットとCloudFrontディストリビューションは閲覧・編集することができません。AWSの内部で管理されているリソースのようです。

### Database & API
DBはDynamoDBを利用しています。各人の支払った品目をここに保存します。CRUD操作はApp Syncを利用してGraphQLで実行します。

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

冒頭で記載した通りこのアプリは私と妻だけが利用するアプリなので、アプリ内データの閲覧・編集には適切な権限制御が必要です。
Cognito上の特定のユーザーグループに所属するユーザーのみがDynamoDBにアクセスできるよう、data/resource.tsにて権限制御を設定しています。
具体的には下記の部分です。

```typescript:amplify/data/resource.ts
const schema = a.schema({
  Todo: a.model({
      content: a.string(),
      isDone: a.boolean()
    })
    .authorization(allow => [allow.group('Admin')]), // User-group based data access
});
```

`.authorization(allow => [allow.group('Admin')]),`で`Admin`グループに属するユーザーしかアクセスできないよう制御しています。
（`Admin`グループはAWSマネジメントコンソールからCognitoにアクセスし、事前に手動で作成しておきます。）


参考
https://docs.amplify.aws/react/build-a-backend/data/set-up-data/
https://docs.amplify.aws/react/build-a-backend/data/customize-authz/user-group-based-data-access/

### Auth
認証にはCognitoを利用しています。

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

# 運用コスト
Amplifyの利用料金は従量課金制です。

https://aws.amazon.com/jp/amplify/pricing/

私の家計管理アプリの運用コストは100〜200円/月程度です。
まあアクティブユーザーが私と妻の2人だけなので、もはやタダみたいなもんですね。

# さいごに
スプレッドシート管理時代に比べて、アプリで管理するほうが格段に楽になりました。
アプリはまだまだ粗削りな部分もあり、たまにバグも見つかるので、マイペースにメンテナンスしていこうかと思います。