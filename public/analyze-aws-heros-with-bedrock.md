---
title: AWS Herosに選出される要因をAmazon Bedrockを使って分析してみる
tags:
  - AWS
  - AI
  - bedrock
private: false
updated_at: '2024-07-23T08:03:48+09:00'
id: 061686669ff45babf443
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに
[AWS Heros プログラム](https://aws.amazon.com/jp/developer/community/heroes/?community-heroes-all.sort-by=item.additionalFields.sortPosition&community-heroes-all.sort-order=asc&awsf.filter-hero-category=*all&awsf.filter-location=*all&awsf.filter-year=*all&awsf.filter-activity=*all)というものをご存知でしょうか？
AWSコミュニティに対して大きく貢献し、コミュニティに多大なる影響を与えた人を称えるプログラムです。AWS社員さんの他薦により選出され、選出されたHerosはAWSサービスチームとのディスカッション機会や、AWS re:Inventへの招待券、AWSクレジットなど様々な特典を得ることができます。

AWS Herosにはいくつかのカテゴリが存在します。
コンテナに精通した人はコンテナヒーロー、活発なコミュニティ活動を行った人はコミュニティヒーロー、など。
![heroes-logo-dark.4a3d8b6fa153722278d39c43da901d2c724cefc5.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/159675/3312ad09-0335-9aee-29b1-5966a3b1dc21.png)
私は最近このプログラムの存在を知り、色々調べていくうちに自分もAWS Heros プログラムに選出されたいなあと思い始めました。
しかし、何から始めたらよいのかよくわかりません。
そこで、生成AIを使ってAWS Herosに選出されるためにどういった活動をすればよいかを分析してみました。

# 使用するAI
- Amazon Bedrock（のチャットプレイグラウンド）
- モデル：Cluade Sonnet

# 分析
Bedrockを使って分析していきます。
前提として、私はDevToolsカテゴリのHerosを目指したいので、DevTools Herosについて分析します。

### 既存のHerosがどういった要因で選出されたのかを分析
まず、既存のDevTools Heros達がどういった要因・理由で選出されたのかを分析していきます。
[ここ](https://aws.amazon.com/jp/developer/community/heroes/?community-heroes-all.sort-by=item.additionalFields.sortPosition&community-heroes-all.sort-order=asc&awsf.filter-hero-category=heroes%23devtools&awsf.filter-location=*all&awsf.filter-year=*all&awsf.filter-activity=*all)に各Herosのプロフィールページがあり、どういった経緯で選出されたのかが簡単にですがまとめられています。ここからひとりひとりのリンクをコピーしてBedrockに食わせていきます。
↓
```
https://aws.amazon.com/developer/community/heroes/adam-bien/?did=dh_card&trk=dh_card
https://aws.amazon.com/developer/community/heroes/bhuvaneswari-subramani?did=dh_card&trk=dh_card
https://aws.amazon.com/jp/developer/community/heroes/brian-hough/?did=dh_card&trk=dh_card
https://aws.amazon.com/developer/community/heroes/adam-elmore/?did=dh_card&trk=dh_card
https://aws.amazon.com/developer/community/heroes/gert-leenders/?did=dh_card&trk=dh_card
https://aws.amazon.com/developer/community/heroes/jared-short/?did=dh_card&trk=dh_card
https://aws.amazon.com/developer/community/heroes/johannes-koch/?did=dh_card&trk=dh_card
https://aws.amazon.com/developer/community/heroes/kenta-goto/?did=dh_card&trk=dh_card
https://aws.amazon.com/developer/community/heroes/matt-coulter/?did=dh_card&trk=dh_card
https://aws.amazon.com/developer/community/heroes/matthew-bonig/?did=dh_card&trk=dh_card
https://aws.amazon.com/developer/community/heroes/monica-colangelo/?did=dh_card&trk=dh_card
https://aws.amazon.com/jp/developer/community/heroes/sebastian-korfmann/?did=dh_card&trk=dh_card
https://aws.amazon.com/developer/community/heroes/thorsten-hoger/?did=dh_card&trk=dh_card
https://aws.amazon.com/developer/community/heroes/yasunori-kirimoto/?did=dh_card&trk=dh_card

上記のURLはAWS DevTools Herosプログラムに選出された人の紹介ページです。
紹介ページには彼らの簡単な来歴と、AWS DevTools Herosに選出された理由・要因が記載されています。
全てのリンク先の情報を解析して、Herosに選出される理由として多いものを分析して教えて下さい。
```

```
上記のURL一覧を確認した結果、AWS DevTools Herosに選出される主な理由として、以下のようなポイントが多く見られました。

1. アクティブな技術コミュニティへの貢献
   - ブログ執筆、書籍出版、セミナー講演など、技術知識の共有や啓発活動
   - ユーザーグループやコミュニティでのリーダーシップ
   - オープンソースプロジェクトへの貢献

2. AWS製品やサービスの専門的な知識とスキル
   - 特定のAWSサービスの深い理解と実践経験
   - 認定資格の取得と専門性の証明
   - プロダクションレベルでのAWS活用事例

3. 革新的なソリューションの開発と導入
   - クラウドネイティブアプリケーションの設計・開発
   - 新しいAWS機能の早期導入と活用
   - ベストプラクティスの確立と普及

4. 技術的リーダーシップとメンターシップ
   - 企業内でのAWS導入やクラウド移行の主導
   - チームメンバーの育成と技術指導
   - アーキテクチャの設計やコード審査など

これらの活動を通じて、AWS DevTools Herosとして選出された人々は、AWS技術の発展と普及に大きく貢献していることがうかがえます。
```

ふむふむ。
AWSコミュニティへの大きな貢献や、AWSに関する専門スキル、技術的リーダーシップなどが要因で選出された人が多いと。


では、各Herosに共通する部分を聞いてみましょう。
↓
```
Herosに共通する特徴はありますか？
```

```
はい、AWS DevTools Herosに選出された人々に共通する特徴が几つか見受けられます。

1. 技術への情熱と探究心
   Herosの多くは、常に新しい技術を探求し、学び続ける姿勢を持っています。AWS の最新機能や関連テクノロジーに対する深い関心と知識欲があります。

2. オープンでコラボレーティブな姿勢  
   技術コミュニティに積極的に貢献し、他者との知識共有を重視しています。ブログ、書籍、プレゼンテーションなどを通じて自らの経験や知見を発信しています。

3. リーダーシップとメンターシップ
   多くのHerosが企業内や技術コミュニティにおいてリーダーシップを発揮しています。チームメンバーの育成やメンタリングに力を入れる姿勢があります。  

4. 実務での貢献とベストプラクティスの実践
   Herosは実際の業務においてAWSを効果的に活用し、優れたソリューションを構築する実績があります。ベストプラクティスの確立と普及に尽力しています。

5. 認定資格の取得と専門性
   AWS の各種認定資格を取得し、特定の領域で専門的な知識とスキルを身につけています。深い技術力が評価されています。

つまり、技術への情熱、オープン性、リーダーシップ、実務貢献、専門性が、AWS DevTools Herosに共通する特徴として挙げられます。
```

技術への飽くなき探究心と、それをナレッジとしてアウトプットすることが大事と。
また、ここでもリーダーシップと専門性が述べられていますね。


よくあるアウトプットの形として、DevTools Heros達はOSSにコミットしている方がとても多い印象です。
それらが選出に影響しているかどうかを聞いてみましょう。
↓
```
OSSへの貢献がメインでHerosに選出された人を列挙してください
```

```
確かに、OSSへの顕著な貢献がメインの理由でAWS DevTools Herosに選出された人が複数います。代表例を挙げますと、

- Jared Short
  AWS SAMやAWS CDK、Serverless ApplicationリポジトリなどのOSSプロジェクトに大きく貢献。

- Johannes Koch
  Ballerina言語のメンテナで知られ、Ballerina CloudのAWS統合に貢献。

- Kenta Goto
  AWS CDKの共同設立者として主要貢献者。CDK workshopsの運営にも携わる。

- Sebastian Korfmann
  AWS CDKの主要貢献者で、CDKの本も執筆。コアチームのメンバー。

- Matthew Bonig
  Pulumi OSSプロジェクトの共同設立者でプロジェクトリーダー。クラウド デプロイツールを開発。

- Yasunori Kirimoto
  AWS電子書籍リーダーアプリ端末のOSS化に貢献。大規模マルチアカウント構成のOSS化にも尽力。

上記のように、AWS公式が提供するOSSプロジェクトや、AWSを活用する上で有用なOSSプロジェクトに中核的な役割を果たした方々がHerosに選ばれています。

OSSへの貢献を通じて、クラウドネイティブ開発の生産性や可用性の向上に多大な影響を与えたことが、彼らがHerosとして評価された大きな理由です。
```

具体的な個人名レベルで挙げてくれましたね。
今更ながら、URL先の内容をちゃんと読み取ってくれていることがわかりました。


逆に、OSS貢献以外で選出された人はいるのか聞いてみましょう。
↓
```
オープンソースプロジェクトに貢献している人が多く見受けられます。
逆に、オープンソースプロジェクトに貢献していないけどDevTools Herosに選出されている人はいますか？
```

```
はい、オープンソースプロジェクトへの直接の貢献が明記されていないけれども、AWS DevTools Herosに選出されている人々が確かにいます。

例えば、以下の人々がその典型例です。

- Adam Bien
  企業のクラウド移行を主導し、書籍執筆、講演、ブログ等で技術情報を広く発信。

- Bhuvaneswari Subramani  
  AWS認定資格を多数取得し、研修講師やメンターとして活躍。マネージドサービスの専門家。

- Matt Coulter
  クラウドネイティブアプリケーションのアーキテクチャ設計で実績。AWS re:Invent、ブログ等で知見を共有。

- Monica Colangelo
  企業の DevOpsとAWSの導入を主導。技術コミュニティでのリーダーシップを発揮。

- Thorsten Hoeger
  クラウドネイティブ開発の第一人者として知られ、書籍、ブログ、講演で積極的に情報発信。

これらの人々に共通するのは、オープンソース貢献よりも、企業運用での実践的な経験、優れた技術力、コミュニティへの貢献、情報発信力が強みとなり、Herosに選ばれた点にあります。

つまり、オープンソースへの直接の関与がなくても、他の分野での卓越した実績があれば十分にHerosに名を連ねられる可能性があると言えます。
```

OSS貢献以外で選出された人も多数いるようです。

### これからHerosを目指すには何を意識すればよいか質問 
既存のHeros達のことはよくわかったので、次に「これからHerosになるにはどうすべきか？」を聞いてみましょう。

:::note info
Herosの審査基準は相対評価とのことなので、その旨もプロンプトに含めてみます。
https://qiita.com/numaguchi/items/82435e5cf5c042ed3afa

> また、いずれのカテゴリでも言えることは、AWS HEROとして選出される絶対的な「基準」はなく、つねに「相対的」に審査されています。
:::

↓　　　　　　　　　　　　　　　　
```
AWS Herosへの選出基準に絶対的な評価はなく、既存のHerosと常に相対的に評価されるそうです。
2024年7月現在、これからAWS DevTools Herosに選出されるために必要な要素は何だと思いますか？
具体的に、どういったアクションを起こすべきだと考えられますか？
```

```
AWS DevTools Herosへの選出は相対的な評価であり、時代とともに求められる要件は変わっていくでしょう。2024年7月時点でHerosに選ばれるためには、以下の点が重要になると考えられます。

1. 最新のAWS技術の活用と貢献
   CloudNativeアプリケーションの設計・開発、Severless、コンテナ、機械学習などの最新の領域でAWSを活用し、その知見を積極的に発信すること。新しいAWS製品やサービスの早期導入と、ベストプラクティスの形成に貢献する。

2. OSSプロジェクトへの中核的貢献
   AWSの公式OSSプロジェクトやAWS関連の有力OSSプロジェクトに対して、コア開発者として中心的な役割を果たす。プロジェクトの成長と技術的方向性に大きな影響力を持つ。

3. 高度な専門性とAWS認定資格
   特定のAWS領域で極めて高い専門性を持ち、認定資格の最上位レベル(プロフェッショナル)を取得する。その分野のエキスパートとして認知される。

4. テクノロジーリーダーシップと情報発信力  
   企業内外で明確なテクノロジーリーダーシップを発揮し、コミュニティでの存在感が大きい。書籍執筆、講演、ブログなどでの積極的な情報発信を行う。

5. 革新的で高難易度のソリューションの実装
   クラウドネイティブで革新的、かつ高い難易度を持つソリューションを実際に構築・運用した実績があり、その過程で高度な技術的課題を克服した経験がある。

こういった点に注力し、より先端的で価値の高い貢献を追求することが、2024年時点でHerosに求められる要素になるのではないかと考えられます。
```

https://qiita.com/numaguchi/items/82435e5cf5c042ed3afa

最新技術を使った知見の発信、OSSへの中核的貢献、テクノロジーリーダーシップなど、どれも頷ける内容ですがなんだか当たり障りのない回答にも聞こえます。
既存のHerosとの差別化について、深堀りして聞いてみましょう。
↓
```
既存のHerosとの差別化をするために、どんなことを素べきだと思いますか
```

```
既存のAWS DevTools Herosとの差別化を図るためには、以下のようなアプローチが考えられます。

1. 新しい技術領域への特化
   機械学習、AI、量子コンピューティングなど、まだHerosの活躍が少ない分野でAWSを活用し、その第一人者としての地位を確立する。新領域での先駆的な貢献が評価されるでしょう。

2. オープンソースプロジェクトの立ち上げと主導
   既存のHerosがコア開発者として関わるOSSプロジェクトとは別に、新しいOSSプロジェクトを立ち上げ、リーダーシップを発揮する。新規プロジェクトのビジョンと技術的方向性を主導することが重要。

3. 革新的なアーキテクチャパターンの確立
   マルチクラウド、ハイブリッドクラウド、エッジコンピューティングなど、新しいクラウドアーキテクチャパターンを提唱し、実装して成果を上げる。斬新なアプローチで技術的インパクトを与える。

4. 特化した分野でのカンファレンス開催
   AIオペレーション、クラウドセキュリティ、DevSecOpsなど、特化した分野で世界的な規模のカンファレンスやサミットを企画・開催する。技術コミュニティでの影響力を最大化する。

5. 破壊的なビジネスモデルの創出
   AWSを活用して、従来の産業や事業モデルを根底から覆すような、破壊的なビジネスモデルやサービスを立ち上げる。技術的側面と事業的側面の両方で大きなインパクトを与える。

このように、既存の枠を超えた先進的な取り組みに挑戦し、AWS技術の可能性を最大限に広げていくことが、差別化のカギとなるでしょう。
```

既存Herosの少ない領域・分野で第一人者として活躍する、新規OSSを立ち上げる、破壊的なビジネスモデル（？）を創出する、などが挙がりました。
何か新しいものを立ち上げて、そこで第一人者になれってことですね。

# さいごに
生成AIはこうした1on1の壁打ち相手にとても適していると思います。
今回はAWS Herosについて分析しましたが、他のテーマでやってみるのも面白いと思います。

:::note warn
おことわり
本記事に記載した生成AIの結果はAWS Herosの選考基準を保証するものではなく、あくまで個人的に調査したものです。
生成AIの回答は必ずしも正しいとは限りませんので、その点ご理解ください。
:::

# 参考記事

https://qiita.com/numaguchi/items/82435e5cf5c042ed3afa
