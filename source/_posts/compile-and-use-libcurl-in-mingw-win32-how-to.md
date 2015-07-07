Title: 使用MinGW编译libcurl静态库
Date: 2010-12-12 20:47
Author: mcxiaoke
Category: Develop
Tags: C, MinGW,Windows,LibCurl
Slug: compile-and-use-libcurl-in-mingw-win32-how-to

libcurl
7.21以后的版本在Windows下的编译比较简单，自带了MinGW和VC环境的Makefile文件，首先去Curl官网下载源代码：  
<http://curl.haxx.se/download.html>，

任选一个下载即可，推荐这个：  
[curl-7.21.2.tar.gz](http://curl.haxx.se/download/curl-7.21.2.tar.gz)  
下载完成后解压开，打开命令行进入curl源码目录，(在此之前请先设置好MinGW的环境变量)：  

    cd curl-7.21.2 
     
编译libcurl库文件：  

    cd lib  
    make -f Makefile.m32  
    
等待编译完成即可  
编译curl可执行文件： 
 
    cd ../src  
    make -f Makefile.m32  
    
一会儿就编译完成了

编译完成后，我们需要复制include头文件和库文件到一个目录供程序开发用 
 
1. 新建curllib目录  
2. 新建curllib/include目录，将源代码include目录里的curl文件夹复制到curllib/include目录，这些是使用libcurl需要的头文件  
3. 新建curllib/lib目录，将源代码lib目录里编译好的库文件libcurl.a，libcurldll.a，libcurl.dll复制到curllib目录  
4. 将MinGW安装目录下lib文件夹里的libwldap32.a和libws2\_32.a复制到curllib/lib目录

至此，curllib就是我们开发中要使用到的libcurl的全部文件，下面新建一个测试程序，假设文件名为curltest.c，代码如下：

```
\#include \<stdio.h\>  
\#include \<curl/curl.h\>

int main(void)  
{  
CURL \*curl;  
CURLcode res;

curl = curl\_easy\_init();  
if(curl) {  
curl\_easy\_setopt(curl, CURLOPT\_URL, "www.google.com");  
res = curl\_easy\_perform(curl);

/\* always cleanup \*/  
curl\_easy\_cleanup(curl);  
}  
return 0;  
}  
```

** 方法一、命令行编译使用licurl的程序

假设测试代码curltest.c位于e:/project  
假设curllib文件夹的位置为c:/curllib  
命令行运行下列命令编译这个测试程序：

```  
cd e:/project  
gcc -I. -Ic:/curllib/include -g -O2 -DCURL\_STATICLIB -c curltest.c  
gcc -s -o curltest.exe curltest.o -Lc:/curllib/lib -lcurl -lwldap32
-lws2\_32  
```

这时可以看到curltest.c目录下生成了一个curltest.exe文件  
接着在命令行输入：  


```
curltest.exe  
```

如果看到命令行窗口输出一些HTML代码，就表示编译成功

** 方法二、Code::Blocks中使用libcurl静态库
 
1. 新建工程，在工程里添加代码同上的curltest.c文件  
2. 将上面curllib/include目录下的curl文件夹复制到MinGW安装目录的include目录  
3. 工程名上右键打开Build Options选项，在Compiler
Settings选项卡下的\#defines里面输入CURL\_STATICLIB，（这表示使用静态库）  
4. 在Linker Settings选项卡下面的link
libraries里添加上面curllib/lib目录里的四个文件：
  
```
    C:curllibliblibcurl.a  
    C:curllibliblibcurldll.a  
    C:curllibliblibwldap32.a  
    C:curllibliblibws2\_32.a  
```

然后回到工程页面，点击Build即可

今天为了编译和使用libcurl库折腾了一下午，记下来供需要的人参考，需要注意的是，本文中编译的是不带ssl和zlib支持的libcurl，如果需要编译支持ssl和zlib的curl，还需要先编译openssl，zlib和libssh，编译zlib比较简单，直接使用源码自带的makefile文件即可，编译openssl需要安装MSYS和Perl，还需要修改一些代码，libssh的编译依赖openssl，网上都可以找到方法，也可以看源码的README文件。

附一篇在C语言中使用libcurl库的文章供参考：  
[使用 cURL 和 libcurl 通过 Internet
进行对话](http://www.ibm.com/developerworks/cn/opensource/os-curl/)  
下面几篇是Curl的文档和教程：  
[Scripting HTTP Requests Using
Curl](http://curl.haxx.se/docs/httpscripting.html)  
[Curl Man Page](http://curl.haxx.se/docs/manpage.html)  
[Curl Mannul](http://curl.haxx.se/docs/manual.html)  
[Using The libcurl C Interface](http://curl.haxx.se/libcurl/c/)  
[libcurl - small example
snippets](http://curl.haxx.se/libcurl/c/example.html)  
[programming with
libcurl](http://curl.haxx.se/libcurl/c/libcurl-tutorial.html)

