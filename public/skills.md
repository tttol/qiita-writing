---
title: CodexのSkillsとは何で、AGENTS.mdやMCPとはどう違うのか
tags:
  - SKILLS
  - codex
  - Claude
private: false
updated_at: '2025-12-29T07:52:03+09:00'
id: be0f340b058e5153f58b
organization_url_name: null
slide: false
ignorePublish: false
---
CodexにSkillsがやってきましたね。

https://developers.openai.com/codex/skills/

Claude Codeには前からありましたが、ついにCodexにもきました。自分はCodexを使うことが多いのであまりSkillsのことはキャッチアップできてなかったので、この機にちゃんと勉強します。

# Skillsとは
AIエージェントに特定のタスクに対する専門知識を与える機能、です。

Codexの公式ドキュメントにはこう書いてました。

> Agent Skills let you extend Codex with task-specific capabilities. A skill packages instructions, resources, and optional scripts so Codex can perform a specific workflow reliably. 

SKILL.mdというファイルに、以下のフォーマットで記すようです。

```md
---
name: skill-name
description: Description that helps Codex select the skill
metadata:
  short-description: Optional user-facing description
---

Skill instructions for the Codex agent to follow when using this skill.
```

例えばコードレビュー用のskillを作っておいて、プロンプトに「コードレビューをして」と含めるとコードレビュー用のSKILL.mdが読み込まれるという感じです。

# AGENTS.mdと何が違うか
AGENTS.mdに書いたらいいのでは？と思いますよね。自分もそう思いました。

SKILL.mdがAGENTS.mdと違う点は、**SKILL.mdは必要な時にのみLLMにロードされる**という点だそうです。（公式なエビデンスは見つからなかったので「だそうです」という言い方にしています）AGENTS.mdは毎回の呼び出しでロードされるので、巨大なAGENTS.mdだとコンテキストウィンドウが圧迫されてしまいます。しかし、SKILL.mdは毎回はロードされずLLMが必要・不要を判断します。先ほどのコードレビュー用のskillが事前に用意されていたとしたら、ユーザーがコードレビューに関するプロンプトを送信した時のみコードレビュー用のSKILL.mdがロードされます。

では、LLMはSKILL.mdの必要・不要をどう判断しているか？それはSKILL.mdのdescriptionをみて判断しているようです。

ただしここで注意したいのが、**SKILL.mdのdescriptionを長文にしてはいけない**ということです。公式ドキュメントによるとnameは64文字以下、descriptionは1024文字以下という仕様があります。

![スクリーンショット 2025-12-23 5.23.40.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/159675/02d3226a-c271-49a1-8681-3dbff39032be.png)
引用：https://agentskills.io/specification

SKILL.mdはmetadataを含むファイルであり、nameとdescriptionは最小限に留めよとも書いてあります。

> This file includes metadata (name and description, at minimum) and instructions that tell an agent how to perform a specific task.

引用：https://agentskills.io/what-are-skills


# MCPと何が違うか
MCPもSkillsと同様に必要な時に必要なナレッジを外部リソースからとってくるという点は同じですね。ただ、MCPはMCPサーバーから取得するのに対してSkillsは自端末内のMarkdownファイルから取得する点が異なります。

どの記事で読んだか忘れましたが、**MCPは関数でSkillsは手続きだ**、みたいな表現をしている方がいました。

自分はこれが結構しっくりきています。

MCPは確かにget_xxxみたいな名前がついていることもあるくらいで、関数チックな使い方をします。Skillsは手続き的にナレッジへアクセスするとドキュメントにも書いてありました。

> Skills solve this by giving agents access to procedural knowledge and company-, team-, and user-specific context they can load on demand.

引用：https://agentskills.io/home

# まとめ
- SkillsはLLMに特定のタスクに関する専門知識を渡す機能
- SKILL.mdというマークダウンファイルからナレッジを供給する
- SkillsはAGENTS.mdと違って必要な時にのみロードされるので、コンテキストウィンドウの圧迫を抑制可能
- MCPは関数的、Skillsは手続き的


# 参考

https://zenn.dev/aki_think/articles/978556f1652aa6

https://agentskills.io/home

https://zenn.dev/r_kaga/articles/810cc2e8326ca5
