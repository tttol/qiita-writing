---
title: Storage Browser for Amazon S3をさわってみた！
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

:::note warn
2024/9/7現在、Storage Browserはアルファリリース状態です。今後GAされるまでに変更が加えられる可能性があります。
本記事は執筆時点でのライブラリを用いて実装したものになりますので、ご注意・ご了承ください。
:::
# Storage Browserの概要
# 導入方法
※2024/9/6時点 アルファリリースでの方法
### Next.js × AWS Amplifyのプロジェクトを新規作成
まず下記コマンドでNext.jsの新規プロジェクトを作成します。
```bash
npx create-next-app@latest
```

次に下記コマンドでAmplifyを導入します。
```bash
cd my-app
npm create amplify@latest
npm add --save-dev @aws-amplify/backend@latest @aws-amplify/backend-cli@latest typescript
```

ここまでで、まずNext.js × AmplifyのWebアプリケーションが出来上がります。
CLIから`npm run dev`を実行して http://localhost:3000 にブラウザからアクセスするとアプリケーションを表示できるはずです。

### StorageBrowserの導入
GitHubのIssueを参考に、StorageBrowserのライブラリをインストールします。
```bash
npm i --save @aws-amplify/ui-react-storage@storage-browser aws-amplify@storage-browser
```

### バックエンドのコードを書く
backend.ts, resource.ts(auth & storage)を書いていきます。
これらは全てAmplifyで利用する設定ファイルです。

```typescript:backend.ts
import { defineBackend } from '@aws-amplify/backend';
import { auth } from './auth/resource';
import { data } from './data/resource';
import { storage } from './storage/resource';

/**
 * @see https://docs.amplify.aws/react/build-a-backend/ to add storage, functions, and more
 */
defineBackend({
  auth,
  storage
});
```

```typescript:auth/resource.ts
import { defineAuth } from '@aws-amplify/backend';

/**
 * Define and configure your auth resource
 * @see https://docs.amplify.aws/gen2/build-a-backend/auth
 */
export const auth = defineAuth({
  loginWith: {
    email: true,
  },
});
```

```typescript:storage/resource.ts
import { defineStorage } from "@aws-amplify/backend";

export const storage = defineStorage({
  name: 'storage-browser-example',
  access: (allow) => ({
    'public/*': [
      allow.guest.to(['read']),
      allow.authenticated.to(['read', 'write', 'delete']),
    ],
    'protected/{entity_id}/*': [
      allow.authenticated.to(['read']),
      allow.entity('identity').to(['read', 'write', 'delete'])
    ],
    'private/{entity_id}/*': [
      allow.entity('identity').to(['read', 'write', 'delete'])
    ]
  })
});
```

### フロントエンドのコードを書く
フロントエンドもIssueを参考に書いていきます。

```typescript:DefaultStorageBrowser.tsx
"use client"
import { StorageBrowser } from '@aws-amplify/ui-react-storage';
import "@aws-amplify/ui-react-storage/styles.css";

// these should match access patterns defined in amplify/storage/resource.ts
const defaultPrefixes = [
  'public/',
  (identityId: string) => `protected/${identityId}/`,
  (identityId: string) => `private/${identityId}/`,
];

export default function DefaultStorageBrowser() {
  return (
    <StorageBrowser defaultPrefixes={defaultPrefixes} />
  )
}
```
`defaultPrefixes`は`storage/resource.ts`の設定内容と一致させる必要があります。

このコンポーネントをpage.tsxでインポートしましょう。
```typescript:page.tsx
"use client"
import outputs from "@/../amplify_outputs.json";
import { Amplify } from "aws-amplify";
import DefaultStorageBrowser from "../components/DefaultStorageBrowser";

Amplify.configure(outputs);

export default function Home() {
    return (
        <DefaultStorageBrowser />
    );
}
```

`npm run dev`コマンドでアプリを起動しブラウザからアクセスすると、以下のようにS3バケットの一覧がGUIで表示されます。
![スクリーンショット 2024-09-07 20.30.51.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/159675/1ccfb337-17ca-bff9-c480-95abce676cce.png)

![スクリーンショット 2024-09-07 20.32.50.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/159675/153da825-97df-5322-6126-af5087ff359c.png)

:::note info
私の実装に何か不備があるのか、CSSが一部おかしいようで、UIが崩れてる箇所があります。
（パンくずリストなど）
アルファリリースゆえのものなのかどうかは判断がつきませんでした。
:::
# できること
Storage Browserでできること。
### S3バケット内のファイルを自身のWebアプリケーションから直感的に操作可能
### Using custom UI

# さいごに 
