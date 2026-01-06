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
色々試した結果、起動時間が「99.9 ms -> 42.9 ms」になりました。

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
omzで管理していたプラグインは全てzinitにも存在していたので、サクッと移行しました。すると、起動時間が54 msまで短縮できました（すごい）

https://x.com/tttol777/status/2007933168662585619?s=20

# 対応2：遅延読み込み

