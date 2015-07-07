Title: Android程序反编译流程
Date: 2010-11-22 16:28
Author: mcxiaoke
Category: Android
Tags: Android, Decompile
Slug: android-apk-files-decompile-how-to

**Android程序的反编译**

APK是Android
Package的简称，是Android平台安装包的标准封装格式，实际上是标准的zip压缩文件，里面包含清单文件，数字签名文件，压缩后的资源文件，以及二进制dex字节码文件，Android程序的反编译包括两部分，三个流程，二部分是资源文件resources.arsc的反编译和字节码文件classes.dex的反编译；三流程是资源文件反编译为人类可读的XML文本格式和图片文件，字节码文件dex反编译为标准的Java压缩格式jar(标准的Java
Class文件)，然后将class文件反编译为Java源代码。下面说明各个流程的方法和所需要的工具：

**1. 资源文件的反编译**

资源文件包括resources.arsc和res目录中的XML文件，如果直接打开是乱码，resources.arsc是压缩过的二进制文件，XML也是压缩过的二进制格式的文件，我们要做的是还原他们的本来面目：二进制格式的图片，和文本格式的XML文件。我们使用最方便的工具：Android
Apktool，这个工具的作者是XDA的高手，他同时还是Brut版Google
Maps的作者。这个工具使用非常方便。

Android
Apktool发布地址：<http://forum.xda-developers.com/showthread.php?t=640592>  
Android Apktool项目地址：<http://code.google.com/p/android-apktool/>

以Windows系统为例，需要下载这两个文件：  

写好的bat批处理脚本：[apktool-install-windows-r05-ibot.tar.bz2](https://android-apktool.googlecode.com/files/apktool-install-windows-r05-ibot.tar.bz2)  

apktool可执行文件：[apktool1.5.2.tar.bz2](https://android-apktool.googlecode.com/files/apktool1.5.2.tar.bz2)

将它们解压到同一个文件夹就可以了，使用方法如下，  
以Android系统自带的Gallery.apk文件为例：

-   反编译资源文件，并将字节码反编译为smali文件：命令行输入 apktool.bat
    d Gallery.apk
    ，当前目录下会生成一个Gallery目录，里面就是反编译好的文件，res目录里为反编译好的资源文件，smali目录里为反编译好的smali字节码文件。由于我们只需要资源文件，所以可以使用下面的命令。
-   仅反编译资源文件，命令行输入 apktool.bat d -s Gallery.apk
    同样会生成Gallery目录，里面的res目录里为反编译好的资源文件，但是classes.dex没有反编译为smali文件。

**2. 二进制字节码文件的反编译**

解压apk文件可以发现里面的classes.dex，这就是二进制字节码文件，我们要把它反编译为标准的Java
Class文件，需要的工具是dex2jar，该工具是国人开发的，目前还在不断更新中。  
dex2jar项目地址： <http://code.google.com/p/dex2jar/>  
dex2jar下载地址：
[dex2jar-0.0.9.15.zip](https://dex2jar.googlecode.com/files/dex2jar-0.0.9.15.zip)

使用方法非常简单，还是以上面的Gallery程序为例，取出里面的classes.dex文件后，命令行输入
dex2jar.bat classes.dex
就会在同级目录下发现反编译好的jar文件classes.dex.dex2jar.jar，下面的步骤就是将Jar文件反编译为Java源文件了，需要是的工具是JD-GUI，或者DJ
Java Decompiler。  
JD-GUI是著名的Java反编译工具JAD的GUI版本，下载地址：  
    <http://java.decompiler.free.fr/?q=jdgui>  
此程序还提供一个Eclipse插件，使用方法，直接用JD-GUI打开刚才反编译好的Jar文件就可以看到Java源代码了，如果原始程序没有经过混淆，那么你会发现代码非常清晰，很容易看懂。

**3. 使用Logcat查看代码的功能**

反编译出来后一般还有很多代码对应的功能不容易弄懂，此时可以打开程序，对照Logcat里面的调试信息，很容易看到代码对应的功能，至于代码里的资源文件ID，由于都是数字，需要到R.java里面去查找对应的资源文件名称。这是一种很好的学习的方法，但也可能被用作非法用途。

