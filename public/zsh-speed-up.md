---
title: zshの起動時間を短縮する
tags:
  - 'zsh'
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---
皆さんのシェルの起動時間は何msでしょうか？
私はClaude CodeやCodex CLIを使う頻繁に使うようになってから必然的にターミナルの起動回数が増え、次第にシェルの起動時間が気になるようになりました。そんな折に以下の記事を読み、自分も起動時間の短縮を試してみました。

https://zenn.dev/i9wa4/articles/2026-01-01-zsh-startup-optimization-zinit

# 先に結果
色々試した結果、起動時間が **「99.9 ms -> 42.9 ms」** になりました。

BEFORE
```bash
❯ hyperfine --warmup 3 --runs 10 'zsh -i -c exit'
Benchmark 1: zsh -i -c exit
  Time (mean ± σ):      99.9 ms ±   0.6 ms    [User: 58.1 ms, System: 41.9 ms]
  Range (min … max):    99.1 ms … 101.3 ms    10 runs
 ```

AFTER
```bash
❯ hyperfine --warmup 3 --runs 10 'zsh -i -c exit'
Benchmark 1: zsh -i -c exit
  Time (mean ± σ):      42.9 ms ±   0.4 ms    [User: 26.5 ms, System: 15.6 ms]
  Range (min … max):    42.3 ms …  43.6 ms    10 runs
```

なお環境は以下の通りです。
- PC: M4 MacBook Air
- shell: zsh(wezterm)

# ベンチマークとボトルネック
### ベンチマーク
まずは現在のシェルの起動時間を計測しましょう。色々やり方はありますが、私は`hyperfine`というRust製のツールを使いました。

https://github.com/sharkdp/hyperfine

以下のように実行することでzshの起動時間を計測できます。
```bash
# --warmup 3: 最初に実行した3回分の結果を計測対象外とする（キャッシュの影響を考慮）
# --runs 10: 10回実行する
# 'zsh -i -c exit': zshを起動してすぐ終了する
❯ hyperfine --warmup 3 --runs 10 'zsh -i -c exit'
Benchmark 1: zsh -i -c exit
  Time (mean ± σ):      99.9 ms ±   0.6 ms    [User: 58.1 ms, System: 41.9 ms]
  Range (min … max):    99.1 ms … 101.3 ms    10 runs
  ```

`--warmup`で3回＋`--runs`で10回なので、合計13回のzsh起動＆終了が実施されます。
自分の環境では99ms要していました。

### ボトルネックの分析
今の起動時間がわかったので、次は速度のボトルネックを分析します。
.zshrcの先頭に`zmodload zsh/zprof`を、.zshrcの末尾に`zprof`を記載することでzsh起動時に関数ごとの所要時間が表示されるようになります。

```sh:.zshrc
zmodload zsh/zprof

~~~

zprof
```

```bash
num  calls                time                       self            name
-----------------------------------------------------------------------------------
 1)    2          13.72     6.86   58.95%     13.72     6.86   58.95%  compaudit
 2)    1          19.77    19.77   84.96%      6.05     6.05   26.01%  compinit
 3)    5           1.68     0.34    7.23%      0.67     0.13    2.90%  zinit
 4)    1           0.66     0.66    2.85%      0.66     0.66    2.85%  is-at-least
(略)
```
この結果を元にLLMと壁打ちしながらボトルネックを分析していきます。
自分の環境ではプラグインマネージャーと特定のプラグインが読み込みに時間を要していることがわかったので、この2点を対処していきます。

# 対応1：プラグインマネージャーの移行
今までOh My Zsh!(以下omzと呼ぶ)と言うzshのプラグインマネージャーを使っていたのですが、Zinitに乗り換えました。自分の環境ではomzがzsh起動時に過剰にプラグインのロードを行なっていたようです。後述の遅延読み込みがomzではできないこともあったので、いっそプラグインマネージャー自体を移行してやるか！という気持ちになった次第です。

で、移行先のZinitですが、起動時間の短さを売りにしたプラグインマネージャーのようです。

https://github.com/zdharma-continuum/zinit

![zinit.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/159675/ef9bb265-2bf0-4383-bb98-2178236415e0.png)
omzで管理していたプラグインは全てzinitにも存在していたので、サクッと移行しました。すると、起動時間が54 msまで短縮できました。すごい。

https://x.com/tttol777/status/2007933168662585619?s=20

# 対応2：遅延読み込み
ボトルネックの2つ目として、読み込みに時間を要しているプラグインを遅延読み込みするようにします。いわゆるlazy loadです。Zinitのコマンドを用いて遅延読み込みを実現します。

```bash
# zsh-users/zsh-autosuggestionsというプラグインを遅延読み込みさせるケース
zinit ice wait lucid atload'_zsh_autosuggest_start'
zinit light zsh-users/zsh-autosuggestions
```

- ice
    - 次回のプラグイン読み込み時に有効となる設定を指定します
    - 2行目の`zinit light zsh-users/zsh-autosuggestions`に対してwait, lucid, atload'_zsh_autosuggest_start'の3つを適用します。
- wait
    - プラグイン読み込みを遅延させます
    - zshの起動後に`_zsh_autosuggestion_start`が読み込まれます
- lucid
    - プラグイン読み込み時に表示されるメッセージを非表示にします
    - 遅延読み込みされたプラグインが発するメッセージを非表示にすることが目的です
- atload
    - プラグイン読み込み後に実行するコマンドを指定します
- light
    - レポート機能なしでプラグインを読み込みます
    - 通常の`zinit load`よりも高速に実行されます

これで、zsh起動後にプラグインの読み込みが開始されるようになります。ベンチマークで使ったhyperfineコマンドはzshの起動までを計測しているので、プラグインの遅延読み込み分の時間はカウントされません。

なお、iceによる遅延読み込みはプラグインインストール以外の処理にも使えます。例えばこんな感じで複数行に渡るコマンド実行を遅延させる場合は以下のように書きます。
```bash
zinit ice wait lucid atload'
autoload -Uz compinit && compinit -C
if [ -e /usr/local/share/zsh-completions ]; then
    fpath=(/usr/local/share/zsh-completions $fpath)
fi
zstyle ":completion:*" matcher-list "m:{a-z}={A-Z}"
zstyle ":completion:*" ignore-parents parent pwd ..
zstyle ":completion:*:default" menu select=1
zstyle ":completion:*:cd:*" ignore-parents parent pwd
'
zinit light zdharma-continuum/null
```

zdharma-continuum/nullは空のプラグインです。コマンドを遅延実行させるためのhookを担っており、zdharma-continuum/nullが読み込まれた後にコマンドが実行されるようにしています。

リポジトリを見てもわかる通り、本当に空っぽのプラグインです。

https://github.com/zdharma-continuum/null

# 対応3：Homebrewの環境変数
これは小ネタに近いんですが、Homebrew関連の環境変数の扱いを修正しました。
```diff
- eval "$(/opt/homebrew/bin/brew shellenv)"
+ export HOMEBREW_PREFIX="/opt/homebrew"
+ export HOMEBREW_CELLAR="/opt/homebrew/Cellar"
+ export HOMEBREW_REPOSITORY="/opt/homebrew"
+ export PATH="/opt/homebrew/bin:/opt/homebrew/sbin${PATH+:$PATH}"
+ export MANPATH="/opt/homebrew/share/man${MANPATH+:$MANPATH}:"
+ export INFOPATH="/opt/homebrew/share/info:${INFOPATH:-}"
```

`eval "$(/opt/homebrew/bin/brew shellenv)`は単に環境変数のexportをしているだけのようで、evalを毎回実行するよりも直にexportした方が早いのでこうしました。

# さいごに
Zinitに移行したことが一番効果が大きかったです。zshに限らず、プラグインマネージャーは定期的に見直した方が良いですね。

最後に私のdotfilesを共有して終わりにしたいと思います。

https://github.com/tttol/dotfiles
