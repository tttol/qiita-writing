---
title: DBを使わずローカルで完結するTODO管理アプリ
tags:
  - 'rust'
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに
TODOアプリを自作するというのは多くのエンジニアが通る道だと思うのですが、多分に漏れず自分もその道を通りました。自分が作ったTODOアプリはデータ管理の部分が少し変わっていて、DBを使わずにローカル環境のファイルでデータを管理するという手法をとっています。あまり例のないパターンだと思うので、記事にして紹介します。

# 作ったもの
こんな感じのシンプルなTODOアプリを作りました。アプリ名は**REM**です。
![app.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/159675/26857fb7-6c2d-4195-817c-cdc097e49c1a.png)
ソースコードはこちらです。

https://github.com/tttol/rem


# アーキテクチャ
REM上で作成したTODOタスクは全てJSONファイルとしてローカルマシンに保存されます。例えば「議事録を書く」というタスクを作成したら、以下のようなJSONが内部で生成されます。
```json
{
  "id": "202509200628_de3ab00e863aaba0a19713a471f6d8efbcecc6a39510029bfd70b0a018306685",
  "title": "議事録書く",
  "status": "todo",
  "description": ""
}
```

![スクリーンショット 2025-11-25 6.58.42.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/159675/80628604-0e24-4af2-9bfc-1b9e3f6cfb3c.png)
このJSONファイルはタスクのステータスによって`todo` or `doing` or `done`のディレクトリに保存されます。

```bash
rem
├── doing
│   └── 議事録を書く.json
├── done
│   └── 実装をする.json
└── todo
    └── メール送る.json
```



![スクリーンショット 2025-11-25 6.51.27.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/159675/b86eabdd-f463-4ab4-993a-acbf7f243633.png)

# 技術スタック
## Backend
Rustを採用しました。REMはファイルの読み書きを頻繁に実施するので、ここのIOの処理が高速に行えることを期待してRustを選びました。
Rustでデスクトップアプリケーションを制作することができるTauriというフレームワークがあり、こちらを利用しています。`npm create tauri-app@latest`を実行することでTauri製のアプリケーションの雛形を簡単に作成することができ、FrontendはTypeScriptで、BackendはRustで書けるようになります。

## Frontend
TypeScript(React.js)を採用しました。前述の通りTauriで雛形を作るとデフォルトではTSを利用することになります。TSで困ることは特にないと感じたので、そのままTSで書き進めました。

# 歯痒いところ
本当は完成したアプリケーションをdmgファイルにして配布したり、Homebrewで配信したりなどしたかったのですが、叶いませんでした。というのも、Mac向けのデスクトップアプリケーションを配布するにはApple Developer　Programという開発者向けのプログラムに参加する必要があり、 **参加には99ドル/年の支払いが必要でした。**。（99ドル≒15万円）10万以上の支払いをする気にはなれなかったので、dmgファイルでの配布は諦めて、各自でソースコードをビルドして使ってもらう形をとりました。

もうちょっと料金を安くしてくれたらなあ・・・と個人的には思います。Androidアプリにも似た制度があって、こちらは25ドル/年だそうです。
まあ、今回は個人利用目的のアプリなのでいいんですけどね。
