Title: 转移Windows的Users文件夹
Date: 2010-10-10 14:19
Author: mcxiaoke
Category: Windows
Tags: Windows
Slug: windows-7-users-folder-robocopy

转移用户文件夹，默认是在C:Users

## 安装系统时可用

在安装Win7的过程中，要求输入用户名及密码的时候，先不如输入任何信息，按“Shift+F10”呼出命令行窗口，输入以下命令：  

```
robocopy "C:Users" "D:Users" /E /COPYALL /XJ  
rmdir "C:Users" /S /Q  
mklink /J "C:Users" "D:Users"  
```

而后关闭命令行窗口，继续安装直至完成。

## 安装完成后

0. 关闭所有应用程序；  
1. 在我的电脑（计算机）上点右键，选择管理，进入计算机管理，展开本地用户和组选项；  
2. 鼠标点击“Administrator”，选择属性，而后在随后的对话框中去掉“帐户已禁用”之前的勾；  
3. 注销当前用户（注意，不是“切换用户”），而后以“Administrator”登录  
4. 打开命令行窗口，输入以下命令：  

    ```
    robocopy "C:Users" "D:Users" /E /COPYALL /XJ /XD "C:UsersAdministrator"  
    ```

5. 注销Administrator，重新用你的用户名登录Windows
7，而后到“计算机管理器”里禁用Administrator；  
6. 以管理员身份打开一个DOS窗口，输入以下命令：  
    ```
    rmdir "C:Users" /S /Q  
    mklink /J "C:Users" "D:Users"  
    ```
    
7. 完成

robocopy是Windows 7自带的一个很强大的工具，XP用户可在此下载：  
[RobocopyGUI](http://www.brothersoft.com/robocopy-gui-105335.html)  

微软MSDN的介绍：<http://technet.microsoft.com/en-us/library/cc733145(WS.10).aspx>  
Robocopy的介绍：<http://en.wikipedia.org/wiki/Robocopy>  
Robocopy的用法：<http://ss64.com/nt/robocopy.html>  

Robocopy的教程两篇：  
<http://marui.blog.51cto.com/1034148/296473>  
<http://marui.blog.51cto.com/1034148/297397>

Windows 7完整安装实例：<http://isouth.org/archives/294.html>

