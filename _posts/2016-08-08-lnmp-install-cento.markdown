---
layout: post
title: Centos6 下安装Nginx+Mysql+PHP
date: 2016-08-08 17:32:55.000000000 +08:00
---


### 安装nginx

#### 添加源

wget http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm

#### 安装源库

chmod +x nginx-release-centos-7-0.el7.ngx.noarch.rpm
rpm -i nginx-release-centos-7-0.el7.ngx.noarch.rpm

#### 安装nginx

yum -y install nginx

#### 安装完成后的默认配置文件路径

默认nginx配置文件: /etc/nginx/nginx.conf 【nginx主要的配置文件】
默认nginx的ssl配置文件: /etc/nginx/conf.d/ssl.conf 【配置SSL证书的，也可以并入到nginx.conf文件里】
默认nginx的虚拟主机配置文件: /etc/nginx/conf.d/virtual.conf 【如同Apache的虚拟主机配置，也可以并入到nginx.conf文件里】
默认的web_root文件夹路径: /usr/share/nginx/html 【web目录夹，放置Magento主程序】

#### 配置iptables

service iptables stop

#### 启动nginx

service nginx start

#### 设置开启启动：

chkconfig nginx on

#### 如果安装以后service nginx start报permission denied

需要执行

`$ vim /etc/selinux/config`

SELINUX=disabled

`$ setenforce 0`


打开IP地址 可见“Welcome to nginx!”表示安装成功。

另外：如果nginx设置目录在其他路径，一定要给o+x的权限，否则会报403forbidden

### 安装PHP


```
$ rpm -Uvh https://mirror.webtatic.com/yum/el7/epel-release.rpm
$ rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm

$ yum install php70w.x86_64 php70w-cli.x86_64 php70w-common.x86_64 php70w-gd.x86_64 php70w-ldap.x86_64 php70w-mbstring.x86_64 php70w-mcrypt.x86_64 php70w-mysql.x86_64 php70w-pdo.x86_64

$ yum install php70w-fpm

$ yum -y install php70w-xml.x86_64


$ systemctl enable php-fpm.service

```

#### 安装PHP组件，使PHP支持 MySQL、PHP支持FastCGI模式(补充，非必装)

yum install php-mysql php-gd libjpeg* php-imap php-ldap php-odbc php-pear php-xml php-xmlrpc php-mbstring php-mcrypt php-bcmath php-mhash libmcrypt libmcrypt-devel php-fpm php-intl

#### 安装gcc和g++

`$ yum -y install gcc`

`$ yum -y install gcc-c++`

#### 安装bcmath扩展

`$ yum -y install php70w-bcmath`
    
#### 设置开机启动

chkconfig php-fpm on 

#### 配置PHP支持Nginx

vim /etc/php-fpm.d/www.conf

修改user和group为nginx

vim /etc/nginx/nginx.conf
放在http里面

fastcgi_connect_timeout 300;
fastcgi_send_timeout 300;
fastcgi_read_timeout 300;
fastcgi_buffer_size 128k;
fastcgi_buffers 4 128k;
fastcgi_busy_buffers_size 128k;
fastcgi_temp_file_write_size 128k;

upstream fastcgi_backend {
    server  127.0.0.1:9000;
}

下面是demo

```
server
{
  listen 80;
  server_name www.dev.com;
  root /home/www/dev/public;
  index index.php index.html index.htm;
location / {
    try_files $uri $uri/ /index.php?$query_string;
}


  location = /favicon.ico {
    log_not_found off;
    access_log off;
  }


#  location ~* \.(jpg|jpeg|gif|css|png|js|ico)$ { aio on; directio 512k; access_log off; expires 7d; break; }

  location ~ \.php$ {
    fastcgi_index                   index.php;
    fastcgi_pass                    127.0.0.1:9000;
    include                         fastcgi_params;
    fastcgi_intercept_errors        On;
    fastcgi_param SCRIPT_FILENAME   $document_root$fastcgi_script_name;
    fastcgi_ignore_client_abort     On;
    fastcgi_buffer_size             128k;
    fastcgi_buffers                 4 128k;
  }
}
```


### 安装Mysql

另外一篇文章有介绍