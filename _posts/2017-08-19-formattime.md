---
layout: post
title: PHP格式化时间戳函数分享
date: 2017-08-19 19:22:49.000000000 +08:00
---

CMS中一般显示时间比较新的文章需要显示几分钟前，几天前这样，我封装好了两个函数可以直接拿来用

下面是封装好的方法

```
function formatTime($time) {
        $time = (int) substr($time, 0, 10);
        $int = time() - $time;
        $str = '';
        if ($int <= 2){
            $str = sprintf('刚刚', $int);
        }elseif ($int < 60){
            $str = sprintf('%d秒前', $int);
        }elseif ($int < 3600){
            $str = sprintf('%d分钟前', floor($int / 60));
        }elseif ($int < 86400){
            $str = sprintf('%d小时前', floor($int / 3600));
        }elseif ($int < 2592000){
            $str = sprintf('%d天前', floor($int / 86400));
        }else{
            $str = date('Y-m-d H:i:s', $time);
        }
        return $str;
    }
```


或者 更详细的

```
    public static function formatTime($time)
    {
        if (is_int($time)) {
            $time = intval($time);
        } elseif ($time instanceof Carbon) {
            $time = intval(strtotime($time));
        } else {
            return '';
        }
        $ctime = time();
        $t = $ctime - $time; //时间差 （秒）

        if ($t < 0) {
            return date('Y-m-d', $time);
        }

        $y = intval(date('Y', $ctime) - date('Y', $time));//是否跨年


        if ($t == 0) {
            $text = '刚刚';
        } elseif ($t < 60) {//一分钟内
            $text = $t . '秒前';
        } elseif ($t < 3600) {//一小时内
            $text = floor($t / 60) . '分钟前';
        } elseif ($t < 86400) {//一天内
            $text = floor($t / 3600) . '小时前'; // 一天内
        } elseif ($t < 2592000) {//30天内
            if ($time > strtotime(date('Ymd', strtotime("-1 day")))) {
                $text = '昨天';
            } elseif ($time > strtotime(date('Ymd', strtotime("-2 days")))) {
                $text = '前天';
            } else {
                $text = floor($t / 86400) . '天前';
            }
        } elseif ($t < 31536000 && $y == 0) {//一年内 不跨年
            $m = date('m', $ctime) - date('m', $time) - 1;

            if ($m == 0) {
                $text = floor($t / 86400) . '天前';
            } else {
                $text = $m . '个月前';
            }
        } elseif ($t < 31536000 && $y > 0) {//一年内 跨年
            $text = (12 - date('m', $time) + date('m', $ctime)) . '个月前';
        } else {
            $text = (date('Y', $ctime) - date('Y', $time)) . '年前';
        }

        return $text;
    }
```


