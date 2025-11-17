---
title: Docker Desktop以外のランタイムから起動したコンテナをDocker CLIから操作できる理由
tags:
  - 'docker'
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---
# ある日の疑問
自分は今までローカル環境ではDocker Desktopを使ってコンテナを扱うことが多かったのですが、最近チームが変わってRancher Desktopを利用するようになりました。この時はDockerに対する理解度が低かったので、「あ〜RancherだからDockerコマンドは使えないなあ」とか思っていました。Rancherで起動したコンテナに対していつもの手癖で`docker ps`を実行すると、なんとコンテナの状態が確認できるではありませんか。

RancherとDockerは全く別物のソフトウェアだと思っていたので「Rancherで起動したコンテナがなんでDocker CLIから確認できるんだ？？？」と軽く混乱しました。気になって色々調べてみると、自分はコンテナランタイムのことを何ひとつわかっていなかったので、自分の調べた結果を記事にすることにしました。

# コンテナが起動するまでに起きていること
まず、冒頭の疑問に対する回答です。

Q：なぜRancher Desktop（=Docker Desktop以外のランタイム）から起動したコンテナをDocker CLIで操作できるのか？
A：Rancher DesktopはDocker互換のAPIエンドポイントを提供しており、Docker CLIはそのエンドポイントを経由してコンテナ情報を取得しているから。

この回答の意味を理解するためには、コンテナが起動するまでに内部でどんなことが起きているかを知る必要があります。例えば、`docker ps`を実行してからコンテナの情報を受け取るまでにどんなことが起きているのでしょうか。

**Docker Desktopの場合**
```
docker ps
↓
/var/run/doker.sockのソケットにアクセス
↓
dockerd
↓
containerd
↓
runc
↓
コンテナ本体へアクセス
```

**Rancher Desktopの場合**
```
docker ps
↓
~/.rd/docker.sockのソケットにアクセス
↓
Docker互換のAPIエンドポイント
（dockerdの代わり）
↓
containerd
↓
runc
↓
コンテナ本体へアクセス
```

# Docker CLI
`dockerp ps`, `docker run`などのCLIコマンドのことです。
# socket
`/var/run/docker.sock`はCLIコマンドとデーモン間の通信をつなぐソケットです。docker.sockはDocker Desktopを起動することで自動的にソケットもオープンになります。

余談ですが、`Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?`というエラーに遭遇したことのある人は多いのではないでしょうか？これはDocker CLIがソケットにアクセスできない場合に発生するエラーで、/var/run/docker.sockのソケットにアクセスできない旨のメッセージが表示されています。

```
$ docker ps
Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?

```
※これが発生する時は大体Docker Desktopが起動できていないことが多い印象

なお、Docker CLIがどのソケットを接続先とするかはDOCKER_HOSTという環境変数で指定することができます。`echo $DOCKER_HOST`で確認も可能です。もし環境変数を指定していない場合は`docker context ls`からも確認可能です。

# dockerd, containerd
ソケットにアクセスできたら、次はデーモンを探します。

Docker Desktopではdockerdというデーモンを経由してcontainerdにアクセスします。二つのデーモンを利用する理由は、それぞれのデーモンによって役割が異なるためです。

- dockerd
    - Dockerfileの解析
    - Docker Composeとの連携
    - Docker Networkの構築
    - など・・
- containerd
    - コンテナライフサイクルの管理
    - イメージのpull/push
    - kubernetesサポート
    - など・・・

dockerdはDocker固有の機能に特化し、containerdはコンテナ自体の管理に特化しています。技術的にはDocker CLIからソケットを通してcontainerdに直接アクセスすることも可能ですが、dockerdが提供する豊富な機能を利用しないのは勿体無いので、dockerdを経由する方式が採用されているようです。

また、歴史的経緯からデーモンがdockerdとcontainerdの2つに分離した事情もあります。元々、初期のDockerはdockerdだけの巨大なモノリスとして開発・運用されていた経緯があり、一枚岩の辛さを解消するために一部の機能をcontainerdに切り出しました。その名残りで、現在も2つのデーモンが存在する形となっているようです。

https://www.codingknownsense.com/clear-your-concept/docker-a-comprehensive-guide-to-containerization/docker-history/

# Docker互換API
# runc, cgroups
# まとめ
