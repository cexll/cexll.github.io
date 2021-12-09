---
title: Laravel 使用时的一些优化技巧
subtitle: ""
date: 2021-06-24 13:35:41.099
lastmod: 2021-06-24 13:35:41.099
draft: true
author: "cexll"
authorLink: ""
description: ""

tags: [
  php,laravel,优化
]
categories: [
  php,laravel
]

hiddenFromHomePage: false
hiddenFromSearch: false

featuredImage: ""
featuredImagePreview: ""

toc:
  enable: true
math:
  enable: false
lightgallery: false
license: ""
---

<!--more-->





## 优化Laravel响应性能
使用Laravel提供的缓存提高响应速度
```bash
APP_ENV = production
APP_DEBUG = false
php artisan optimize // 缓存框架引导文件
php artisan config:cache // 创建缓存文件以加快配置加载
php artisan event:cache // 发现并缓存应用程序的事件和侦听器
php artisan route:cache // 创建路由缓存文件以更快地进行路由注册
php artisan view:cache // 编译应用程序的所有Blade模板
composer install --no-dev // 安装生产环境扩展
composer dump-autoload -o  // 优化 composer 的自动加载
```

## 查询慢sql
在 `app/Providers/AppServiceProvider.php` -> boot()下添加如下代码
在laravel.log 查询响应超过10的sql
```php
DB::listen(function($query){
            $sql = $query->sql;
            $bingings = $query->bindings;
            $time = $query->time;
            if($time>10){
                Log::debug(compact('sql','bingings','time'));
            }
});
```

## 使用Shadowfax提高Laravel性能使起运行在swoole上
[Shadowfax](https://github.com/huang-yi/shadowfax)是一个Laravel拓展包，它可以让你的Laravel应用运行在Swoole之上，以获得巨大的性能提升。
### 安装Swoole
```
pecl install swoole
```
### Composer安装
```
composer require huang-yi/shadowfax
```
### 发布资源
```
php artisan shadowfax:publish
```
### 优化配置
```yml
name: shadowfax
type: http
host: 127.0.0.1
port: 1215
mode: process # process | base 模式
access_log: true
app_pool_capacity: 10

server:
  worker_num: 4 # 进程数量
  enable_coroutine: true # 启用协程
  hook_flags: SWOOLE_HOOK_ALL # 启用一键协程

abstracts:
  - cookie
  - session
  - session.store
  - redirect
  - auth
  - auth.driver
  - Illuminate\Session\Middleware\StartSession

controllers:
  - "*"

cleaners:
  - app/Cleaners/

db_pools:
  mysql: 3

redis_pools:
  default: 3

controller_server:
  host: 127.0.0.1
  port: 1216
```
### 启动
```
php shadowfax start
```
