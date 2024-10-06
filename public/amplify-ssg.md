---
title: Next.js(SSG)なアプリをAWS Amplifyにデプロイする際の注意点まとめ2024
tags:
  - 'Amplfy'
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに
Next.jsのSSG(Static Site Generation)で作成したWebアプリケーションをAWS Amplifyにデプロイする際、いくつかハマりポイントがありました。忘れないよう記事にしておこうと思います。

# SSGとは
Static Site Generationの略です。Next.jsで実装したアプリケーションを`next build`でビルドする際に静的なコンテンツ(HTML, CSS)に落とし込み、それらをCDNでキャッシュして配信することで高速なレスポンスを実現できます。静的コンテンツと聞くと利用シーンが限定されそうに聞こえますが、外部へのデータフェッチを行うアプリケーションであってもSSGを利用することができます。（ソースコード側で少し工夫は必要）

# ハマりポイント
### next.config.jsの設定
Next.jsのレンダリング方式は、デフォルトではSSRが適用されています。これをSSGにするためにはnext.config.jsに追記が必要です。
```diff
/** @type {import('next').NextConfig} */
const nextConfig = {
+    output: "export",
+    images: { unoptimized: true },
};

export default nextConfig;
```

まず`output: "export",`ですが、これを設定した状態で`next build`でビルドを行うと、outというディレクトリに成果物が生成されます。outには静的アセットのみが出力されており、これらをWebサーバーに置けばそれだけで配信が可能になります。next.jsのリポジトリを確認したところ、以下の記述がありました。
```
  /**
   * The type of build output.
   * - `undefined`: The default build output, `.next` directory, that works with production mode `next start` or a hosting provider like Vercel
   * - `'standalone'`: A standalone build output, `.next/standalone` directory, that only includes necessary files/dependencies. Useful for self-hosting in a Docker container.
   * - `'export'`: An exported build output, `out` directory, that only includes static HTML/CSS/JS. Useful for self-hosting without a Node.js server.
   * @see [Output File Tracing](https://nextjs.org/docs/advanced-features/output-file-tracing)
   * @see [Static HTML Export](https://nextjs.org/docs/advanced-features/static-html-export)
   */
```
> https://github.com/vercel/next.js/blob/canary/packages/next/src/server/config-shared.ts#L884

次に`images: { unoptimized: true },`ですが、これは「画像の最適化を行わない」という設定になります。
画像の最適化、というのは`next/image`の`<Image>`タグを用いた最適化のことを指しています。`<Image>`タグはHTMLの`<img>`を拡張したもので、デバイスごとに画像サイズを最適なものに変更したり、画像読み込みのタイミングを調整したりと、裏で色々してくれます。
https://nextjs.org/docs/pages/building-your-application/optimizing/images
https://nextjs.org/docs/pages/api-reference/components/image#unoptimized

しかし、この最適化処理はサーバーで行われます。SSGは`next build`実行時に生成された静的アセットを配信するため、ページ表示時にサーバーで行われる処理を実施することはできない？ようです。SSGなページで`<Image>`を使うと、画像が読み込まれませんでした。loaderを変更することで解消できるとの記事も確認しましたが、今回はそこまでするモチベーションはなかったので、単純に最適化をしない設定にすることで回避しました。
https://zenn.dev/kisukeyas/scraps/6fd8bf7e63e4a3
https://ebisu.com/note/next-image-ssg/

### Amplifyの設定
# さいごに
