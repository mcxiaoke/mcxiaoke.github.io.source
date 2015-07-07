title: 在VPS上配置Nginx和PHP
date: 2014-06-30 14:00
author: mcxiaoke
categories: 
- Web
tags: 
- Nginx
- PHP
- VPS
#slug: vps-nginx-php5-fpm-config
---
之前买了一个廉价VPS，系统是Ubuntu 12.04，一直没有用上，最近装上Nginx和PHP，下面就是配置记录。

首先当然是安装各种必备的工具：

```
sudo apt-get install php5-common php5-cli php5-fpm
sudo apt-get install nginx
```

下面是nginx的配置文件，位置是 `/etc/nginx/sites-available/mcxiaoke.com`

这里假设我的域名是 `mcxiaoke.com`

```
# 配置HTTP站点
server {
listen 80;
# 站点根目录
root /var/www/;

index index.php index.html index.htm index.php;

server_name mcxiaoke.com;

# 重定向所有HTTP到HTTPS
rewrite ^(.*)$ https://$host$1 permanent;

location / {
try_files $uri $uri/ /index.php$is_args$args;
}

error_page 404 /404.html;

# redirect server error pages to the static page /50x.html
#
error_page 500 502 503 504 /50x.html;
location = /50x.html {
root /usr/share/nginx/www;
}

# 配置PHP
location ~ \.php$ {
try_files $uri =404;
fastcgi_pass 127.0.0.1:9000;
include fastcgi_params;
}

}

# 配置HTTPS站点
server {
listen 443;
server_name mcxiaoke.com;
#站点根目录
root /var/www/;
index index.html index.htm index.php;

ssl on;
# 这里是网站的SSL证书
ssl_certificate /etc/nginx/ssl/ssl.crt;
ssl_certificate_key /etc/nginx/ssl/ssl.key;

ssl_session_timeout 10m;

ssl_ciphers RC4:HIGH:!aNULL:!MD5;
ssl_prefer_server_ciphers on;

#ssl_protocols SSLv3 TLSv1;
#ssl_ciphers ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv3:+EXP;
#ssl_prefer_server_ciphers on;

location / {
try_files $uri $uri/ /index.php$is_args$args;
}

error_page 404 /404.html;

# redirect server error pages to the static page /50x.html
#
error_page 500 502 503 504 /50x.html;
location = /50x.html {
root /usr/share/nginx/www;
}

# 配置PHP
location ~ \.php$ {
try_files $uri =404;
fastcgi_pass 127.0.0.1:9000;
include fastcgi_params;
}

}
```

配置好之后可以在某个目录下放一个info.php文件，内容如下：

```
<?php phpinfo(); ?>;
```

然后重启服务：

```
sudo service php5-fpm restart
sudo service nginx restart
```

如果能正常访问，能看到服务器的PHP配置信息，就表明配置没有问题。

参考文章：  
[How To Install LEMP stack on Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-install-linux-nginx-mysql-php-lemp-stack-on-ubuntu-14-04)  
[Mac OSX 10.9搭建nginx+mysql+php-fpm环境](http://askubuntu.com/questions/134666/what-is-the-easiest-way-to-enable-php-on-nginx)