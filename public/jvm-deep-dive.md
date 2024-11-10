---
title: JVMを意図的にクラッシュさせる方法
tags:
  - 'Java'
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに
「JVMをクラッシュさせるとしたらどんな手段を取りますか？」

この質問に答えることができるでしょうか？私はできませんでした。
私は業務でJavaを利用し始めておよそ5年以上経過します。毎日使っているプログラミング言語の仮想マシンのことなので、これはエンジニアとして知っておくべきだなあと思い、色々試してみました。その過程と結果を記事にまとめます。

# Unsafeクラスを使ってNULL参照
ピュアJavaでJVMクラッシュを再現する場合、sun.misc.Unsafeクラスを使って不正なメモリ操作を実行することでクラッシュが可能です。
```java
import sun.misc.Unsafe;

public class CrashUnsafe {
   public static void main(String[] args) throws Exception {
      var field = Unsafe.class.getDeclaredField("theUnsafe");
      field.setAccessible(true);
      var unsafe = (Unsafe) field.get(null);
      
      var address = 0;　// メモリアドレス 0はアクセス不可
      unsafe.putAddress(address, 0L); 
   }
}
```

```bash
#
# A fatal error has been detected by the Java Runtime Environment:
#
#  SIGSEGV (0xb) at pc=0x000000010837af80, pid=70206, tid=5635
#
# JRE version: OpenJDK Runtime Environment Corretto-21.0.1.12.1 (21.0.1+12) (build 21.0.1+12-LTS)
# Java VM: OpenJDK 64-Bit Server VM Corretto-21.0.1.12.1 (21.0.1+12-LTS, mixed mode, sharing, tiered, compressed oops, compressed class ptrs, g1 gc, bsd-aarch64)
# Problematic frame:
# V  [libjvm.dylib+0xa32f80]  Unsafe_PutLong(JNIEnv_*, _jobject*, _jobject*, long, long)+0x130
#
# No core dump will be written. Core dumps have been disabled. To enable core dumping, try "ulimit -c unlimited" before starting Java again
#
# An error report file with more information is saved as:
# /Users/xxx/Documents/workspace/jvm-deep-dive/hs_err_pid70206.log
#
# If you would like to submit a bug report, please visit:
#   https://github.com/corretto/corretto-21/issues/
#
zsh: abort      /usr/bin/env  -XX:+ShowCodeDetailsInExceptionMessages -cp  CrashUnsafe
```

`SIGSEGV (0xb) at pc=0x000000010837af80, pid=70206, tid=5635`とあるように、Segmentation Faultが発生していることがわかります。
`unsafe.putAddress(address, 0L); `がOSのメモリアドレス 0にJVMがアクセスしようとしたため不正なメモリアクセスと判断されます。これによりOSがSIGSEGVを発信しそのシグナルを受けとったJVMはエラーを出力し、Javaのプロセス自体を終了させます。ほとんどのOSではメモリアドレス 0はOSに保護された領域とされており、アドレス0へのアクセスは無効とするよう設計されています。C言語などの低レベル操作を実行できる言語では、アドレス0はNULLに対応する特別な値とされています。

なお、Unsafeクラスとは低レベルレイヤーのメモリ操作を実行することができるクラスです。クラス名からわかるように危険な操作なため、本番環境で利用することは想定されていません。アプリケーションがクラッシュする恐れがあるためです。

上述したコードではリフレクションを使ってUnsafeクラスを利用しました。これは、Unsafeクラスを直接使おうとすると例外が発生するためです。
```Java
import sun.misc.Unsafe;

public class CrashUnsafeFail {
   public static void main(String[] args) throws Exception {
      var unsafe = Unsafe.getUnsafe(); // 直接呼び出し      
      var address = 0;
      unsafe.putAddress(address, 0L); 
   }
}
```

```bash
Exception in thread "main" java.lang.SecurityException: Unsafe
        at jdk.unsupported/sun.misc.Unsafe.getUnsafe(Unsafe.java:99)
        at CrashUnsafeFail.main(CrashUnsafeFail.java:5)
```

`Unsafe.getUnsafe()`で直接呼び出そうとするとSecurityExceptionがスローされます。これはJavaのセキュリティモデルに基づくアクセス制限が課されているためです。getUnsafeのソースコードを見ると以下の通りSecurityExceptionをスローしていることがわかります。
```java
    @CallerSensitive
    public static Unsafe getUnsafe() {
        Class<?> caller = Reflection.getCallerClass();
        if (!VM.isSystemDomainLoader(caller.getClassLoader()))
            throw new SecurityException("Unsafe");
        return theUnsafe;
    }
```

そして、メソッドに`@CallerSensitive`アノテーションがついています。JavaDocを確認すると、このアノテーションが付与された対象はReflectionクラスかそれと同等に値するものから呼び出すよう記載されていました。
```java
/**
 * A method annotated @CallerSensitive is sensitive to its calling class,
 * via {@link jdk.internal.reflect.Reflection#getCallerClass Reflection.getCallerClass},
 * or via some equivalent.
 *
 * @author John R. Rose
 */
@Retention(RetentionPolicy.RUNTIME)
@Target({METHOD})
public @interface CallerSensitive {
}
```

# NULL参照するC言語コードをネイティブ実行
Unsafeを使わない方法として、ネイティブコード実行を使ってC言語の力を借りる方法があります。

```Java:CrashNative.java
public class CrashNative {
    static {
        System.loadLibrary("crash");
    }

    private native void causeSegmentationFault();

    public static void main(String[] args) {
        new CrashNative().causeSegmentationFault();
    }
}
```

```c:CrashNative.c
#include <jni.h>
#include "CrashNative.h" 
#include <stdlib.h>

JNIEXPORT void JNICALL Java_CrashNative_causeSegmentationFault(JNIEnv *env, jobject obj) {
    int *ptr = NULL; 
    *ptr = 1;        
}
```
まず、CrashNative.javaで`private native void causeSegmentationFault();`というネイティブメソッドを定義します。次にCrashNative.cでNULLポインタである`int *ptr = NULL; `を定義し、そこにアクセスするコードを書きます。

ここまでできたらCLIで操作していきます。

```bash
# javaファイルのコンパイル
javac CrashNative.java

# ヘッダファイルの生成 -> CrashNative.h
javac -h . CrashNative.java

# CrashNative.cのコンパイル
# Mac環境では、libcrash.soではなくlibcrash.dylibでoutする
gcc -shared -fpic -o libcrash.dylib -I${JAVA_HOME}/include -I${JAVA_HOME}/include/darwin CrashNative.c

# 実行
java -Djava.library.path=. CrashNative
```

```bash
#
# A fatal error has been detected by the Java Runtime Environment:
#
#  SIGSEGV (0xb) at pc=0x0000000104f67f9c, pid=85066, tid=5379
#
# JRE version: OpenJDK Runtime Environment Corretto-21.0.1.12.1 (21.0.1+12) (build 21.0.1+12-LTS)
# Java VM: OpenJDK 64-Bit Server VM Corretto-21.0.1.12.1 (21.0.1+12-LTS, mixed mode, sharing, tiered, compressed oops, compressed class ptrs, g1 gc, bsd-aarch64)
# Problematic frame:
# C  [libcrash.dylib+0x3f9c]  Java_CrashDemo_causeSegmentationFault+0x18
#
# No core dump will be written. Core dumps have been disabled. To enable core dumping, try "ulimit -c unlimited" before starting Java again
#
# An error report file with more information is saved as:
# /Users/xxx/Documents/workspace/jvm-deep-dive/hs_err_pid85066.log
#
# If you would like to submit a bug report, please visit:
#   https://github.com/corretto/corretto-21/issues/
# The crash happened outside the Java Virtual Machine in native code.
# See problematic frame for where to report the bug.
#
zsh: abort      java -Djava.library.path=. CrashDemo
```

Unsafeの例と同様のログが出力されることを確認できました。

# NPEとSIGSEGVの違い
NPE(NullPointerException)とSIGSEGVの違いは何でしょうか？NPEもNULLポインタへのアクセスを知らせる例外なので、同じことを指してるように見えます。

答えは、NPEはJVM管轄内でのNULLポインタアクセスを報告しており、SIGSEGVはOSレベルでの不正なメモリアクセスを報告しています。

例えばNPEが起きる例は以下です。
```java
public class NPE {
    public static void main(String[] args) {
        Object n = null;
        n.hashCode();
    }
}
```

```bash
Exception in thread "main" java.lang.NullPointerException: Cannot invoke "Object.hashCode()" because "n" is null
        at NPE.main(NPE.java:4)
```

nullの変数に対して何かを参照しようとすることでNPEが発生しています。この事象はJVM上で動くプログラムの中で発生しているため、JVMがNULL参照を検知してNPEをスローしてくれます。JVMのメモリ保護機構一環としてNPEがスローされています。

一方で、SIGSEGVはOSレベルでの不正なメモリアクセスを報告するシグナルです。JVMは何かしらのOS上で実行するので必ずJVMの外にはOSが存在します。JVMを超え、OS上で不正なメモリアクセスを行った場合はOSがSIGSEGVを発信し、JVMのプロセス自体が終了します。

### 余談：シグナル
SIGSEGVって何？という方向けの余談です。SIGSEGVとはOSが発信するシグナルの一つです。SIGSEGV以外にもSIGBUS, SIGKILなどがあります。

- SIGBUS (シグナル番号 7)
  - "bus error"を意味します。これは、プログラムが物理メモリやハードウェアの制限を超えるアクセスを試みた場合に送信されるシグナルです。例えば、アラインメントされていないメモリアドレスへのアクセスや、存在しないメモリ領域へのアクセスなどが原因で発生することがあります。
- SIGSEGV (シグナル番号 11)
  - "segmentation fault"を意味します。これは、プログラムが許可されていないメモリ領域へのアクセスを試みた場合、またはアクセスしようとしたメモリがプロセスにとって有効ではない場合に送信されます。例えば、NULLポインタの参照や、プログラムが使用することのできないメモリ領域への書き込み試みなどが原因で発生します。SIGSEGVは、メモリの不正なアクセスを示す一般的なシグナルです。
- SIGKILL (シグナル番号 9)
  - プロセスを即時に終了させるために送信されます。このシグナルは「キャッチ」や「無視」ができないため、プロセスによって拒否されることはありません。システム管理者やプログラム自体が、回復不能なエラー状態にあるプロセスを終了させるために使用されます。SIGKILLはプロセスを強制的に終了させるための最終手段です。

# さいごに
一通り試して感じたことは「よほどのことをしない限りJVMはクラッシュしない」ということです。不正なメモリアクセスが実行されないようにJVMは様々なガードレールを敷いていましたね。