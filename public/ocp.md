---
title: SOLID原則のO：OCPに対する自分の理解
tags:
  - '設計'
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---
# SOLID原則のO
SOLID原則とはシステム設計に関する原則を5つ定め、その頭文字をとってSOLIDと称しているものです。

- S: SRP - 単一責任の原則
- O: OCP - オープン・クローズドの原則
- L: LSP - リスコフの置換原則
- I: ISP - インターフェース分離の原則
- D: DIP - 依存関係逆転の原則

Clean Arichtectureという本でSOLID原則とはなんぞやというところを自分は学んだのですが、どうも座学だけではいまいちピンときてない状態で、ふんわりした理解のままいました。

# 腹落ち
いつも聴いているリファラジというPodcastでOCPに関するエピソードがありました。

https://podcasts.apple.com/us/podcast/103-%E9%96%8B%E6%94%BE%E9%96%89%E9%8E%96%E5%8E%9F%E5%89%87-solid%E3%81%AEo-%E6%8B%A1%E5%BC%B5%E3%81%AB%E9%96%8B%E3%81%8D-%E4%BF%AE%E6%AD%A3%E3%81%AB%E9%96%89%E3%81%98%E3%82%8B-%E3%81%A3%E3%81%A6%E4%BD%95/id1721989211?i=1000739203916

Podcastでは、OCPってSOLIDの中でもちょっと難解な概念だよね〜という導入から始まり、「拡張に開いて修正に閉じる」という概念を具体例を交えて丁寧に説明されていました。これをきいて、自分の中でずっとふんわりとしていたOCPに対する理解がようやく腹落ちした気がしました。

Podcastでは上下左右の矢印キーの入力に応じてそれぞれ挙動を返す関数を例にとっていました。私の解釈をソースコードに起こしてみます。

＜仕様＞
- 関数は↓↑の矢印キーをインプットとして受け取り、方向に応じた挙動を返却する(最初の仕様ではインプットは上下のみ。後の仕様変更で左右のインプットが要求される予定。)

＜ダメな実装＞
OCPに反した実装例です。

```java
public Object onKey(String input) {
    return switch(input) {
        case "←":
            yield LEFT //左方向を表すオブジェクト
        case "↓":
            yield DOWN //下方向を表すオブジェクト
        case "↑":
            yield UP //上方向を表すオブジェクト
        case "→":
            yield RIGHT //右方向を表すオブジェクト
        default:
            yield UNDEFINED
    }
}
```

＜良い実装＞

# Strategyパターン
