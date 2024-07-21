# qiita-writing
Qiita記事執筆用のリポジトリ  
公式：https://github.com/increments/qiita-cli

# How to
## 記事新規作成
 ```bash
 npx qiita new 記事のファイルのベース名
 ```
※mainブランチでPUSHすると自動で公開されてしまうので、下書きのときはブランチを切る。

## 記事プレビュー
```bash
npx qiita preview
```

## 記事公開
```bash
npx qiita publish 記事のファイルのベース名
```
or　mainブランチにマージ

## 画像UP
localhostのプレビューページからアップロード用のリンクに飛ぶ。