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

私のサンプルでは[aws-samples/amazon-cognito-passwordless-auth](https://github.com/aws-samples/amazon-cognito-passwordless-auth/tree/main)というAWSが公式に配布しているサンプルをnpm installして利用しています。公式サンプルには[end-to-end example](https://github.com/aws-samples/amazon-cognito-passwordless-auth/tree/main/end-to-end-example)というローカル起動できるサンプルアプリケーションが用意されています。しかし、end-to-end exampleはAmplifyを利用したものではなく、単純なReactアプリケーションとなっています。

私の作ったサンプルは、end-to-end exampleをAmplify上で動作するように実装したものになります。そのため、アプリの挙動自体は私のサンプルもend-to-end exampleも同じものとなっています。

:::note warn
[aws-samples/amazon-cognito-passwordless-auth](https://github.com/aws-samples/amazon-cognito-passwordless-auth/tree/main)はサンプルにしてはかなりしっかり実装されていますが、あくまでもサンプルなので、今後このリポジトリが継続的にメンテナンスされるかどうかはわかりません。
（実際、aws-samplesの中には公開して数年後にpublic archiveになっているリポジトリもあります。）
:::

### 1. AmplifyでWebアプリケーションを作成する
まずはWebアプリケーションを作成します。フレームワークの選択肢は色々ありますが私はNext.jsを採用しました。ここはAmplifyの公式ドキュメントを参考に進めます。
https://docs.amplify.aws/nextjs/start/

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

### 4. 動作確認

# 実装の解説
### amazon-cognito-passwordless-authの導入
### バックエンドのコード修正
### フロントエンドのコード修正
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
