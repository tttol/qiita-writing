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
それぞれのレイヤーを説明していきます。

# Docker CLI
`docker ps`, `docker run`などのCLIコマンドのことです。

# socket
`/var/run/docker.sock`はCLIコマンドとデーモン間の通信をつなぐソケットです。docker.sockはdockerdが起動するため、Docker Desktopを起動することでdockerdが起動し自動的にソケットもオープンになります。

余談ですが、`Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?`というエラーに遭遇したことのある人は多いのではないでしょうか？これはDocker CLIがソケットにアクセスできない場合に発生するエラーで、`/var/run/docker.sock`のソケットにアクセスできない旨のメッセージが表示されています。

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
    - kubernetesサポート(CRI: Container Runtime Interface)
    - など・・・

dockerdはDocker固有の機能に特化し、containerdはコンテナ自体の管理に特化しています。技術的にはDocker CLIからソケットを通してcontainerdに直接アクセスすることも可能ですが、dockerdが提供する豊富な機能を利用しないのは勿体無いので、dockerdを経由する方式が採用されているようです。

また、歴史的経緯からデーモンがdockerdとcontainerdの2つに分離した事情もあります。元々、初期のDockerはdockerdだけの巨大なモノリスとして開発・運用されていた経緯があり、一枚岩の辛さを解消するために一部の機能をcontainerdに切り出しました。その名残りで、現在も2つのデーモンが存在する形となっているようです。

https://www.codingknownsense.com/clear-your-concept/docker-a-comprehensive-guide-to-containerization/docker-history/

# Docker互換API
Rancher DesktopなどのDocker Desktop以外のランタイムではではdockerdは存在せず、代わりにDocker互換のAPIのエンドポイントが存在します。（APIエンドポイントの例：Mobyやnerdctlなど）`docker ps`などのDocker CLIのコマンドをこのAPI層が受け取り、dockerdの代わりの役割を果たします。コンテナの起動や運用といったコアな部分の処理はcontainerdが行うので、dockerdがなくてもそれに代わる存在があれば問題ありません。

# runc
runcはコンテナをLinuxのプロセスとして実際に起動するコンテナランタイムです。コンテナの起動・停止・削除であったり、Linuxカーネルの機能を使ってコンテナを隔離し独立した環境として稼働させたりする役割があります。具体的にはcgroups, namespaceというLinuxカーネルの機能を利用しています。

- cgroups
    - 各プロセスが利用できるリソース（CPU、メモリなど）の制限・管理を行う
    - 例：コンテナAはCPU2コア、メモリ512MBまで、といった制限を与える
- namespace
    - 各プロセスを分離させる
    - 例：
        - PID namespace: プロセスIDを分離しコンテナ内のプロセスは自分がPID 1だと思い込ませる
        - Network namespace: ネットワークインターフェースやIPアドレスを分離

ちなみにruncはGoで実装されています。

# そもそもコンテナランタイムってなんだ？
ここまで当然のようにコンテナランタイムという言葉を使ってきましたが、この言葉の定義を改めて確認します。コンテナランタイムとは「コンテナの起動・実行するためのソフトウェア」です。コンテナランタイムは高レベル・低レベルに分類されることが多く、ここまでで説明してきたdockerd, containerd, runcを整理すると以下のようになります。

- 高レベルランタイム
    - ユーザーからのリクエストを受けてコンテナイメージのpull/pushやコンテナの起動・停止を指示する
    - 例：Docker Desktop, Rancher Desktopなどのユーザーインターフェース
    - 例：dockerd, containerdなどのデーモン
- 低レベルランタイム
    - 高レベルランタイムから指示を受けて実際にコンテナの起動・停止などをLinuxプロセスのレベルで実行する
    - 例：runc

# まとめ
ということで、Rancherで起動したコンテナがなぜDockerコマンドから確認できるんだ？という疑問から深掘りを始めた結果、containerdやruncといった低レベル層の話にまで発展してしまいました。正直、コンテナランタイムについてはもっと深掘りできる余地があるのですが、深掘りしすぎると記事が長くなってしまうのでこの辺りでやめておきます。（あと自分の知識が足りないので書きづらいというのもある）
