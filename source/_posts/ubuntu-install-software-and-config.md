title: Ubuntu软件安装和使用配置
date: 2011-05-30 14:00
author: mcxiaoke
categories: 
- Linux
tags: 
- Ubuntu
#slug: ubuntu-install-software-and-config
---
* 2011.05.06 Ver 1.0
* 2011.05.11 Ver 1.1
* 2011.05.12 Ver 1.2
* 2011.05.29 Ver 1.3 增加Dropbox安装配置，删除过时内容  
 
每次重装的话都要安装很多软件，修改配置，我的记性不是很好，每次都要网上查，这次记下来这些资料备用

## 系统安装

### 全新安装
WIN+LINUX双系统的话，在WIN下面可以用easybcd启动ISO安装，当然，最简单的方式是直接刻盘光盘安装。要补充的一点是，如果安装时要使用lvm，就必须使用文本模式安装，图形模式虽然也可以，但比较复杂，而且容易出问题
单系统模式就简单了，直接使用整个硬盘，但是要自己控制分区方案的话，分区时要选择高级，手动分区

###保留数据重装
保留数据一般是指保留/home分区的数据，前提是你的/home是单独分区的。重装系统时选高级手动分区，只格式化/分区，/home分区直接挂载为新的/home分区(我不清楚lvm虚拟卷可不可直接识别和挂载)，不能格式化，为避免新系统的配置文件冲突，最好使用livecd先删除/home/username目录下的.开头的大量配置文件，如果nautilus里看不到，使用Ctrl+H就可以看到所有的隐藏文件，新系统的用户名最好和原来的一致，这样就可以减少转移文件的麻烦。更详细的说明可以参考这篇文章：http://ubuntuabc.com/123/?p=23

## 新系统配置

### 配置软件源
首先备份原有的软件源 
 
```
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
```

然后去 
 
```
http://wiki.ubuntu.org.cn/Qref/Source
```

挑选一个速度最快的源加入 sources.list 不要加入多个，反而影响索引速度  

#### 更新软件源  

```
sudo apt-get update
```

#### 更新系统

```
sudo apt-get dist-upgrade
```

#### Unity配置工具

```
sudo apt-get install compizconfig-settings-manager
```

#### PPA管理器
安装y-ppa manager

```
sudo add-apt-repository ppa:webupd8team/y-ppa-manager
sudo apt-get update
sudo apt-get install y-ppa-manager
```

#### 字体安装
直接双击字体，在查看对话框中点击安装即可
或者直接复制到~/.fonts目录就可以，不需要任何配置，非常简单
文件浏览右键管理员

```
sudo apt-get install nautilus-gksu
```

#### 设置中文MAN


```
alias cman='man -M /usr/share/man/zh_CN'
```

想用中文man时： cman command
想用英文man： man command

设置curl不检查SSL证书

```
echo 'insecure' &gt;&gt; ~/.curlrc
```
#### Unity配置

解除Unity的系统托盘显示限制
- 命令行方式：
解禁所有程序

```
gsettings set com.canonical.Unity.Panel systray-whitelist "['all']"
```

或者只解禁部分程序，把 YOUR_APPLICATION 替换成你需要解禁的程序

```
gsettings set com.canonical.Unity.Panel systray-whitelist "['JavaEmbeddedFrame', 'Mumble', 'Wine', 'Skype', 'hp-systray', 'YOUR_APPLICATION']"
```

- GUI 方式：
安装 dconf-tools

```
sudo apt-get install dconf-tools
```

在终端中输入 dconf-editor ，然后找到  desktop &gt; unity &gt; panel ，把 systray-whitelist 的值改为 ['all']


## 安装常用软件

####压缩和解压缩

```
sudo apt-get install unace unrar zip unzip p7zip-full p7zip-rar sharutils uudeview mpack lha arj cabextract file-roller
```

不要安装源里的rar，会乱码，p7zip-rar和unrar都不会乱码  

新版iBus输入法，自带的版本很老，可以用ppa安装最新版

```
sudo add-apt-repository ppa:shawn-p-huang/ppa
sudo apt-get update
sudo apt-get install ibus-gtk ibus-pinyin ibus-pinyin-db-open-phrase
```

####文本编辑器
自带的gedit其实够用，可以不安装其他的，需要的话也可以安装scribes，geany
需要修改一点gedit的配置才可以自动识别中文编码：运行gconf-editor，展开/apps/gedit-2/preferences/encodings节点，加入GB18030和BIG5-HKSCS

#### 浏览器
11.04自带最新的4.0

```
sudo add-apt-repository ppa:mozillateam/firefox-stable
sudo apt-get update &amp;&amp; sudo apt-get upgrade
```

如果没有安装Firefox可以直接

```
sudo apt-get install firefox
```

安装Chrome稳定版，需要直接去官网下载

```
wget https://dl-ssl.google.com/linux/direct/google-chrome-stable_current_i386.deb
sudo dpkg -i google-chrome-stable_current_i386.deb
```

Chrome 10以后的版本默认为难看的楷体，字体修改方法

```
http://imcn.me/html/y2011/3306.html
http://imcn.me/html/y2011/3386.html
http://www.linuxidc.com/Linux/2011-05/35375.htm
```

#### 文档阅读
增强自带的PDF阅读能力，增加pdf和其它格式的相互转换

```
sudo apt-get install poppler-data poppler-utils
```

#### 备份同步
安装dropbox
首先官网下载

```
https://www.dropbox.com/download?dl=packages/nautilus-dropbox_0.6.7_i386.deb
```

安装后还需要安装dropbox的daemon，这个无法直接下载，可以用vpn
或者直接用代理下载下面这个

```
http://www.getdropbox.com/download?plat=lnx.x86
```

然后直接在～目录解压，文件会解压到 ~/.dropbox-dist/ ，daemon就装好了，直接就可以运行
更详细的说明见：
http://freedomhui.com/?p=149
电子书制作和阅读

```
sudo apt-get install calibre fbreader
```

需要最新版的话可以自己下载源码编译安装

#### 图像处理

```
sudo apt-get install gimp inkscape imagemagick dia
```

#### 音乐播放

```
sudo add-apt-repository ppa:nilarimogard/webupd8
sudo apt-get update
sudo apt-get install audacious
```

MP3 GBK标签转码工具

```
sudo apt-get install python-mutagen
```

执行转码

```
find . -iname "*.mp3" -execdir mid3iconv -e gbk {} ;
```

#### 视频播放

```
sudo apt-get install vlc vlc-plugin-pulse mozilla-plugin-vlc
sudo apt-get install mplayer smplayer
```

#### 下载工具
最常见的wget和aria2c


```
sudo apt-get install wget aria2
```

推荐用下面的Firefox插件

```
sudo add-apt-repository ppa:plushuang-tw/uget-devel
sudo apt-get update
sudo apt-get install uget
```

特别推荐使用优蛋 linux版，很好用
下载地址

```
http://bbs.ylmf.net/forum.php?mod=viewthread&amp;tid=1916551
```

当然，用Firefox的话，有最强大的下载插件：DownThemAll，最好的下载工具

#### 安装Ubuntu Tweak

```
sudo add-apt-repository ppa:tualatrix/ppa
sudo apt-get update
sudo apt-get install ubuntu-tweak
```

## 桌面美化

#### 安装Faenza图标和equinox主题

```
sudo add-apt-repository ppa:tiheum/equinox
sudo apt-get update
sudo apt-get install faenza-icon-theme  gtk2-engines-equinox equinox-theme
```

#### 安装elementary主题

```
sudo add-apt-repository ppa:elementaryart/elementarydesktop
sudo apt-get update
sudo apt-get install elementary-theme elementary-icon-theme elementary-wallpapers
```

#### 安装nautilus-elementary

```
sudo add-apt-repository ppa:am-monkeyd/nautilus-elementary-ppa
sudo apt-get update
sudo apt-get dist-upgrade
```

打开/usr/share/themes/.gtkrc，注释掉这一行： `#include "apps/nautilus-elementary.rc"` 
重启 nautilus -q

#### 安装系统状态指示器

```
wget http://mh21.de/temp/indicator-multiload_0.1-0~5_i386.deb
sudo dpkg -i indicator-multiload_0.1-0~5_i386.deb
```

## 开发环境配置

#### 安装基本开发环境和版本工具

```
sudo apt-get install build-essential autoconf automake1.9 cvs subversion git mercurial
```

#### 安装JDK
需要在更新管理器里启用第三方软件源

```
sudo apt-get install sun-java6-jdk sun-java6-plugin
```

或者通过ppa安装

```
sudo add-apt-repository ppa:ferramroberto/java
sudo apt-get update
sudo apt-get install sun-java6-jdk sun-java6-plugin
```

#### 配置JDK


```
sudo update-alternatives --config java
```

解决Java程序乱码
注释掉此文件的所有行：

```
/usr/lib/jvm/java-6-sun/jre/lib/fonts/fonts.dir
http://wiki.ubuntu.org.cn/Java%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AE
```

#### 安装Android开发环境

```
sudo apt-get install git-core gnupg flex bison gperf build-essential zip curl zlib1g-dev libc6-dev x11proto-core-dev libx11-dev
```

#### 安装Eclipse
不建议直接使用源安装，不好控制
建议直接去官网下载压缩包，解压即可使用，如果放在/usr/local 或 /opt 分区的话，为使用方便，需要chown root:root -R  一下，不过强烈建议直接解压放在/home分区的某个目录，使用最方便，不用担心莫名其妙的权限问题
添加桌面启动器和面板启动器可以参考这里：  
<http://www.5dlinux.com/article/1/2008/linux_15330.html>
我的方法是直接 ln -s一个放在~/bin目录里

#### 安装Ruby
系统自带的是1.8.7，需要使用1.9.2的话必须自己编译安装，不过无论使用哪个版本，强烈推荐使用rvm安装和配置ruby环境，与系统隔离，使用和卸载都方便，官网的安装教程非常详细，推荐安装在～/目录

```
bash  < (curl -s https://rvm.beginrescueend.com/install/rvm)
```

在.bashrc的最后加入


```
[[ -s "$HOME/.rvm/scripts/rvm" ]] &amp;&amp; . "$HOME/.rvm/scripts/rvm"
rvm  install 1.9.2-p180
rvm 1.9.2-p180
rvm --default use 1.9.2
```

检查安装情况 `which ruby`

#### 安装Eclipse插件

```
ADT：
http://dl-ssl.google.com/android/eclipse/
https://dl-ssl.google.com/android/eclipse/
```

## 其它配置

#### 配置Evolution使用GMail
imap配置里用户名必须是包含@gmail.com的，要不然无法链接，而且很bt的是根本没有提示和错误说明 #ubuntu 还有imap服务器必须加端口号993 加密方式为SSL

#### 软件包备份还原
备份

```
sudo tar cizvf apt-backup.tar.gz /var/cache/apt/archives --exclude=/var/cache/apt/archives/partial/* --exclude=/var/cache/apt/archives/lock
```
还原

```
sudo apt-get update &amp;&amp; sudo tar xzvf backup.tar.gz -C /
```

## 其它软件

#### 图形方式编辑启动菜单

安装Grub Customizer

```
sudo add-apt-repository ppa:danielrichter2007/grub-customizer
sudo apt-get update
sudo apt-get install grub-customizer
```

#### 安装和配置openvpn
安装openvpn

```
sudo apt-get install openvpn

复制配置文件到 /etc/openvpn/ ，添加权限
cd /etc/openvpn/
sudo chmod +x *
连接openvpn
sudo openvpn --config /etc/openvpn/client.ovp --script-security 2
```

出现下列字样表示连接成功


```
openvpn --config /etc/openvpn/client.ovp --script-security 2
```

使用SSH Tunneling Proxy

<http://wiki.wowubuntu.com/blog/ubuntu_ssh_tunneling>
<http://wiki.wowubuntu.com/blog/gstm>

## 故障和问题
暂无