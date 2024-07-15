---
title: UNIX系計算言語”bc”をちゃんと調べてみる
tags:
  - Mac
  - Terminal
  - UNIX
  - bc
private: false
updated_at: '2024-04-12T07:18:32+09:00'
id: c0fced5e1fde84ed09b0
organization_url_name: null
slide: false
ignorePublish: false
---
## bcとは
UNIX系で計算を行うときに使う計算言語です。  

     NAME
           bc - An arbitrary precision calculator language

"An arbitrary precision calculator language"を和訳すると「任意精度計算言語」です。

<br/>
Macのターミナルで`bc`と打ってEnterを押すと、ターミナル上でbcを使うことができます。

```shell
$ bc
bc 1.06
Copyright 1991-1994, 1997, 1998, 2000 Free Software Foundation, Inc.
This is free software with ABSOLUTELY NO WARRANTY.
For details type `warranty'. 
```

bcは特に指定がなければ対話型モードで実行されます。  
例えば、

```shell
1+2
```

と入力してEnterを押すと

```shell
3
```

と結果が返ってきます。  
↓↓実際の使用例↓↓
![スクリーンショット 2018-08-17 20.53.01.png](https://qiita-image-store.s3.amazonaws.com/0/159675/82470872-2d45-36ae-d010-69877a5ea710.png)

※`scale=5`とすることで小数を5桁まで表示するように設定しています。（デフォルトでは`scale=0`）
## オプション
`bc`は起動時にオプションを付けることができます。以下は`man bc`でオプションを確認した結果です。

```shell
OPTIONS
       -h, --help
              Print the usage and exit.

       -i, --interactive
              Force interactive mode.

       -l, --mathlib
              Define the standard math library.

       -w, --warn
              Give warnings for extensions to POSIX bc.

       -s, --standard
              Process exactly the POSIX bc language.

       -q, --quiet
              Do not print the normal GNU bc welcome.

       -v, --version
              Print the version number and copyright and quit.
```

英語を全部訳すのは面倒なのでよく使うやつだけ見てみましょう。
#### -q, --quiet
実行時に最初に出てくるバージョン情報やcopyrightを非表示にしてくれます。`bc="bc -q"`みたいなエイリアスを登録してもいいぐらいマストなオプションだと思います。
#### -l, --mathlib
`bc -l`で実行すると、三角関数（sin,cos,arctan）、自然対数（log）、指数関数（exp()）が使えるようになります。  
`-l`をつけた場合、`scale`が自動で20に設定されます。（＝小数を20桁表示する）

```shell
c(0)+1   #c(0) = cos(0)
2.00000000000000000000
s(0)  #sin(0)
0
l(1)   #log(1)
0
l(2.71828182845904523536) #log(e)
.99999999999999999999
```

## 基数計算
ibase, obaseというパラメータをいじると任意の基数での入力・出力が可能になります。  
例えば、`ibase=10, obase=16`とすると、10進数で入力した計算式を16進数で結果出力します。  （デフォルトではどちらも10なので10進数で表示される）

```bash
ibase=10
obase=16
10+5
F
```
## 応用：非対話モードで実行
以下のようにリダイレクション`<<`を使って非対話モードでbcを実行することができます。

```shell
$ bc <<EOF
> scale=3
> 4/3
> EOF
1.333
```

上記では、`EOF`が入力されるまで式の入力を受け付けます。
<br/>
あるいは、パイプ記号`|`を使って以下のように書き換えることもできます。

```bash
$ echo "scale=3; 4/3;" | bc
1.333
```

## 円周率を任意の桁まで求める

```bash
$ bc -lq
scale = 10;
(12*a(1/49)+32*a(1/57)-5*a(1/239)+12*a(1/110443))*4
3.1415926420
```

`a()`はtanの逆関数arctanのことです。  
`scale`で指定した桁数分の円周率を表示します。


## 参考

http://pubs.opengroup.org/onlinepubs/9699919799/utilities/bc.html

https://ja.wikipedia.org/wiki/Bc_(UNIX)

https://tech.nikkeibp.co.jp/it/article/COLUMN/20060228/231172/
