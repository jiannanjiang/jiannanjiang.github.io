---
layout: post
title: nginx 安装 https
date: 2017-08-11 10:39:49.000000000 +08:00
---

### 操作流程

+ 第一步，生成csr文件和key文件
```
$ cd /etc/ssl/private
$ openssl req -new -newkey rsa:2048 -sha256 -nodes -out maketea_loc.csr -keyout maketea_loc.key -subj "/C=CN/ST=Beijing/L=Beijing/O=maketea Inc./OU=Web Security/CN=*.maketea.loc"
```

+ 第二步，提交csr文件到CA机构

+ 第三步，拿到crt文件

+ 第四步，maketea_loc.csr maketea_loc.key maketea_loc.crt 三个文件放到/etc/ssl/private目录下

+ 第五步，修改nginx文件
```
server {  
    listen 80; #也可以不监听80端口 看需要
    listen 443 ssl;
    server_name www.maketea.loc;

    ssl on;
    ssl_certificate /etc/ssl/private/maketea_loc.crt;
    ssl_certificate_key /etc/ssl/private/maketea_loc.key;
}
```

一般的SHA-1形式https就配置好了

为了更安全 ，可以考虑使用迪菲－赫尔曼密钥交换

`$ cd /etc/ssl/certs`  
`$ openssl dhparam -out dhparam.pem 2048`

然后在nginx ssl配置的后面加上下面的配置
```
ssl_prefer_server_ciphers on;
ssl_dhparam /etc/ssl/certs/dhparam.pem;
ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
ssl_ciphers "EECDH+ECDSA+AESGCM EECDH+aRSA+AESGCM EECDH+ECDSA+SHA384 EECDH+ECDSA+SHA256 EECDH+aRSA+SHA384 EECDH+aRSA+SHA256 EECDH+aRSA+RC4 EECDH EDH+aRSA !aNULL !eNULL !LOW !3DES !MD5 !EXP !PSK !SRP !DSS !RC4";
keepalive_timeout 70;
ssl_session_cache shared:SSL:10m;
ssl_session_timeout 10m;
```
同时，如果是全站 HTTPS 并且不考虑 HTTP 的话，可以加入 HSTS 告诉你的浏览器本网站全站加密，并且强制用 HTTPS 访问
```
add_header Strict-Transport-Security max-age=63072000;
add_header X-Frame-Options DENY;
add_header X-Content-Type-Options nosniff;
```
同时也可以单独开一个 Nginx 配置，把 HTTP 的访问请求都用 301 跳转到 HTTPS
```
server {  
        listen 80;
        server_name  www.maketea.loc;
        return 301 https://www.maketea.loc$request_uri;
}
```
### 颁发证书的机构

目前一般市面上针对中小站长和企业的 SSL 证书颁发机构有：

StartSSL

Comodo / 子品牌 Positive SSL

GlobalSign / 子品牌 AlphaSSL

GeoTrust / 子品牌 RapidSSL

其中 Postivie SSL、AlphaSSL、RapidSSL 等都是子品牌，一般都是三级四级证书，所以你会需要增加 CA 证书链到你的 CRT 文件里。

以 Comodo Positive SSL 为例，需要串联 CA 证书，假设你的域名是 example.com

那么，串联的命令是

`$ cat example_com.crt COMODORSADomainValidationSecureServerCA.crt COMODORSAAddTrustCA.crt AddTrustExternalCARoot.crt > example_com.signed.crt`

在 Nginx 配置里使用 example_com.signed.crt 即可

### 级联问题

有些时候，由第三方机构签发的证书在浏览器上是OK的，但是到了例如安卓端会不认这个证书，Nginx官方是这样说的

```
Some browsers may complain about a certificate signed by a well-known certificate authority, while other browsers may accept the certificate without issues. This occurs because the issuing authority has signed the server certificate using an intermediate certificate that is not present in the certificate base of well-known trusted certificate authorities which is distributed with a particular browser. In this case the authority provides a bundle of chained certificates which should be concatenated to the signed server certificate. The server certificate must appear before the chained certificates in the combined file
```

就是说需要有个中间证书

一般类似godaddy这种机构会提供这个证书给你，你要做的就是把这个串放在crt文件的后面，做成一个新的crt，就可以正常使用了

`$ cat nginx.crt bundle.crt > nginx.chain.crt`


### 测试的时候自签证书的方法


`$ openssl ca -in nginx.csr -out nginx.crt`

### 参考文献
https://s.how/nginx-ssl/