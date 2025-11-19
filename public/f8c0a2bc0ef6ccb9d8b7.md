---
title: JVMのGarbage Collectionに対する自分の理解
tags:
  - Java
  - GarbageCollection
private: false
updated_at: '2025-07-04T05:23:45+09:00'
id: f8c0a2bc0ef6ccb9d8b7
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに
JavaではGarbage Collection（以下GCと呼ぶ）という機能があります。これは、JVMのメモリ空間上で不要になったオブジェクトを解放することでメモリの空きを確保する機能です。GC＝メモリ解放してくれる機能、ぐらいにしか理解できておらずもっと深い理解をしておくべきだなと感じたので、改めて知識を整理し、自分の理解を文章にまとめておこうと思います。

# JVMのメモリ空間
JVMのメモリ空間にはスタックとヒープメモリと呼ばれる二つの領域があります。

### スタック
- 一時的な情報やローカル変数を格納するための領域
- 格納されるデータの例
    - メソッド引数
    - オペランドスタック（演算の途中結果などを一時的に保持する）
    - プリミティブ型の変数
    - オブジェクトのアドレス（例：`0a2fcd`）
- **GCの対象外。** メソッドの実行が終了した時点で自動的に解放される。


### ヒープメモリ
- オブジェクトが持つ実際の値などを格納するための領域
- 格納されるデータの例
    - オブジェクトが持つ実際の値
        - 例：`var hoge = new Hoge("aaa");`の"aaa"の部分
    - 配列の一つ一つのデータ
    - **GCの対象となる。** GCが解放しない限りメモリ内にデータが残り続ける。


![スクリーンショット 2025-06-17 21.46.52.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/159675/d24cf657-51df-47c2-94f6-09e8c8d2156b.png)

### Metaspace
クラスのメタデータを格納する領域です。メタデータとはクラスの構造・メソッドの情報・フィールドの情報などを指します。Java 7まではヒープの一部とされていましたがJava 8からはヒープとは独立した領域が確保されるようになりました。これによりヒープ領域の枯渇によるOOMリスクが低減しました。

クラス数の多いアプリケーションではアプリケーション起動時にMetaspaceが枯渇する可能性があるため、`-XX:MaxMetaspaceSize`オプションで適宜拡張が必要です。

# ヒープメモリ内の構造
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/159675/f172c554-70f7-44a5-985c-05e608c8e5e8.png)

ヒープメモリ内は世代によって更にYoung GenerationとOld Generaionという領域に分割されます。新しく発生したデータはYoung Generaionに格納され、Young Generationで不要になったデータはMinor GCによってOld Generaionに移動されます。世代ごとの領域の管理をGCが担っています。

### Young generation
Young Generationの中は更にEden space, Suvivor spaceの2つの領域に分割されます。

- Eden space
    - 新しく発生したデータはEdenに格納される
    - Eden内で不要になったデータはSuvivorに移動される（Minor GC）
- Suvivor space
    - Edenで不要になったデータを格納する領域
    - Suvivor内で一定時間生き残ったデータはOld Generationに移動される

### Old Generation
Old GeneraionにはYoung Generationで不要になったデータが移動されてきます。

# Minor GC/Major GC(Full GC)
Minor GCはコピーGCとも呼ばれ、データを他の領域にコピーするだけの処理です。
Major GCはメモリの解放を行う処理であり、重い処理です。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/159675/0c88160d-1a55-4e68-8a75-6cbd3c58edb6.png)

::: note
＜Young GenerationがEden spaceとSuvivor spaceに分割されている理由＞
- メモリのフラグメンテーションを防ぐ
- 軽量な処理であるMinor GCを頻繁に実行し、重い処理であるMajor GCの実行頻度を減らすため
:::

# GCの挙動
GCの内部の挙動は大きく3つのフェーズに分かれます。基本的なGCアルゴリズムでは全てのフェーズでSTWが発生しますが、CMS GCやG1 GCではSTW時間を最小化するために可能な限り並行処理を行います。

::: note
Stop The World・・・GCの実行中にアプリケーションスレッドを全て停止してしまうこと
:::

### 1. Mark
ヒープメモリ全体を探索し、アプリケーションから到達可能なオブジェクトを全て洗い出してマークします。Markフェーズでマークされたオブジェクトはアプリケーションで使用中のオブジェクトとみなされます。

### 2. Sweep
Markフェーズで検知できなかったオブジェクトは到達不可能オブジェクトとみなされ、GCの解放対象となります。Sweepフェーズでこれらのオブジェクトを解放しメモリ空間の空きを確保します。

### 3. Compaction
Sweepフェーズでメモリの解放を行なった結果、メモリ空間のフラグメンテーションが起こりえます。Compactionフェーズでは断片化したメモリ空間を整頓するために使用中の領域を片側に寄せ、空き領域を連続した領域にします。

3つのフェーズの中でCompactinフェーズが最もSTWの時間が長くなります。後述する様々な種類のGCでは、Compactionフェーズを可能な限り短くすることでいかにSTWの時間を最小化できるかについて尽力されています。

# GCの種類
### Serial GC
Serial GCは単一のスレッドでGCを実行します。Serial GCを実行している間はSTWによりアプリケーションスレッドが停止します。シングルコアCPUが主流だった時代はSerial GCがよく利用されていたようです。

### Parallel GC
Parallel GCではマルチスレッドでGCを実行します。STWは発生しますがSerial GCよりもSTWの時間を短縮することが可能です。

### CMS GC(Concurrent Mark-Sweep GC)
アプリケーションスレッドと並行してGCを実行します。そのためGC中であってもアプリケーションスレッドが停止しません。また、CMS GCはCompactionフェーズをスキップします。

::: note warn
CMS GCはJDK 9で非推奨となり、JDK 14で削除されました。

CMS GCの問題点
- Compactionを実行しないためメモリのフラグメンテーションリスクが大きく、OOMが起きやすい。
- アプリケーションスレッドと並行でGCスレッドが実行されるためCPU負荷が高い。アプリケーションスレッドのスループットを低下させるリスクがあった。
:::

### G1 GC(Garbage-First GC)
G1 GCではリージョンと呼ばれる固定長の単位でヒープメモリを分割し、最もガベージの多いリージョンから優先的にGCを実行します。GCの各処理を可能な限りアプリケーションスレッドと並行で行うことでSTWを最小に抑えるよう努めます。

::: note
G1 GCはJDK 9以降ではデフォルトのGCとして採用されています。Javaの起動引数で明示的に他のGCを指定しない限りG1 GCが適用されます。
:::

# まとめ
- JVMにはスタックとヒープという二つの領域がある
- ヒープの中はYoung(Eden+Suvivor), Oldに分かれる
- GCはMark→Sweep→Compactionの順に動作する
- GCにはMinor GCとMajor GCがある
- Serial GC, Parallel GC, CMS GC, G1 GCがある（これ以外にもある）。現在のデフォルトはG1 GC。

# 参考

https://docs.oracle.com/en/java/javase/21/gctuning/index.html#GUID-8A443184-7E07-4B71-9777-4F12947C8184

https://techblog.zozo.com/entry/g1gc-heap-region

https://qiita.com/dung717/items/6552005a4fdccc893c43
