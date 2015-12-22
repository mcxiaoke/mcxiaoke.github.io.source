title: Ubuntu上Nginx服务配置笔记
date: 2015-12-22 16:00
author: mcxiaoke
categories: 
- Web
tags: 
- Nginx
- PHP
- VPS
---
Ubuntu 14.04系统上安装和配置Nginx/MySQL/PHP/Python的一些笔记。

* 初次发布 - 2014-06-30 14:00
* 更新时间 - 2015-12-22 16:00

## SSH配置

登录服务器，配置SSH

```bash
# SSH登录
ssh root@8.8.8.8 # 这里替换为你服务器的IP
```

添加用户

```bash
adduser mcxiaoke #创建名为mcxiaoke的新用户
gpasswd -a mcxiaoke sudo #将用户mcxiaoke添加到sudo用户组
```

生成公钥私钥对

```bash
# 如果本地已经有，可以忽略这一步
ssh-keygen
#默认存放在/Users/username/.ssh/[id_rsa, id_ras.pub]
# id_rsa是私钥，id_rsa.pub是公钥
```

复制公钥到服务器

```bash
# 如果本地有ssh-copy-id
ssh-copy-id mcxiaoke@8.8.8.8
# 你的公钥会被添加到远程服务器的 .ssh/authorized_keys
```

手动服务公钥到服务器

```bash
# 查看你的公钥
cat ~/.ssh/id_rsa.pub

# 以下命令在服务器上执行
su - mcxiaoke #切换用户
mkdir .ssh
chmod 700 .ssh #更改权限
vim .ssh/authorized_keys #也可以用nano
# 将刚刚打印出来的公钥文本粘贴进来，保存
chmod 600 .ssh/authorized_keys #更改权限
```

使用公钥登录

```bash
# 经过以上配置，你已经可以使用公钥登录了
ssh mcxiaoke@8.8.8.8
# 如果不是默认22端口，可以用 -p2222 这样指定端口号
```

修改SSH配置

```bash
vim /etc/ssh/sshd_config
# 可根据需要修改
Port 2222 #修改默认端口
PermitRootLogin no #禁止ROOT登录
PermitEmptyPasswords no #禁止空密码
PasswordAuthentication no #禁止密码登录
```

重启SSH服务

```bash
service ssh restart

#之后就不能使用密码登录了
ssh -p2222 mcxiaoke@8.8.8.8
```

## 软件包

```bash
# 如果是root用户不需要sudo
# 部分系统需要安装sudo
apt-get install sudo

# 更新服务器
sudo apt-get update

# 安装常用包
sudo apt-get install curl wget vim git
```

安装LEMP软件包

```
#安装nginx
sudo apt-get install nginx

# 测试nginx安装成功，假设IP为 8.8.8.8
# 浏览器访问 http://8.8.8.8 确认看到 Welcom to nginx!

# 安装MySQL，可选
sudo apt-get install mysql-server
# MySQL初始化配置
sudo mysql_install_db
# MySQL安全配置
sudo mysql_secure_installation

# 安装PHP，可选
sudo apt-get install php5-fpm
# sudo apt-get install php5-mysql
# 修改PHP配置
sudo vim /etc/php5/fpm/php.ini
# 修改这一行 cgi.fix_pathinfo=0
# 重启PHP服务
sudo service php5-fpm restart
```

## Nginx配置

配置Nginx支持PHP

```bash
sudo vim /etc/nginx/sites-available/default
```

```nginx
server {
    listen 80 default_server;
    listen [::]:80 default_server ipv6only=on;
    
    # 如果要支持HTTPS，修改这里
    # 可以使用 https://letsencrypt.org 的免费SSL证书
    #listen 443 ssl;
    #ssl_certificate     www.example.com.crt;
    #ssl_certificate_key www.example.com.key;
    #ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
    #ssl_ciphers         HIGH:!aNULL:!MD5;
    
    # 重定向所有HTTP到HTTPS
	# rewrite ^(.*)$ https://$host$1 permanent;

	# 网站根目录，根据需要修改
    root /usr/share/nginx/html;
    # 增加index.php
    index index.php index.html index.htm;

	# 假设域名是 ssl.mcxiaoke.com
    server_name ssl.mcxiaoke.com; #绑定域名

    location / {
        try_files $uri $uri/ =404;
    }

    error_page 404 /404.html;
    error_page 500 502 503 504 /50x.html;
    location = /50x.html {
        root /usr/share/nginx/html;
    }

	#支持php-fpm的配置
    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass unix:/var/run/php5-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```


配置完成后，测试一下

```bash
# 重启nginx服务
sudo service nginx restart

# 增加phpinfo
sudo vim /usr/share/nginx/html/info.php
# 写入内容： <?php phpinfo(); ?>

# 重启服务
sudo service php5-fpm restart
sudo service nginx restart
```

浏览器访问 http://ssl.mcxiaoke.com/info.php ,如果出现了PHP信息表格，表明配置没有问题，安全起见，建议移除info.php文件。

nginx也支持自动列目录文件的功能

```nginx
location /public {
	autoindex on;
	autoindex_exact_size off;
	autoindex_localtime on;
}
```

## Flask配置

安装软件包

```bash
sudo apt-get install build-essential python-dev python-pip
sudo pip install uwsgi virtualenv flask
```

创建示例应用

```bash
virtualenv myapp
cd myapp
source bin/activate
pip install uwsgi flask
```

创建一个简单的Flask应用，文件名 `myapp.py`

```python
from flask import Flask
app = Flask(__name__)
@app.route('/')
def helloworld():
    return 'hello, world.'
```

## Gunicorn

```bash
# 安装gunicorn
pip install gunicorn
#启动应用
gunicorn myapp:app -b localhost:8000
```

或者创建一个 `gunicorn_start` 的`bash`脚本文件

```bash
#!/usr/bin/env bash

NAME="HelloFlask"
# 替换为你的实际路径
FLASKDIR=/home/mcxiaoke/myapp
VENVDIR=/home/mcxiaoke/myapp
SOCKFILE=//home/mcxiaoke/myapp/app.sock
USER=mcxiaoke
GROUP=mcxiaoke
NUM_WORKERS=1

echo "Starting $NAME"

# activate the virtualenv
cd $VENVDIR
source bin/activate

export PYTHONPATH=$FLASKDIR:$PYTHONPATH

# Create the run directory if it doesn't exist
RUNDIR=$(dirname $SOCKFILE)
test -d $RUNDIR || mkdir -p $RUNDIR

# Start your unicorn
exec gunicorn myapp:app -b 127.0.0.1:8000 \
  --name $NAME \
  --workers $NUM_WORKERS \
  --user=$USER --group=$GROUP \
  --log-level=debug \
  --bind=unix:$SOCKFILE
```

执行权限

```bash
chmod a+x gunicorn_start
# 测试一下
./gunicorn_start
```

创建应用的nginx站点配置

```nginx
# sudo vim /etc/nginx/sites-available/myapp
server {
	listen 80 default_server;
	server_name myapp.mcxiaoke.com;
	location / {
        proxy_pass http://127.0.0.1:8000;
        #proxy_set_header Host $host;
        #proxy_set_header X-Real-IP $remote_addr;
	 }
	 
	 #location /static {
     #   alias  /home/mcxiaoke/myapp/static/;
     #}
}
```

测试一下

```bash
# 启用站点
sudo ln -s /etc/nginx/sites-available/myapp /etc/nginx/sites-enabled/
# 重启nginx
sudo service nginx restart
```

打开浏览器，如果能看到页面就说明没问题了。

## Supervisor

```bash
# 安装
sudo apt-get install supervisor
```

创建一个supervisor配置文件

```ini
[program:myapp]
command = gunicorn myapp:app -b localhost:8000
directory = /home/mcxiaoke/myapp
user = mcxiaoke
```

读取并执行任务

```bash
sudo pkill gunicorn
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl start myapp
```


## 参考资料

[How To Install LEMP stack on Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-install-linux-nginx-mysql-php-lemp-stack-on-ubuntu-14-04)  
[How To Set Up Nginx Server Blocks on Ubuntu 14.04 LTS](https://www.digitalocean.com/community/tutorials/how-to-set-up-nginx-server-blocks-virtual-hosts-on-ubuntu-14-04-lts)  
[How to Configure Nginx](https://www.linode.com/docs/websites/nginx/how-to-configure-nginx)    
[Nginx Full Example Configuration](https://www.nginx.com/resources/wiki/start/topics/examples/full/)   
[Kickstarting Flask on Ubuntu - Setup and Deployment](https://realpython.com/blog/python/kickstarting-flask-on-ubuntu-setup-and-deployment/)  
