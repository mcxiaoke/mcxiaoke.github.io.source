Title: Windows下编译QT 4.6.2
Date: 2010-05-13 00:19
Author: mcxiaoke
Category: Develop
Tags: Windows, QT, MinGW
Slug: qt-4-6-2-compile-use-vs2008-or-mingw

### 编译准备

下载NOKIA网站上的QT SDK，解压或安装到相关目录。  
本文以QT
4.6.2为例，下载地址：  
<http://get.qt.nokia.com/qt/source/qt-win-opensource-4.6.2-vs2008.exe>

或者直接下载源码：  
<http://get.qt.nokia.com/qt/source/qt-everywhere-opensource-src-4.6.2.zip>

下载后安装好的目录结构为（假设安装到D:QT）：  

```
├─share  
├─lib  
├─bin  
├─mingw  
└─qt  
    ├─translations（QT语言文件目录）  
    ├─tools（QT相关工具源码目录）  
    ├─src（源代码目录）  
    ├─qmake（qmake源代码目录）  
    ├─plugins（图形，数据库等插件目录）  
    ├─mkspecs（平台配置文件）  
    ├─lib（库文件）  
    ├─include（头文件）  
    ├─examples（示例程序）  
    ├─doc（文档）  
    ├─demos（演示程序）  
    ├─config.tests  
    ├─bin（命令行工具）  
    └─qtc-debugging-helper
```

configure配置工具在qt目录，以下假设QTDIR=D:QTSDKqt，编译前最好清空lib,demos,examples,docs四个目录，配置完成后在这几个目录下放一个空的Makefile文件，避免make时报错。

### VS2005动态编译

1.设置VS2005的环境变量  
2.设置目标平台  
set QMAKESPEC=win32-msvc2005  
3.进入QTDIR目录运行配置，生成Makefile文件 

```
configure -platform win32-msvc2005 -release -opensource -shared -fast
-qt-sql-sqlite -plugin-sql-sqlite -no-qt3support  -qt-zlib -qt-gif
-qt-libpng -qt-libmng -qt-libtiff -qt-libjpeg -no-webkit
-qt-style-windowsxp -qt-style-windowsvista  
```

4.运行nmake /I /K（附带/I /K
选项可以避免出现错误时编译自动终止，用于跳过错误继续编译其它文件）

经过这样编译的Qt库不依赖mingwm10.dll,libgcc\_s\_dw2-1.dll，但依赖Qt库的Dll文件和微软的CRT运行时库，使用此Qt库编译Windows平台下的程序，发布时需带上用到的Qt链接库Dll文件和微软的CRT运行时库Dll文件。

## VS2005静态编译
  
（VS2008编译同理，启动相应的命令行，设置相应的目标平台，修改相应的配置文件）  

1.设置VS2005的环境变量  
2.设置目标平台  
    set QMAKESPEC=win32-msvc2005  
3.修改mkspecs/win32-msvc2005目录下的配置文件qmake.conf  
将下面两行：  

```
QMAKE\_CFLAGS\_RELEASE    = -O2 -MD  
QMAKE\_CFLAGS\_DEBUG      = -Zi -MDd  
```

修改为：  

```
QMAKE\_CFLAGS\_RELEASE    = -O2 -MT  
QMAKE\_CFLAGS\_DEBUG      = -Zi -MTd  
```

（D是指dynamic，T是指static，d是指debug）  
4.进入QTDIR目录，运行配置，生成Makefile文件  

```
set QMAKESPEC=win32-msvc2005  
configure -platform win32-msvc2005 -release -no-exceptions -opensource
-static -fast -qt-sql-sqlite -plugin-sql-sqlite -no-qt3support  -qt-zlib
-qt-gif -qt-libpng -qt-libmng -qt-libtiff -qt-libjpeg -no-webkit
-qt-style-windowsxp -qt-style-windowsvista  
```
5.运行运行nmake /I /K

经过这样编译的Qt库不依赖于任何Dll文件（如微软的CRT运行时库），使用此Qt库编译Windows平台下的程序发布时不需要附带任何额外的Dll文件。

## MinGW静态编译

1.设置MinGW的环境变量  
2.设置目标平台  
set QMAKESPEC=win32-g++  
3.修改mkspecs/win32-g++目录下的配置文件qmake.conf  
将下面一行：  

```
QMAKE\_LFLAGS = -enable-stdcall-fixup -Wl,-enable-auto-import
-Wl,-enable-runtime-pseudo-reloc  
```

修改为：  

```
QMAKE\_LFLAGS = -static -enable-stdcall-fixup -Wl,-enable-auto-import
-Wl,-enable-runtime-pseudo-reloc  
```

然后将下面一行：  

```
QMAKE\_LFLAGS\_DLL        = -shared  
```

修改为：  

```
QMAKE\_LFLAGS\_DLL        = -static  
```

4.QTDIR目录，运行配置，生成Makefile文件  

```
set QMAKESPEC=win32-g++  
configure -platform win32-g++ -release -no-exceptions -opensource
-static -fast -qt-sql-sqlite -plugin-sql-sqlite -no-qt3support  -qt-zlib
-qt-gif -qt-libpng -qt-libmng -qt-libtiff -qt-libjpeg -no-webkit
-qt-style-windowsxp -qt-style-windowsvista  
```

5.运行运行mingw32-make -i -k（-i -k选项的含义与上面nmake的相同）

 经过这样编译的Qt库，不依赖任何Dll文件（如mingwm10.dll,libgcc\_s\_dw2-1.dll），使用此Qt库编译的Windows平台下的程序发布时不需要附带任何额外的Dll文件。

## 编译事项说明

编译完成后可以删除bin目录中所有不是当前编译日期的文件。

建议编译前移除examples和demos文件夹的所有文件，避免重编译这两个文件夹，加快编译速度。

另外，编译命令请根据自己的实际情况配置，上面我的配置是（以VS2005静态编译为例）： 

``` 
-platform win32-msvc2005 目标平台  
-release 关闭调试信息  
-no-exceptions 除去异常支持  
-opensource 开源版  
-static 创建静态库  
-fast
```

快速配置，只生成Qt库文件及子目录的Makefile文件，其它的Makefile文件后面再使用qmake生成  

```
-qt-sql-sqlite SQLite驱动支持  
-plugin-sql-sqlite  SQLite链接插件支持  
-no-qt3support  不编译Qt3兼容库  
-qt-zlib zlib库  
-qt-gif -qt-libpng -qt-libmng -qt-libtiff -qt-libjpeg 图形格式插件库  
-no-webkit 不编译webkit，此选项可极大加快编译速度，需要使用WebKit的可以删除此选项  
-qt-style-windowsxp -qt-style-windowsvista 支持XP和Vista主题样式
```

注意：使用静态编译的程序通常较大，建议发布前使用UPX压缩，一般可以减少至少50%的大小。


 

