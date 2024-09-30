---
title: AWS Amplify製のアプリにパスキー認証を導入する方法
tags:
  - 'AWS Amplify'
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに
AWS Amplify(以下Amplifyと呼ぶ)で作ったWebアプリケーションにパスキー認証を導入した際の作業ログです。
「Amplify × パスキー認証」に関する情報はググってもあまりHITしなかったので、記事にしておこうと思った次第です。
使用してるフレームワークやライブラリのバージョンは執筆時点でのlatestもしくはstableなバージョンを採用しています。参考にされる場合はその点ご留意ください。

# サンプルアプリケーションの解説
↓私が作ったサンプルアプリケーションはこちらです。

https://github.com/tttol/amplify-passwordless-auth/tree/main

私のサンプルでは[aws-samples/amazon-cognito-passwordless-auth](https://github.com/aws-samples/amazon-cognito-passwordless-auth/tree/main)というAWSが公式に配布しているサンプルをnpm installして利用しています。また、公式サンプルには[end-to-end example](https://github.com/aws-samples/amazon-cognito-passwordless-auth/tree/main/end-to-end-example)というReactのサンプルアプリケーションが用意されており、こちらを参考に作ったものとなります。

:::note warn
[aws-samples/amazon-cognito-passwordless-auth](https://github.com/aws-samples/amazon-cognito-passwordless-auth/tree/main)はサンプルにしてはかなりしっかり実装されていますが、あくまでもサンプルなので、今後このリポジトリが継続的にメンテナンスされるかどうかはわかりません。
（実際、aws-samplesの中には公開して数年後にpublic archiveになっているリポジトリもあります。）
:::

サンプルアプリケーションの構成は以下のとおりです。
※ここにmermaid製のAWSアーキテクチャ図

`$ npm run dev`でアプリを起動すると、サインイン画面が表示されます。
![スクリーンショット 2024-09-30 7.01.59.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/159675/bdfefd0b-a0c1-e593-60fc-90cae303a7eb.png)

### Magic Linkでサインイン
初回訪問時はパスキーの登録がないため、メールアドレスでサインインします。`Enter your e-mail address to sign in:`からメールアドレスを入力して次に進みます。そして、`Sign in with magic link`を選択します。
![スクリーンショット 2024-09-30 7.11.14.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/159675/72a6b24c-d088-6615-2da0-053577102aca.png)

magic linkとは、サインイン用に発行されるURLのことです。ユーザーはmagic linkのURLを踏むだけでサインインが可能です。magic link発行のリクエストを送信すると、以下のようなメールが届きます。
![スクリーンショット 2024-09-30 7.21.45.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/159675/d7f82257-ad6d-a935-525b-194bf0d38185.png)

裏の仕組みとしては、AWS KMSでハッシュ値を生成してその値をDynamoDBに保存し、ハッシュ値をmagic linkに含めます。ハッシュ値をDynamoDBに保存する際に有効期限もセットで登録しておくことで、一定時間経過するとmagic linkが無効になるようになっています。ユーザーからmagic link経由でのサインインリクエストが来ると、DynamoDBに保存したハッシュ値と比較し、リクエスト内容の検証を行った後認証するかどうかをレスポンスとして返します。

簡単に説明しましたが、magic linkに関する詳細な仕様は[こちら](https://github.com/aws-samples/amazon-cognito-passwordless-auth/blob/main/MAGIC-LINKS.md)を御覧ください。

:::note info
メール送信にはAmazon SESを利用しています。ドメイン登録やID検証など、SESの設定がすでに完了している必要があります。
※ちなみに私は未設定だったのでRoute 53で安ドメインを取ってSESに登録しました。ここの作業ログもいつか記事にしたいと思います。
:::

### パスキーを登録
magic linkでのサインインが成功すると、以下の画面に遷移します。
右上トーストの`Register new authenticator`からパスキー認証に利用するデバイスを登録することができます。
![スクリーンショット 2024-09-30 8.04.15.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/159675/8454addc-c40e-c85d-afba-5c32f63da05d.png)

私の環境（MacBook）ではTouch IDを要求されました。指紋認証で進めていきます。
![スクリーンショット 2024-09-30 8.06.09.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/159675/8afd2127-1ade-b75b-5273-68c9e4e12274.png)

Touch ID認証に成功すると、デバイス名を入力を促されます。`macbook`などとしておきます。
![スクリーンショット 2024-09-30 8.07.26.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/159675/3756dbd6-3a79-da1b-7e0f-4c908edde1e0.png)

これで登録完了です。
![スクリーンショット 2024-09-30 8.10.25.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/159675/5d36a1f3-b5f5-731c-232f-3289cf4bdc67.png)

### パスキーでサインイン
先ほど登録したパスキーでサインインしてみます。一度サインアウトして、今度は`Sign in with face or touch`を選択します。
![スクリーンショット 2024-09-30 7.11.14.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/159675/72a6b24c-d088-6615-2da0-053577102aca.png)

Touch IDを求められます。
![スクリーンショット 2024-09-30 8.15.15.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/159675/c720f7f3-9090-857b-f598-3d7e47e4030f.png)

認証に成功しました。
![スクリーンショット 2024-09-30 8.16.38.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/159675/ea0979e0-59ae-3b96-4f60-370424f201a5.png)
# 実装の解説
### 2. amazon-cognito-passwordless-authの導入
npm, yarn等で`amazon-cognito-passwordless-auth`をインストールします。
```bash
# npmの場合のみ記載します
npm i amazon-cognito-passwordless-auth
```

### 3. ソースコード修正
backend.tsに以下を追記します。
backend.ts全文：https://github.com/tttol/amplify-passwordless-auth/blob/main/amplify/backend.ts
```typescript:backend.ts(一部抜粋)
const userPool = backend.auth.resources.userPool as cdk.aws_cognito.UserPool;
const userPoolClient = backend.auth.resources.userPoolClient as cdk.aws_cognito.UserPoolClient;
const authStack = Stack.of(userPool);

const passwordless = new Passwordless(authStack, "ZenigamePasskeyAuth", {
  userPool,
  userPoolClients: [userPoolClient],
  allowedOrigins: [
    FRONTEND_URL!
  ],
  fido2: {
    allowedRelyingPartyIds: [
      FRONTEND_HOST!
    ],
  },
});

backend.addOutput({
  custom: {
    fido2ApiUrl: passwordless.fido2Api?.url ?? "",
  },
});
```

次に、フロントエンドのコードも修正します。以下をpage.tsx等に追記します。
page.tsx全文：https://github.com/tttol/amplify-passwordless-auth/blob/main/src/app/page.tsx
```typescript:page.tsx(一部抜粋)
  Amplify.configure(outputs);
  Passwordless.configure({
    clientId: outputs.auth.user_pool_client_id,
    cognitoIdpEndpoint: outputs.auth.aws_region,
    fido2: {
      baseUrl: outputs.custom.fido2ApiUrl,
      authenticatorSelection: {
        userVerification: "required",
      },
    },
  });
```

# 仕様の解説
### WebAuthnの仕様
### Amazon Cognitoへの適用
# さいごに 
# 参考

https://github.com/aws-samples/amazon-cognito-passwordless-auth/tree/main

https://docs.amplify.aws/nextjs/build-a-backend/add-aws-services/custom-resources/

https://docs.aws.amazon.com/cognito/latest/developerguide/user-pool-lambda-challenge.html

https://developer.mozilla.org/ja/docs/Web/API/Web_Authentication_API

https://zenn.dev/ncdc/articles/2d7b13c79e31f7

https://zenn.dev/kaibutsu/articles/40b3bfb3261b7f
