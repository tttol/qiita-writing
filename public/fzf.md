---
title: fzfでターミナル上でのファイル検索を快適にする
tags:
  - 'CLI'
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに
最近Neovimを使うようになってターミナル上で生活する時間が増えてきまして、CUIの便利ツールをよく探すようになりました。本記事ではターミナル上でのファイル検索を快適にしてくれるfzfというツールについて紹介します。

# fzf ひいてはFuzzy Finderとはなんぞや
fzfはFuzzy Finderをもじったものです（だと思ってます）。
リポジトリはこちら

https://github.com/junegunn/fzf

Fuzzy Finderというのは日本語でいうところのあいまい検索とほぼ同義です。例えば、qiita_article_getter.pyという名前のファイルを検索したい時、前方一致的に`qiita_article_ge`と打つと確実にHITしますよね。これが、あいまい検索が有効なサービスの場合、`qiitagetter`でもHITすることになります。ファイル名を前方一致的に一言一句違わずに入力せずとも、ファイル名の断片的な情報で検索が可能になるということです。

fzfをインストールすると、ターミナル上でのファイル検索時にこうしたあいまい検索が可能になります。

以下のGIFは、`labeltest`という文字列から`LabeledSum.test.tsx`というファイルを検索する例です。
![fzf-demo.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/159675/544bc241-7279-4b2c-b5a7-936703510b83.gif)
# fzfで検索した結果をcd・nvimコマンドに渡す
fzfはそれ単体でも強力なコマンドですが、他のコマンドに渡すことでよりその効果を発揮します。

例えば`nvim $(fzf)`を実行すると、fzfで検索したファイルをNeovimで開くことができます。
![nvimfzf-demo.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/159675/9902d537-f019-4011-a123-57918e60c650.gif)

自分はこれを`nvf()`という関数にして.zshrcに記載しています。
```bash
nvf() {
  local dir
  cd
  dir=$(fzf)
  [ -n "$dir" ] && nvim "$dir"
}
```


上記のnvfは検索対象をファイルにとしていますが、ディレクトリを検索対象にすることもできます。自分は以下のnvdという関数を別途作って（Claudeに作ってもらって）、同じく.zshrcに記載してます。
```bash
nvd() {
  local dir
  
  # ホームディレクトリに移動。ホームディレクトリを起点にして検索を開始したいため。
  cd
  # fdはfindコマンドの代替版。fdで検索した結果をfzfに渡す。
  dir=$(fd --type d --hidden --exclude .git | fzf \
    --preview 'eza --tree --level=2 --icons --git {} 2>/dev/null || tree -L 2 {} 2>/dev/null || ls -la {}' \
    --preview-window=right:60%)
  # 見つけたディレクトリまでcdで移動&Neovimで開く
  [ -n "$dir" ] && cd "$dir" && nvim .
}
```
![nvd-demo.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/159675/9eb089e9-129e-4dba-80ed-baa5a49e20b8.gif)
# さいごに
今回紹介した例はfzfのほんの一部です。もっと高度でテクニカルな関数を書いて、fzfをより使いこなしていこうと思います。
