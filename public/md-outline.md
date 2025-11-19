---
title: NeovimでMarkdownファイルのアウトラインを表示する
tags:
  - 'neovim'
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに
最近エディタをVSCodeからNeovimに乗り換えました。最初はvim特有のハードルの高さに苦戦しましたが、プラグインや設定を自分好みにすることで現在はそこそこ快適な環境で作業ができるようになりました。苦戦したものの一つとして、Markdownファイルのアウトライン機能があります。

例えばqiitaだったら、記事の右側に見出しをリスト化したアウトラインが表示されますよね。
![スクリーンショット 2025-11-15 5.19.46.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/159675/ecad8412-d6ab-48c3-8f75-63579e9a0798.png)
NeovimでMarkdownファイルを閲覧・編集する時にもこれが表示されると嬉しいなあと思うわけです。そんな欲求を叶えてくれるプラグインを紹介します。

# Makrdownファイルのアウトラインを表示するプラグイン
`md-outline.nvim`というNeovimのプラグインです。

https://github.com/tttol/md-outline.nvim

デモはこんな感じです。
![demo5.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/159675/a80210f2-1d8d-4f35-8e84-a67bb49896b3.gif)
真ん中部分のエディタから入力した見出しの内容が、右側のウィンドウでアウトラインとなってリアルタイムに出力されているのがわかると思います。また、現在いるカーソルに合わせてアウトライン側がハイライトされるようになっています。

# インストール
lazy.nvimを使っている場合、以下でインストールが可能です。
```lua
return {
  'tttol/md-outline.nvim',
}
```

# 主な機能
## アウトライン自動表示
インストール時のオプションには一つだけ`auto-open`というプロパティがあり、こちらをtrue/falseで指定することができます。デフォルト値はtrueです。
```lua
return {
  'tttol/md-outline.nvim',
  config = function()
    require('md-outline').setup({
      auto_open = false -- default: true
    })
  end
}
```

`auto-open`がtrueの場合、.mdファイルをNeovimで開いたら自動的にアウトラインのウィンドウが右側に表示されます。falseの場合は自動表示が行われませんので、別途`:MdoOpen`を手動で実行することでアウトラインウィンドウを開くことができます。
## `:MdoOpen`, `:MdoClose`コマンド
先ほども触れましたが、手動でアウトラインウィンドウの開閉を行う場合は以下のコマンドを利用します。
- `:MdoOpen`: アウトラインウィンドウを開く
- `:MdoClose`: アウトラインウィンドウを閉じる
ただし、現在アクティブになっているカレントバッファがMarkdownファイルではない場合、アウトラインウィンドウは自動でクローズされます。なので、`:MdoClose`を使うシーンは実はあまりないと思います。

# さいごに
薄々気づかれているかもしれませんが、このプラグインは私が自作したプラグインです！
いいプラグインがないかなあと探していたんですが、肌に合うものがなかったので「ないなら自分で作ってしまえ！」の精神でえいやで作ってみました。なお、プラグイン制作は初めてのことなので、実装がイマイチだったりお作法に則ってない部分がもしありましたら、IssueやPRでこっそりご指摘ください。
