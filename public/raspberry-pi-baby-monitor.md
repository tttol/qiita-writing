---
title: Raspberry Pi でベビーモニターを自作する
tags:
  - RaspberryPi
private: false
updated_at: '2024-07-16T21:56:17+09:00'
id: 8f31a4faf72cbc2fa0ae
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに
Raspberry Pi（以下ラズパイと呼ぶ）を使って簡易ベビーモニターを自作しました。
ベビーモニターとは赤ちゃんのベッド等に取り付けて、赤ちゃんの様子を監視できるカメラ&モニターのセットのことを指します。
市販のものはお高いので自作することにしました。ちょうど前々から気になっていたラズパイでできそうだったので、勉強も兼ねてということで。

あと、2024年6月現在、ラズパイのカメラモジュール周りの情報はググってもあまりHITせず、HITしたとしても2019年等の古い記事が多いです。
自分のようにカメラモジュールで何かをつくる人のために、執筆時点での最新情報を記載しておこうかなと思います。
（最新といいつつ、Pi 4なんですけど。。。）

# やりたいこと（要件）
- 別室に居る赤ちゃんの様子を撮影する
- 撮影した映像を配信し他の端末から配信映像を確認する
- 電気を消した部屋の映像を撮影する
- 市販のベビーモニターの価格以下の予算で開発する

# できたもの
完成品はこちらです。
![final1.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/159675/301bf2c0-c53c-f244-eeca-d50932f8ed4b.jpeg)
![final2.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/159675/43bd9058-6b75-7278-2204-bacbc5876623.jpeg)


# 用意したもの
### Rasberry Pi 本体
ラズパイで開発を始めるには、まずラズパイ本体を買わなければいけません。
本体には色々種類があります。

| モデル | 特徴 |
|-------|-----|
| Raspberry Pi Zero| 低コストかつコンパクトな設計で、スペースが限られたプロジェクトに最適である。|
| Raspberry Pi Zero WH| 低コストでコンパクトな設計に加え、Bluetooth 4.1対応（WHモデルのみ）により、無線通信が必要なプロジェクトにも対応可能である。 |
| Raspberry Pi 2 Model B| 消費電力と発熱を抑えた設計で、長時間の使用や熱に敏感な環境での使用に適している。|
| Raspberry Pi 3 Model A+| 小型の作業物やプロトタイプの作成に適しており、軽量かつ高性能なデバイスである。|
| Raspberry Pi 3 Model B| 高性能ながらも手頃な価格で、様々な用途に対応できるバランスの取れたモデルである。|
| Raspberry Pi 3 Model B+| デスクトップPCとしての使用からサーバーや電子工作まで、多岐にわたる用途に対応可能な汎用性の高いモデルである。|
| Raspberry Pi 4 Model B| デュアルディスプレイと4K出力、USB 3.0、ギガビットイーサネット、大容量メモリなど、ハイパフォーマンスが求められる作業に最適である。 |
| Raspberry Pi 5| Pi 4の性能に加えて、Bluetooth 5.0およびデュアルバンド802.11ac Wi-Fiもサポートしている。|

Pi 5が最新のモデルですが、今回の自分の用途にはハイスペックすぎる＆高価だったので、1ランク下のRasberry Pi 4 Model Bを購入しました。

価格はAmazonで¥9,275でした (2024/06時点)
### カメラモジュール(赤外線LEDライト付き)
ラズパイ本体にカメラは付いてないので、追加でカメラモジュールを用意する必要があります。
カメラモジュールも色々と種類がありますが、今回は暗い部屋の中でも撮影したかったので暗視撮影のできるRaspberry Pi用OSOYOO IR-CUTカメラを購入しました。
https://www.amazon.co.jp/gp/product/B0CD7KPH3K/ref=ppx_yo_dt_b_asin_title_o00_s00?ie=UTF8&psc=1

### micro SDカード
ラズパイ本体にHDDやSSDは搭載されていません。代わりに、ラズパイ本体にmicro SDカードを挿入してSDカードをストレージとして利用します。
後の手順でラズパイにOSをインストールしますが、OSイメージはSDカードに保存します。

### SDカードアダプター
※これは「私は必要だった」という物です。
自分の使ってるMacBook AirはSDカードスロットが無いため、SDカード読み込み用に外付けのSDカードアダプターが必要でした。
正直、今回以外でこのアダプターを使う気があまりしないので、この出費は地味に痛かったです。

https://www.amazon.co.jp/SD%E3%83%A1%E3%83%A2%E3%83%AA%E3%83%BC%E3%82%AB%E3%83%BC%E3%83%89%E3%83%AA%E3%83%BC%E3%83%80%E3%83%BC-BENFEI-4in1BENFEI-USB-C%E3%81%8B%E3%82%89SD-CF%E3%82%AB%E3%83%BC%E3%83%89%E3%83%AA%E3%83%BC%E3%83%80%E3%83%BC%E3%82%A2%E3%83%80%E3%83%97%E3%82%BF%E3%83%BC/dp/B08P517NW5/ref=sr_1_1_sspa?__mk_ja_JP=%E3%82%AB%E3%82%BF%E3%82%AB%E3%83%8A&crid=2F0B2B1AWO7A1&dib=eyJ2IjoiMSJ9.Koc7fSOwaBrBVEh7Q32jq9SnSjdqT1lkJDQccZMyjOJWtrcqzGz5QSiumsrpFLbpoUad_LWcO1c3ExAxJEpR5B7-6PYXYkfkB7K0iFqOXwfE7f1C8Kp4DevDAJ3OOimmbNZcz8dgrB9ZlmVqh6G0H2rqJN0mcDHWP_x_vgVZcjH-ACECxH_j9Rl8agjBxUBH6yMkOeGvW8XjB2iq7uElw3-XCCxD7FXEMmkDG5BaazfgyYSffUo0lRwlKtWItKwTMI_twQvOSRPaPCfkccO6yyUXV5GnPhoqd2EWGhsNmcI.-le8SUXWU4F_E1iVs6ycSRFapHOjga31PAzP-TuSmn8&dib_tag=se&keywords=microsd%E3%82%AB%E3%83%BC%E3%83%89%E3%83%AA%E3%83%BC%E3%83%80%E3%83%BC+mac&qid=1718146986&sprefix=microsd%E3%82%AB%E3%83%BC%E3%83%89%E3%83%AA%E3%83%BC%E3%83%80%E3%83%BC+mac%2Caps%2C160&sr=8-1-spons&sp_csd=d2lkZ2V0TmFtZT1zcF9hdGY&psc=1

### ACアダプタ
ラズパイに電源を供給するACアダプタです。
ラズパイ本体のモデルによって電圧・電流が異なるようです。今回使うRaspberry Pi 4 Model B+では5.1V, 3Aであれば問題なさそうなので、こちらの規格のACアダプターを購入しました。

https://www.amazon.co.jp/gp/product/B087CF6127/ref=ewc_pr_img_1

### HDMIケーブル
ラズパイにデスクトップ版のOSをインストールする場合、ラズパイ本体を外部ディスプレイに繋いでデスクトップ画面を表示します。
ラズパイ側の端子はmicro HDMI形式なので、これに対応するケーブルが必要です。
普通の（？）HDMIケーブルは持ってますがmicro HDMIに対応したケーブルは持ってなかったので、購入しました。

https://www.amazon.co.jp/gp/product/B0C4GJKHF9/ref=ox_sc_act_image_1?smid=A3AQDHGJNEJ1W7&psc=1

### 冷却ファン＆ヒートシンク付きケース
ラズパイは起動すると本体自身が発熱します。CPUが色んな処理を実行するので当然ですね。
ラズパイ本体に自身を冷却する能力はないので、何かしら冷却の仕組みを用意する必要があります。
王道は冷却ファン・ヒートシンクです。他にも方法あるかもしれませんが。

冷却ファンとはその名の通り本体を冷却するためのファンです。小さい扇風機みたいなものです。
ファンはモーターで回転するので、ファンを稼働させる電力を別途供給する必要があります。
また、静音性・耐故障性も意識したいところです。

ヒートシンクとは発熱した本体に取り付けることで熱を逃がしてくれる部品です。
これは冷却ファンと違って電力は不要で、発熱箇所に両面テープ等で貼り付けるだけで放熱をしてくれます。ただし、熱すぎると放熱が追いつかなくなるので注意です。
![IMG_2962.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/159675/c1b0678e-498c-af80-0999-eaf221f2575e.jpeg)

今回は冷却ファンとヒートシンクが両方搭載されたケースを購入しました。

https://www.amazon.co.jp/dp/B07VB24K9W?psc=1&smid=A2NFT1NT6EPBDZ&ref_=chk_typ_imgToDp

最初はラズパイ本体をケースなしの丸裸で使ってました。しかし丸裸だと基盤を指でベタベタ触ることになってなんとなくまずそうな気がしますし、ケーブルの抜き差しの際に基盤を破損してしまいそうになります。
冷却の仕組みの有無は別としても、ケースはあったほうがいいなあと買ってから感じました。

### スマホスタンド
ラズパイ本体を固定するスタンドです。100均のスマホスタンドを使いました。

### ディスプレイ
ラズパイとHDMIケーブルで繋いで、ラズパイ上のデスクトップ画面やTerminalウィンドウを表示する用です。
後述しますがラズパイ自身のIPアドレスがわかれば他のホストからラズパイにSSHできるので、ディスプレイなしでも開発可能です。

### キーボード・マウス
ラズパイに接続して使用する用です。こちらも他のホストからSSHするのであれば不要です。

# Rasberry Pi OSのインストール
下記の公式手順を参考に勧めます。
https://www.raspberrypi.com/documentation/computers/getting-started.html

### micro SDカードにOSをインストールする
任意のPCにmicro SDカードを接続し、SDカードにOSイメージをDLします
OSイメージのDLにはRaspberry Pi Imagerを利用する。Win/Mac/Linux版で分かれてるので適したものをインストールします。
OSはRaspberry Pi OS (64-bit)を利用します。

### 設定項目
- hostname
    - raspberrypi.local
- username & password
    - 任意のものを指定
- wireless LAN
    - 自宅のWi-Fiを指定
    - Country:JP
- Locale
    - Asia/Tokyo
    - jp
- SSH
    - Enable SSHにチェック
    - Use password authenticationにチェック

ラズパイには以下ユーザーがデフォルトで用意されている
username: `pi`
passwaord: `raspberry`

### SDカードへのOSインストールが終わったら
PCからSDカードを取り外し、SDカードをラズパイに挿します。
マウス・キーボード・HDMIケーブルもこのタイミングで接続します。

### ラズパイ起動
周辺機器の接続ができたら、最後にACアダプタを接続します。
接続した瞬間にラズパイの端のランプが光りだします。赤点灯と緑点滅の2色です。
![](https://storage.googleapis.com/zenn-user-upload/493403b46dc7-20240623.jpg)

公式によると5分以内に起動するとのこと。それ以上時間がかかる場合は何かしらのエラーが起きてるとみたほうがよさそうです。
自分の場合は1〜2分ほどで起動完了しました。
![](https://storage.googleapis.com/zenn-user-upload/04a4a5c6a297-20240623.jpg)

# カメラ・ケースの取り付け
OS起動ができたら次はカメラモジュールとケースを取り付けます。
一旦電源を落とし、説明書通りに取り付けていきます。

![raspi.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/159675/e87d9ad2-4b62-c7f7-ae06-893f4e7dafad.jpeg)

# rpicamを使って写真・動画を撮影
カメラを利用するにはカメラ用のソフトウェアが必要です。
`rpicam`というソフトウェアがOSにプリインストールされているため、こちらを使います。
ラズパイを起動してterminalを開き、以下コマンドで動作確認ができます。
```bash
# プレビュー表示
rpicam-hello

# 静止画撮影
rpicam-jpeg --output test.jpg
```

ググると`libcamera`というOSSのソフトウェアを使った記事が多くHITしますが、公式ドキュメントでは`rpicam`の情報が多かったので`rpicam`を採用しました。
どっちが正解なのかはわかりませんが、なんとなく公式は`rpicam`を推してるような気がします。（たぶん）
# 撮影と配信
ここから本編。
## rpicamで撮影した映像をRTSPプロトコルで配信する
以下公式ドキュメントを見ながら勧めていきます。
> Stream video over a network with rpicam-apps

https://www.raspberrypi.com/documentation/computers/camera_software.html#stream-video-over-a-network-with-rpicam-apps

### ラズパイ側の操作
配信を観る際にラズパイのIPを指定する必要があるので、あらかじめ確認しておきましょう。
```bash
# ラズパイ自身のIPを確認
hostname -I
```

IPがわかったところで、配信を開始します。以下コマンドを実行することで配信が開始されます。
```bash
# 配信を開始する
rpicam-vid -t 0 --inline -o - | cvlc stream:///dev/stdin --sout '#rtp{sdp=rtsp://:8554/stream1}' :demux=h264
```

### クライアント側の操作
PCで配信映像を見る場合、以下コマンドを実行します。
`<ip-addr-of-server>`は先程確認したラズパイのIPで置換します。
```bash
vlc rtsp://<ip-addr-of-server>:8554/stream1
```

iPadやiPhoneで観たい場合は[VLC](https://apps.apple.com/jp/app/vlc-media-player/id650377962)というアプリをAppStoreからインストールし、アプリ内で`rtsp://<ip-addr-of-server>:8554/stream1`を入力して「ネットワークストリームを開く」をタップすると配信を観ることができます。

![streaming.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/159675/f1b4d258-b261-bf27-b423-d48c4e7ada36.png)
↑iPadから配信を視聴できることを確認

### 起動時に配信を開始する
`crontab -e`でcrontabを開き、以下を追記します。
```bash:crontab
@reboot rpicam-vid -t 0 --inline -o - | cvlc stream:///dev/stdin --sout '#rtp{sdp=rtsp://:8554/stream1}' :demux=h264
```
`sudo reboot`で再起動後、配信が自動で開始されていればOKです。

ただし、これをやるとラズパイをディスプレイに繋いでも配信のウィンドウがでかでかと全画面で表示されるだけになり、**他の操作ができなくなります。**

操作をする場合はラズパイ以外のホストからラズパイにSSHして操作する必要があります。
`hostname -I`で確認したIPをもとに、SSHする↓
```bash
ssh <ラズパイ側のユーザー名>@<ip-addr>
```
:::message
OSをSDカードへインストールする際に指定したユーザー名・パスワードがここで必要になる
:::

その後、以下操作でVLCのプロセスを探してkillすれば配信を終了できます。
```bash
ps aux | grep vlc
sudo kill <PID>
```

もしくは、rpicam-vidコマンドに`-n`オプション（--nopreview）を渡してプレビュー表示をOFFにすることができます。自分はこのオプションをcrontabの中に追記しました。

# さいごに
今回作成したベビーモニターにかかった費用は下記のとおりです。
![sum.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/159675/76e26743-3872-f7af-e980-c9f21b6f2133.png)

**総額：19,261円 !!**

市販ベビーモニターの価格はピンキリですが、調べた限り5,000~20,000円くらいでした。
平均とると12,500円なので、作ったほうが赤字ですね。

まあ色々勉強になりましたし、楽しかったのでプライスレス（？）ということで。


# 参考

https://www.raspberrypi.com/documentation/computers/camera_software.html

https://qiita.com/kashee337/items/a0bb101988b5f5319bbd

https://qiita.com/oshiemon/items/84b997076176a84d797f

https://www.itmedia.co.jp/news/articles/1912/27/news036.html

