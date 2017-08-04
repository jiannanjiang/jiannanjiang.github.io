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

#### 安装PHP组件，使PHP支持 MySQL、PHP支持FastCGI模式

yum install php-mysql php-gd libjpeg* php-imap php-ldap php-odbc php-pear php-xml php-xmlrpc php-mbstring php-mcrypt php-bcmath php-mhash libmcrypt libmcrypt-devel php-fpm php-intl

` yum -y install php70w-xml.x86_64`

#### 设置开机启动

chkconfig php-fpm on 

#### 配置PHP支持Nginx

vim /etc/php-fpm.d/www.conf

修改user和group为nginx

vim  /etc/nginx/conf.d/default.conf
index 增加index.php
配置用户为user nginx nginx;

配置fastCGI监听9000端口

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



### 安装Mysql

另外一篇文章有介绍

### 升级PHP到5.6


rpm -Uvh http://ftp.iij.ad.jp/pub/linux/fedora/epel/6/x86_64/epel-release-6-8.noarch.rpm


rpm -Uvh http://rpms.famillecollet.com/enterprise/remi-release-6.rpm


yum list --enablerepo=remi --enablerepo=remi-php56 | grep php

yum install --enablerepo=remi --enablerepo=remi-php56 php php-opcache php-devel php-mbstring php-mcrypt php-mysqlnd php-phpunit-PHPUnit php-pecl-xdebug php-pecl-xhprof

yum -y install php56u php56u-opcache php56u-xml php56u-mcrypt php56u-gd php56u-devel php56u-mysql php56u-intl php56u-mbstring php56u-bcmath





### Magento官方给出的centos安装PHP方法

yum -y update
yum -y install epel-release
wget https://dl.fedoraproject.org/pub/epel/epel-release-latest-6.noarch.rpm
wget https://centos6.iuscommunity.org/ius-release.rpm
rpm -Uvh ius-release*.rpm
yum -y update
yum -y install php56u php56u-opcache php56u-xml php56u-mcrypt php56u-gd php56u-devel php56u-mysql php56u-intl php56u-mbstring php56u-bcmath php56u-fpm

太慢 需要翻墙

另外PHP装上之后 可能缺少某些扩展 比如intl

安装的话可以用pecl

yum install libicu

yum install libicu-devel.x86_64

/usr/bin/pecl install intl

如果报错了，是因为xml扩展不能加载，重新安装pear可以解决

yum erase php-pear

rpm -Uvh http://ftp.iij.ad.jp/pub/linux/fedora/epel/6/x86_64/epel-release-6-8.noarch.rpm

rpm -Uvh http://rpms.famillecollet.com/enterprise/remi-release-6.rpm

yum install --enablerepo=remi --enablerepo=remi-php56 php-pear

最后在php.ini里面加上 extension=intl.so就可以了