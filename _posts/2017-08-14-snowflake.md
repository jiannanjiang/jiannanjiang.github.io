---
layout: post
title: 使用SnowFlake算法生成唯一ID
date: 2017-08-14 11:31:49.000000000 +08:00
---



> 前言：最近需要做一套CMS系统，由于功能比较单一，而且要求灵活，所以放弃了WP这样的成熟系统，自己做一套相对简单一点的。文章的详情页URL想要做成url伪静态的格式即xxx.html 其中xxx考虑过直接用自增主键，但是感觉这样有点暴露文章数量，有同学说可以把初始值设高一点，可是还是可以通过ID差算出一段时间内的文章数量，所以需要一种可以生成唯一ID的算法。    

考虑过的方法有

- 直接用时间戳，或者以此衍生的一系列方法

- Mysql自带的uuid

以上两种方法都可以查到就不多做解释了

最终选择了Twitter的SnowFlake算法

这个算法的好处很简单可以在每秒产生约400W个不同的16位数字ID(10进制)

原理很简单

ID由64bit组成

其中 第一个bit空缺

41bit用于存放毫秒级时间戳

10bit用于存放机器id

12bit用于存放自增ID

> 除了最高位bit标记为不可用以外，其余三组bit占位均可浮动，看具体的业务需求而定。默认情况下41bit的时间戳可以支持该算法使用到2082年，10bit的工作机器id可以支持1023台机器，序列号支持1毫秒产生4095个自增序列id。


下面是PHP源码

```
<?php
namespace App\Services;

abstract class Particle {
    const EPOCH = 1479533469598;
    const max12bit = 4095;
    const max41bit = 1099511627775;

    static $machineId = null;

    public static function machineId($mId = 0) {
        self::$machineId = $mId;
    }

    public static function generateParticle() {
        /*
        * Time - 42 bits
        */
        $time = floor(microtime(true) * 1000);

        /*
        * Substract custom epoch from current time
        */
        $time -= self::EPOCH;

        /*
        * Create a base and add time to it
        */
        $base = decbin(self::max41bit + $time);


        /*
        * Configured machine id - 10 bits - up to 1024 machines
        */
        if(!self::$machineId) {
            $machineid = self::$machineId;
        } else {
            $machineid = str_pad(decbin(self::$machineId), 10, "0", STR_PAD_LEFT);
        }
        
        /*
        * sequence number - 12 bits - up to 4096 random numbers per machine
        */
        $random = str_pad(decbin(mt_rand(0, self::max12bit)), 12, "0", STR_PAD_LEFT);

        /*
        * Pack
        */
        $base = $base.$machineid.$random;

        /*
        * Return unique time id no
        */
        return bindec($base);
    }

    public static function timeFromParticle($particle) {
        /*
        * Return time
        */
        return bindec(substr(decbin($particle),0,41)) - self::max41bit + self::EPOCH;
    }
}

?>


```


调用方法如下

` Particle::generateParticle($machineId);//生成ID`
` Particle::timeFromParticle($particle);//反向计算时间戳`

这里我做了改良 如果机器ID传0 就会去掉这10bit 因为有些时候我们可能用不到这么多ID

