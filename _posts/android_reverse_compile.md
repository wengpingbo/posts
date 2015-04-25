title: Android APK 反编译破解
author: WEN Pingbo <wengpingbo@gmail.com>
date: 2013/05/20
tags: [android, 逆向]
categories: [security, android]
---

这里以"魔塔50层"这个游戏为例子，讲解一下 android 下面反编译的过程。

## 准备

首先下载目标apk，我是从这里下载的： [com.ss.magicTower](http://www.wandoujia.com/apps/com.ss.magicTower)。

在准备好 android 开发环境后，需要下载如下工具:

* [apktool](https://code.google.com/p/android-apktool/) - 把 apk 文件反编译成 dalvik 中间码，smali
* [jd-gui](http://java.decompiler.free.fr/?q=jdgui) - 查看 jar 源码文件
* [dex2jar](https://code.google.com/p/dex2jar/) - 把 dex 转换成 jar

## 原理

首先把 apk 文件解压缩，然后提取其中的 .dex 文件，用 dex2jar 把 dex 文件转成 jar 文件，这样就可以用 jd-gui 打开 jar 文件，查看具体的 java 源码了。然后定位要修改的地方，再用 apktool 把 apk 文件转换成 dalvik 的中间码，定位到之前要修改的位置，然后修改保存，再用 apktool 重新打包成 apk 文件。最后一步，用 jarsigner 给前面生成的 apk 文件签名，这样就可以把咱重新制作的 apk 安装到 android 系统上了。

## 过程

解压 apk 文件，可以用 rar 或者 7zip，都行。

转换成 jar：

``` bat
dex2jar.bat classes.dex
```

这一步后，就会在当前目录下生成一个 classes_dex2jar.jar 文件。

用 jd-gui 打开，并定位。一般的程序都会用 proguard 来进行代码混淆，所以你这里看到的都是一些稀奇古怪的变量名，类名和方法名，这对定位会造成影响。但是花点时间，还是能够找出来的。[proguard](http://proguard.sourceforge.net) 现在已经默认加到了 android sdk 里，在 sdk/tools/proguard 里。在程序开发中，如果你希望用 proguard 来混淆自己的代码，只需在 default.properties 里添加一句 `proguardproguard.config=proguard.cfg`，就可以启用 proguard。

apktool 反编译：

``` bat
apktool.bat d mota50.apk mota50
```

命令完成后，会生成一个 mota50 的文件夹，定位到 smali\com\ss\magicTower\k.smali 文件，用文本编辑器打开它。然后修改相应的位置。我这里就是修改了判断条件那个地方，把 if(a.h>180) 改成 if(a.h>-180)，这样就永真了，所以就跳过验证了。

android 用的是 dalvik VM 的中间码，与 pc 端的 java 中间码不同，smali 文件就是 dalvik 的中间码，你可以理解为汇编语言，具体关于 dalvik 中间码定义，可以看这里 [dalvik-bytecode](http://source.android.com/tech/dalvik/dalvik-bytecode.html)。

修改完之后，需要重新打包成apk文件：

```
apktool.bat b mota50
```

这个命令，会在 mota50 文件夹中生成 build 和 dist，两个文件夹，apk 文件存放在 dist。你也可以用这个命令：

```
apktool.bak b -f mota50 newmota.apk
```

这个命令会在当前目录生成一个 apk 文件。

这样生成的 apk 文件并不带签名，而在 android 中，不带签名的文件，是无法安装的。所以我们需要给它重新添加签名。
签名需要密钥，可是我们手上没有，需要自己生成一个，命令如下：

```
keytool -genkey -v -keystore magic.keystore -alias magic.keystore -keyalg RSA -keysize 2048 -validity 10000
```
注意： -keystore 和 -alias 参数后面跟的名字一定要一样，否则后面用这个密钥去签名 apk 的时候，会提示找不到证书链！

具体参数含义，请看这里 [app-signing](https://developer.android.com/tools/publishing/app-signing.html)

这个命令完成后，会在当前目录生成一个 magic.keystore 密钥文件
有了密钥，我们就可以通过下面的命令，给apk进行签名：

```
jarsigner -verbose -sigalg MD5withRSA -digestalg SHA1 -keystore magic.keystore -signedjar mota50_signed.apk newmota50.apk magic.keystore
```

注意：由于在jdk7中，默认的签名算法已经改变了，所以你必须自己指定签名算法(-sigalg)和摘要算法(-digestalg)。否则签名无效，无法安装。

这个命令就是用 magic.keystore 密钥给 newmota50.apk 签名，并生成一个 mota50_signed.apk 的文件，这就是最终的文件了。

当然，上面介绍的是手动签名。你也可以通过 eclipse，或者其他的 IDE 来进行签名，网上也有一个 auto_signed 图形工具来签名。这里就不在复述。

安装的时候，你可以安装在 android emulator 里，或者真机里：

```
adb install mota50_signed.apk
```
