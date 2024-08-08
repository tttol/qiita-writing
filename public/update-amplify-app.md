---
title: 変化の速いフロントエンドの世界で半年間アプリをほったらかした男の作業ログ
tags:
  - 'Amplify'
  - 'AWS'
  - 'Next.js'
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに
フロントエンド界隈はアップデートが早く、ライブラリのバージョンが目まぐるしくアップデートされていきます。
キャッチアップを怠るとすぐに置いてけぼりになってしまいます。
この記事は「Next.js × AWS Amplifyで作成した個人開発アプリを半年間塩漬けにした結果、キャッチアップにこれだけ苦労したよ！」という実体験に基づく作業ログです。

# 状況
2024年2月頃にAmplify Gen2を使ったアプリを作り（個人開発）、特に改修なしで数ヶ月稼働させていたのですが、最近になって新機能が欲しくなったのでソース改修ことにしました。
改修を始めたのは8月なので、およそ半年ぶりに改修をすることになります。

まずは[node-check-update](https://www.npmjs.com/package/npm-check-updates)を使って、ライブラリの更新を行いました。
バージョンの遷移は下表のとおりです。

|                         | 更新前           | 更新後           |
|-----------------------------------|------------------|------------------|
| @aws-amplify/ui-react             | ^6.1.5           | ^6.1.14          |
| aws-amplify                       | ^6.0.19          | ^6.4.3           |
| next                              | ^14.2.3          | ^14.2.5          |
| react                             | ^18.2.0          | ^18.3.1          |
| react-dom                         | ^18.2.0          | ^18.3.1          |
| tailwindcss                       | ^3.4.1           | ^3.4.7           |
| @aws-amplify/backend              | ^0.13.0-beta.5   | ^1.0.4           |
| @aws-amplify/backend-cli          | ^0.12.0-beta.5   | ^1.2.1           |
| @testing-library/react            | ^15.0.7          | ^16.0.0          |
| @types/node                       | ^20              | ^22              |
| vitejs/plugin-react               | ^4.2.1           | ^4.3.1           |
| aws-cdk                           | ^2.132.0         | ^2.150.0         |
| aws-cdk-lib                       | ^2.132.0         | ^2.150.0         |
| esbuild                           | ^0.20.1          | ^0.23.0          |
| eslint                            | ^8               | ^8.0.0           |
| eslint-config-next                | ^14.1.3          | ^14.2.5          |
| jsdom                             | ^24.0.0          | ^24.1.1          |
| tsx                               | ^4.7.1           | ^4.16.3          |
| typescript                        | ^5.4.2           | ^5.5.4           |
| vitest                            | ^1.6.0           | ^2.0.5           |

`@aws-amplify/backend`と`@aws-amplify/backend-cli`がbetaから1.X系にアップデートになっており、ここの影響がでかそうな予感がします。
実際にアプリをビルドしてみると色々怒られました。以下に怒られた内容と対処法を記載していきます。

# `npx amplify`->`npx ampx`に修正
### 内容
amplify.ymlに記載しているデプロイコマンドで`npx amplify ~~~`となっている箇所でエラーが出ました。
どうやら`npx ampx ~~~`に変わったようでした。

```sh:エラーメッセージ
InvalidCommandError: The Amplify Gen 2 CLI has been renamed to ampx
```
### 修正方法
- 修正前：`npx amplify pipeline-deploy --branch $AWS_BRANCH --app-id $AWS_APP_ID`
- 修正後：`npx ampx pipeline-deploy --branch $AWS_BRANCH --app-id $AWS_APP_ID`

### 修正されたと思われる時期
どうやら[Amplify Gen2がGAされたタイミング](https://aws.amazon.com/jp/about-aws/whats-new/2024/05/aws-amplify-gen-2-available/)でamplify -> ampxにリネームされたようです。

以下の`ampx@0.2.0`リリースのコミットで追加されてそうです。

https://github.com/aws-amplify/amplify-backend/releases/tag/ampx%400.2.0

amplify -> ampxにリネームしよう！というIssueややりとりは見つけられませんでした。

# DynamoDBから取得したデータの型を修正
### 内容
DynamoDBのTodoテーブルからscanしたデータをTypeScriptで受け取る際、型を`Schema["Todo"][]`としていたのですが、ここが型エラーになりました。
### 修正方法
- 修正前：`Schema["Todo"][]`
- 修正後：`Schema["Todo"]["type"][]`
※`Todo`はDynamoDBのテーブル名

### 修正されたと思われる時期
Amplifyの公式ドキュメントでは、該当箇所が5月頭に変更されていました。
これもGen2 GAのタイミングかなと思われます。

https://github.com/aws-amplify/docs/pull/7414

ソースコードレベルでの変更時期はわかりませんでした。

# amplifyconfiguration.json -> amplify_outputs.jsonに修正 
### 内容
```typescript:ConfigureAmplifyClientSide.tsx
"use client";

import config from "@/amplifyconfiguration.json";
import { Amplify } from "aws-amplify";

Amplify.configure(config, { ssr: true });

export default function ConfigureAmplifyClientSide() {
  return null;
}
```
修正前のソースコードでは、Cognitoの認証処理をクライアントサイドで呼び出すために、上記のConfigureAmplifyClientSideコンポーネントをlayout.tsxで読み込んでいました。
amplifyconfiguration.jsonはAmplifyが自動生成するファイルなのですがある時からjsonが自動生成されなくなり、`import config from "@/amplifyconfiguration.json";`がエラーになりました。

### 修正方法
amplifyconfiguration.jsonの代わりにamplify_outputs.jsonが生成されるようになったので、このjsonを下記のように記述して読み込む必要がありました。
```typescript
import { Amplify } from "aws-amplify";
import config from "../amplify_outputs.json";

Amplify.configure(config);
```

### 修正されたと思われる時期
こちらもGen2 GAのタイミングで公式ドキュメントに修正が入ってました。

https://github.com/aws-amplify/docs/pull/7438


# 参考

https://dev.classmethod.jp/articles/amplify-gen2-ga-cli/