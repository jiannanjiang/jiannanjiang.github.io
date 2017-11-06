---
layout: post
title: Laravel Blade模板的嵌套方法
date: 2017-08-24 13:47:55.000000000 +08:00
---


Laravel自带的Blade模板很强大也很方便

我们使用模板一般除了传递变量以外还有一个重要的用途就是嵌套

通过嵌套我们可以把公共的部分单独拉出来，在需要的地方引入避免重复劳动

根据官方文档我们可以知道模板常用命令有下面这几个

@section 定义

@yield 展示

@extends 继承

@include 引入


另外@if @else @while @unlesss等控制相关这里就不赘述了


### 比较常用的模板结构

这里举个实际的例子

比如我们 有三个页面，首页，列表页和详情页，分别是index,list,detail

在这三个页面中需要共同使用的部分有 页面的外围template,导航栏部分head

其中template用来放整个页面的框架布局

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8" />
    <title>测试</title>
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width,initial-scale=1,minimum-scale=1,maximum-scale=1,user-scalable=no" />
    <meta http-equiv="Content-type" content="text/html; charset=utf-8">
    <meta content="" name="description" />
    <meta content="" name="author" />
    <meta name="csrf-token" content="{{ csrf_token() }}" />
    <link rel='stylesheet' id='_common-css'  href='/css/common.css?ver=11.1' type='text/css' media='all' />
    <script type='text/javascript' src='/js/jquery.min.js'></script>
</head>

<body>
<section class="container">
    @yield('head')
    @yield('content')
</section>
</body>
</html>
```
我们在定义了主体容器container的同时，用@yield命令告诉模板我们要在这里放哪些section

这里我们虽然定义了templade作为整体布局，但是我们在控制器里并不能把view指向template而是要指向到具体的页面
比如首页
```
return view('index');
```

index模板页面如下
```
@extends('layouts.template')
@extends('layouts.head')
@section('content')
这里是页面的实际内容
@stop
```
index页面做了什么呢？
首先它把所有需要用的模板引入了进来，包括template和head
然后它有定义了一个叫content的section。

head模板页面如下
```
@section('head')
    <header class="header">
        <div class="container">
            <ul>
                <li>菜单1</li>
                <li>菜单2</li>
                <li>菜单3</li>
            <ul>
        </div>
    </header>
@stop
```

至此，一套基本可用的模板就搭建好了。
目录结构是这样的
```
resources/
    views/
        index.blade.php
        list.blade.php
        detail.blade.php
        layouts/
            template.blade.php
            head.blade.php
```

流程是这样的
- Controller指向模板index.blade.php
- index模板引入template模板和head模板，并定义content section
- template模板展示自身内容并展示对应的section内容
- 其中index负责引入所有模板，template负责整体结构和展示，index和head模板负责定义section里面的内容

### 特殊嵌套

还有一种比较特殊的需求是需要在默写特定的section 里面引入一小块公共模板
比如有些页面的右侧可以放个热门列表

这时候我们可以在layouts里面价格模板right.blade.php
内容如下
```
<div class="widget widget_ui_viewposts">
    <h3 class="title"><strong>热门阅读</strong></h3>  
    <ul>
        <li>文章1</li>
        <li>文章2</li>
        <li>文章3</li>
    </ul>
</div>
```
一定要注意不要定义section

然后在需要引用的地方用@include

这个方法无论是section里面还是外面都可以直接用的
