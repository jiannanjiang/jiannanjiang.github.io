---
layout: post
title: Laravel 队列
date: 2017-08-06 14:32:55.000000000 +08:00
---


## 驱动
在处理复杂逻辑的时候，可能有些程序运行时间比较长，但是又不能让用户一直等待，所以需要把一部分不需要实时处理的任务放到队列中。

Laravel提供了队列的处理方式

和缓存一样，Laravel的队列支持多种驱动

> Beanstalk，Amazon SQS， Redis

或者本地驱动 sync

数据库也是支持的，但是数据库里面建表

`$ php artisan queue:table`
`$ php artisan migrate`

如果用Redis，需要在config/database.php里面配置

'redis' => [
    'driver' => 'redis',
    'connection' => 'default',
    'queue' => '{default}',
    'retry_after' => 90,
],
下面是jobs表

```
DROP TABLE IF EXISTS `jobs`;
CREATE TABLE `jobs` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
  `queue` varchar(191) COLLATE utf8mb4_unicode_ci NOT NULL,
  `payload` longtext COLLATE utf8mb4_unicode_ci NOT NULL,
  `attempts` tinyint(3) unsigned NOT NULL,
  `reserved_at` int(10) unsigned DEFAULT NULL,
  `available_at` int(10) unsigned NOT NULL,
  `created_at` int(10) unsigned NOT NULL,
  PRIMARY KEY (`id`),
  KEY `jobs_queue_reserved_at_index` (`queue`,`reserved_at`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

```

如果使用以下几种队列驱动，需要安装相应的依赖：

> Amazon SQS: aws/aws-sdk-php ~3.0
> Beanstalkd: pda/pheanstalk ~3.0
> Redis: predis/predis ~1.0


## 创建任务

下面是官方文档中给的示例，放在app\Jobs目录下

```
<?php

namespace App\Jobs;

use App\Podcast;
use App\AudioProcessor;
use Illuminate\Bus\Queueable;
use Illuminate\Queue\SerializesModels;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Contracts\Queue\ShouldQueue;

class ProcessPodcast implements ShouldQueue
{
    use InteractsWithQueue, Queueable, SerializesModels;

    protected $podcast;

    //public $tries = 5;

    //public $timeout = 120;
    /**
     * 创建任务实例
     *
     * @param  Podcast  $podcast
     * @return void
     */
    public function __construct(Podcast $podcast)
    {
        $this->podcast = $podcast;
    }

    /**
     * 执行任务
     *
     * @param  AudioProcessor  $processor
     * @return void
     */
    public function handle(AudioProcessor $processor)
    {
        // 处理上传的播客…
    }
}
```

也可以直接用命令生成 
`$ php artisan make:job SendReminderEmail`

## 分发任务

`dispatch(new ProcessPodcast($podcast));`


## 延时分发

```
$job = (new ProcessPodcast($pocast))->delay(Carbon::now()->addMinutes(10));
dispatch($job);
```

onConnection 指定连接
onQueue 指定队列


## 运行队列

`php artisan queue:work`

这个是需要守护进程持续执行

官方推荐的是 supervisor 

一般我们在linux上运行后台程序用的是nohup，但是这个不是很稳定，也没有重试机制

supervisor 可以监控进程状态，失败了还能重启

### 安装和配置

`$ yum install python-setuptools`
`$ easy_install supervisor`

`$ echo_supervisord_conf > /etc/supervisord.conf`
`$ mkdir -p /etc/supervisor/conf.d/`

编辑/etc/supervisord.conf

```
[include]
files = /etc/supervisor/conf.d/*.conf
```

新建laravel 的队列配置

`$ touch /etc/supervisor/conf.d/laravel-work.conf`

```
[program:laravel-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /home/forge/app.com/artisan queue:work sqs --sleep=3 --tries=3
autostart=true
autorestart=true
user=forge
numprocs=8
redirect_stderr=true
stdout_logfile=/home/forge/app.com/worker.log
```


### 启动服务

`$ supervisord -c /etc/supervisord.conf`

如果配置文件有变化可以重新启动

`$ supervisorctl`
`$ supervisorctl> reread`
`$ supervisorctl> update`
`$ supervisorctl> start laravel-worker:*`

其中 supervisorctl 命令执行后可以看到当前正在跑的进程

重启命令
`$ supervisorctl reload`

如果要配置开机启动 需要
参考http://www.jianshu.com/p/e1c3e6fbae80 未完待续

参考文章

> https://laravel-china.org/topics/2126/supervisor-installation-configuration-use
> http://blog.csdn.net/realghost/article/details/52783100