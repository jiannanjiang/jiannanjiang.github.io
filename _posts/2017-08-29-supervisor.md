---
layout: post
title: 使用进程守护程序supervisor 运行laravel的队列
date: 2017-08-29 14:23:55.000000000 +08:00
---


在用Laravel队列的时候，官方推荐了supervisor作为进程守护程序

它的好处是进程意外中止的时候可以重新拉起来

对于队列中运行时间较长的进程来说是非常实用的

>关于laravel队列的使用可以参考 http://laravelacademy.org/post/6922.html

### 安装

centos7下面可以直接用yum安装

`$ yum install python-setuptools`
`$ easy_install supervisor`
`$ echo_supervisord_conf > /etc/supervisord.conf`


### 配置

`$ vim /etc/supervisord.conf `

其中最后一行
```
[include]
files = relative/directory/*.ini
```
可以定义独立配置文件

这个是laravel官方给的模板
```
[program:laravel-worker1]
process_name=%(program_name)s_%(process_num)02d
command=php /home/wwwroot/site.webshowu.com/artisan queue:work redis --sleep=3 --tries=3 --daemon
autostart=true
autorestart=true
user=root
numprocs=3
redirect_stderr=true
stdout_logfile=/home/wwwlogs/worker1.log
```

下面是配置文件的详细解释
```
;*为必须填写项
;*[program:应用名称]
[program:cat]
;*命令路径,如果使用python启动的程序应该为 python /home/test.py, 
;不建议放入/home/user/, 对于非user用户一般情况下是不能访问
command=/bin/cat
;当numprocs为1时,process_name=%(program_name)s
;当numprocs>=2时,%(program_name)s_%(process_num)02d
process_name=%(program_name)s
;进程数量
numprocs=1
;执行目录,若有/home/supervisor_test/test1.py
;将directory设置成/home/supervisor_test
;则command只需设置成python test1.py
;否则command必须设置成绝对执行目录
directory=/tmp
;掩码:--- -w- -w-, 转换后rwx r-x w-x
umask=022
;优先级,值越高,最后启动,最先被关闭,默认值999
priority=999
;如果是true,当supervisor启动时,程序将会自动启动
autostart=true
;*自动重启
autorestart=true
;启动延时执行,默认1秒
startsecs=10
;启动尝试次数,默认3次
startretries=3
;当退出码是0,2时,执行重启,默认值0,2
exitcodes=0,2
;停止信号,默认TERM
;中断:INT(类似于Ctrl+C)(kill -INT pid),退出后会将写文件或日志(推荐)
;终止:TERM(kill -TERM pid)
;挂起:HUP(kill -HUP pid),注意与Ctrl+Z/kill -stop pid不同
;从容停止:QUIT(kill -QUIT pid)
;KILL, USR1, USR2其他见命令(kill -l),说明1
stopsignal=TERM
stopwaitsecs=10
;*以root用户执行
user=root
;重定向
redirect_stderr=false
stdout_logfile=/a/path
stdout_logfile_maxbytes=1MB
stdout_logfile_backups=10
stdout_capture_maxbytes=1MB
stderr_logfile=/a/path
stderr_logfile_maxbytes=1MB
stderr_logfile_backups=10
stderr_capture_maxbytes=1MB
;环境变量设置
environment=A="1",B="2"
serverurl=AUTO
```


inet_http_server 的配置

可以使用浏览器查看和控制进程状态

```
[inet_http_server]         ; inet (TCP) server disabled by default
port=*:9001          ; (ip_address:port specifier, *:port for all iface)
username=user              ; 用户名 (default is no username (open server))
password=123               ; 密码 (default is no password (open server))
```

其中username password 可以注释掉

注意要打开防火墙的9001端口

### 常用命令

启动
`$ supervisord -c /etc/supervisord.conf`

关闭
`$ supervisorctl shutdown`

重新载入配置
`$ supervisorctl reload`

### 参考文献

https://laravel-china.org/topics/2126/supervisor-installation-configuration-use