---
title: 'ls, cdを快適かつ高速に行うためのTIPS'
tags:
  - 'CLI'
  - 'terminal'
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに
最近普段使いのエディタをVSCodeからNeovimに移行したので、ターミナル上で生活する時間が増えました。
lsやcdでディレクトリ間を移動することが多くなったわけですが、実はlsとcdにはRust製の代替ツールが存在します。自分も最近その存在を知ったのですが、もっと早く知っておけばよかった！と思えるほど便利なツールだったので、紹介したいと思います。

# lsの代替：eza
lsの代替ツールとしてezaというものがあります。

https://github.com/eza-community/eza

lsと何が違うのか？同じディレクトリに対してlsとezaをそれぞれ実行した結果を見比べてみましょう。

![スクリーンショット 2025-12-05 14.54.56.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/159675/ca2ccbec-935c-4d92-b8b7-637495119d40.png)

上がls、下がezaです。ezaの方が若干文字色がカラフルになりますね。
オプションなしだとあまり違いがないので、普段使いレベルまでオプションをつけた時の比較をしてみましょう。

![スクリーンショット 2025-12-05 14.57.37.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/159675/94f317f8-c50b-45a6-844d-cbcc36dd9724.png)

どうでしょうか。結構見栄えが変わりましたね。

- 文字色がカラフルになった
- ディレクトリ・ファイルのアイコンが表示されるようになった
    - ファイルは拡張子ごとにアイコンが変化する
- `.`, `..`などが非表示になった（自分はこれ要らないと思っているのでありがたい）

また、-Lオプションで任意の階層まで深掘りすることが可能です。

![スクリーンショット 2025-12-05 15.05.37.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/159675/23b39c5a-e887-4483-9766-eab8d0aea5e6.png)
※長くなるので全部は映せませんでした


ezaはオプションを色々つけると長くなるので、自分は以下のようなエイリアスを.zshrcに仕込んでいます。
```bash
# アイコンあり・隠しファイル表示・一覧表示
alias ll='eza --icons -al'

# アイコンあり・隠しファイル表示・一覧表示・2階層掘る
alias ll2='eza --icons -al -T -L 2'
```

lsはもう使わないので、思い切って`ll`にezaを割り当てました。

# cdの代替：zoxide
続いてcdの代替、zoxideの紹介です。

https://github.com/ajeetdsouza/zoxide

zoxideは`z`と一文字打つだけでcdと同じ動きをしてくれます。
zoxideの強力な機能は、一度移動したことのあるディレクトリを記憶しておいて、2回目以降に同じ場所移動する時に省略したパス名で移動できる点です。

ちょっと文章では伝わりづらいので、gifで示します。

![demo-z.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/159675/c9a71359-b46b-4eb9-8777-25f40e9c810c.gif)

1回目の移動では`z .config/nvim/lua/lsp`で移動していたところを、2回目の移動では`z lsp`で同じディレクトリに移動できています！
```bash
# 1回目
z .config/nvim/lua/lsp

# 2回目
z lsp
```


同名のディレクトリ名がある場合はspace+tabで候補を一覧から選択できるようになっています。詳しくは以下のGIFで説明されています。

https://github.com/ajeetdsouza/zoxide/blob/main/contrib/tutorial.webp

## 余談
自分はoh my zsh!の補完機能もプラグインとして別途インストールしています。

https://github.com/zsh-users/zsh-completions

この補完機能と組み合わせることで、3回目以降の移動ではさらにタイピング数を減らすことが可能です。
例えば先ほどの`z .config/nvim/lua/lsp`の例だと、以下のように`z l`とタイプするだけで`z lsp`が補完候補として表示されます。

![スクリーンショット 2025-12-05 15.23.30.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/159675/caf13900-607a-4f5c-881a-7633b0a10a19.png)

```bash
# 1回目
z .config/nvim/lua/lsp

# 2回目
z lsp

# 3回目(途中で補完される)
z l
```

# 他にも
catの代替であるbatだったり、grepの拡張版であるripgrepだったりと、UNIXコマンドの多くは代替ツールが多く存在します。有名なものは代替の方が高機能かつ高速であることが多いので、積極的に取り入れていきたいところです。

