---
layout: post
title: 利用Laravel 搭建oauth2 API接口
date: 2017-09-22 14:32:55.000000000 +08:00
---

## 要求 

laravel 5.4以上

## 安装

`$ composer require laravel/passport`

在配置文件 config/app.php 的providers 数组中注册 Passport 服务提供者：

Laravel\Passport\PassportServiceProvider::class,

迁移数据库 执行完后会生成oauth需要的表

`$ php artisan migrate`

这一步注意，执行的时候可能会报错 

> Syntax error or access violation: 1071 Specified key was too long; max key length is 767 byte

这是由于 Laravel5.4默认使用utf8mb4 编码 
utf8 最大长度字符是3字节 utf8mb4是4字节

解决方法就是 
1. 数据库改用utf8mb4 
2. AppServiceProvider.php里面加上Schema::defaultStringLength(191);

```
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use Illuminate\Support\Facades\Schema;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        //

        Schema::defaultStringLength(191);
    }

    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        //
    }
}

```

另外Mysql 5.5.3之后才支持utf8mb4也需要注意下

接下来执行

`$ php artisan passport:install`

会生成两个客户端密钥

```
Client ID: 1
Client Secret: AwDMcCs65rXkzF80wPaINx5fkoXEfa8lcuuPEvQK
Password grant client created successfully.
Client ID: 2
Client Secret: KLlLijWk3hX2Ntfzo2iFPgwT4GyITpBjEuDozp5H
```

## 配置

这里可以配置的只有access token的生命周期默认是永久的
在AuthServiceProvider中配置

```
use Carbon\Carbon;
use Laravel\Passport\Passport;

/**
 * 注册所有认证/授权服务.
 *
 * @return void
 */
public function boot()
{
    $this->registerPolicies();

    Passport::routes();
    Passport::tokensExpireIn(Carbon::now()->addDays(15));
    Passport::refreshTokensExpireIn(Carbon::now()->addDays(30));
}
```

修改auth.php 
['guards']['api']['driver'] => 'passport'  

## 发放access_token

颁发

应用场景 我的用户，在别的网站想用我的账号直接登录，参考微信登录。那么第三方网站就要对接过来，用户选择第三方登录，跳转到我的页面，询问用户是否允许，用户允许以后我会带一个code回去，第三方网站用这个code请求access_token

流程是

请求令牌
``` GET方式 请求到 http://your-app.com/oauth/authorize 带如下参数
    
    'client_id' => 'client-id',
    'redirect_uri' => 'http://example.com/callback',
    'response_type' => 'code',
    'scope' => '',
```

用户允许以后拿到code换token
```
 $response = $http->post('http://your-app.com/oauth/token', [
        'form_params' => [
            'grant_type' => 'authorization_code',
            'client_id' => 'client-id',
            'client_secret' => 'client-secret',
            'redirect_uri' => 'http://example.com/callback',
            'code' => $request->code,
        ],
    ]);
```

token如果过期了，可以刷新
```
$response = $http->post('http://your-app.com/oauth/token', [
    'form_params' => [
        'grant_type' => 'refresh_token',
        'refresh_token' => 'the-refresh-token',
        'client_id' => 'client-id',
        'client_secret' => 'client-secret',
        'scope' => '',
    ],
]);
```


账号密码

这个主要是用于APP（我自己的），用户通过app输入账号和密码，我用账号密码校验正确了就发送access_token 
```
$response = $http->post('http://your-app.com/oauth/token', [
    'form_params' => [
        'grant_type' => 'password',
        'client_id' => 'client-id',
        'client_secret' => 'client-secret',
        'username' => 'taylor@laravel.com',
        'password' => 'my-password',
        'scope' => '',
    ],
]);
```

隐式

这种跟第一种差不多，就是省去了code 直接发放，主要用于 
> JavaScript 或移动应用中客户端登录认证信息不能保存时


客户端证书

这种主要用于机器之间的通信
直接用appid 和 appsecret 换令牌 

```
$response = $guzzle->post('http://your-app.com/oauth/token', [
    'form_params' => [
        'grant_type' => 'client_credentials',
        'client_id' => 'client-id',
        'client_secret' => 'client-secret',
        'scope' => 'your-scope',
    ],
]);
```

私人访问令牌

这个用于在程序里面调用API的时候
比如
```
$user = App\User::find(1);

// 创建一个不带域的令牌...
$token = $user->createToken('Token Name')->accessToken;

// 创建一个带域的令牌...
$token = $user->createToken('My Token', ['place-orders'])->accessToken;
```


在调用api之前需要创建client
创建命令是
`$ php artisan passport:client`

密码和私人的不同其他都一样

`$ php artisan passport:client --password`
`$ php artisan passport:client --personal`

创建好后获得client-id和client-secret

## 创建路由

5.4以后目录结构发生变化，路由统一写在routes文件夹下。
API的路由都写在api.php

确定好路由就可以请求接口了

```
GET 方式
    /api/user
    'headers' => [
        'Accept' => 'application/json',
        'Authorization' => 'Bearer '.$accessToken,
    ],
```


写到这里遇到一个问题

就是无论怎样请求 获取到的token 用来访问接口的时候 总是返回
Unauthenticated

GOOGLE了下发现好多人也遇到这个问题，据说是token过期时间的问题

在AuthServiceProvider boot里面加上
``` 
Passport::tokensExpireIn(Carbon::now()->addDays(15));
Passport::refreshTokensExpireIn(Carbon::now()->addDays(30));
```
这样应该会解决，然而并没有，这里等以后一时间再研究下(已解决 见下文)


这个问题有了一定进展

目前通过用户授权颁发令牌的方式通过了

前提是用户必须登录，之前返回Unauthenticated 应该是因为用户未登录

在应用站跳转到授权站的时候，此时用户需登录状态，授权以后拿到code再来换access_token 这个方式OK的，可以正常获取登录用户的信息

账号密码获取令牌的方式也一样可以通过


站点之前通过 id 和 secret的方式换token，然后拿token请求接口这种方式目前还不行


坑爹啊，官方文档没写全

通过 client_credentials 方式获取token，请求接口的时候，路由不能用auth或者scope等中间件去验证，因为他们会首先验证有没有登录。

我们需要在app\Http\Kernel.php 的 $routeMiddleware 里面定义一个客户端API的中间件

`'client_credentials' => \Laravel\Passport\Http\Middleware\CheckClientCredentials::class,`

然后在路由里面
`Route::middleware('client_credentials')`

或者

`Route::middleware('client_credentials:作用于名称')`

这样就可以实现不登录直接调用api了



参考文档

> https://laravel.com/docs/5.4/passport


