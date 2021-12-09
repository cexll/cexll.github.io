---
title: 使用 Deployer 部署
subtitle: ""
date: 2021-06-24 13:34:50.394
lastmod: 2021-06-24 13:34:50.394
draft: true
author: "cexll"
authorLink: ""
description: ""

tags: []
categories: []

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



# Deployer 介绍

Deployer 是一个使用 PHP 开发的轻量级项目部署工具，Deployer 不需要在目标服务器上安装任何客户端即可使用，它的原理是通过 SSH 登录到目标服务器，然后执行脚本中定义好的 Shell 命令。

同时 Deployer 内置了一批常见项目、框架的部署脚本，例如 Laravel、WordPress 等等，当我们需要部署相关项目时，只需要调整一些参数就可以直接使用。

# 安装 Deployer

首先我们需要在 Homestead 虚拟机中安装 Deployer，直接通过 composer 即可安装：
```
$ composer global require deployer/deployer
```
其中 global 代表这个 composer 包是安装到全局的。
![捕获.PNG][1]
安装完成后我们试试看是否安装成功：

```
$ dep
```

![捕获2.PNG][2]

可以看到有正常的输出，说明安装成功。

默认安装在 本地用户 ~/.config/composer/vender/bin/

# 创建部署脚本

现在我们需要在 Homestead 虚拟机中创建一个目录用于放置部署脚本：

    $ mkdir -p ~/Code/deploy-laravel-shop
    $ cd ~/Code/deploy-laravel-shop
    $ dep init

dep init 命令用来创建一个部署脚本，会询问我们项目类型，我们是 Laravel 项目所以输入 1 然后回车；接下来询问 Repository 也就是我们代码仓库的地址，大家填入自己的 github 仓库地址即可

![捕获3.PNG][3]
接下来会询问是否允许 Deployer 收集匿名的部署数据以改善项目，大家根据自己的情况填入 yes 或者 no：

![捕获4.PNG][4]

可以看到 Deployer 在当前目录下创建了一个名为 deploy.php 的文件。
我们现在来看看这个文件里有哪些东西：
```php
<?php
namespace Deployer;

// 引入官方的 Laravel 部署脚本
require 'recipe/laravel.php';

// 设置一个名为 application 的环境变量，值为 my_project
set('application', 'my_project');

// 设置一个名为 repository，值为初始化脚本时输入的 github 仓库地址
set('repository', 'https://github.com/leo108/laravel-shop-advanced.git');

// 设置环境变量，不需要关心
set('git_tty', true);

// shared_files / shared_dirs 这两个环境变量是数组格式，add 函数可以往数组里添加值
add('shared_files', []);
add('shared_dirs', []);

// Deployer 会把这个变量下的目录加上写权限
add('writable_dirs', []);

// 添加一台服务器，服务器 IP/域名 是 project.com
host('project.com')
    // 设置一个这台服务器独享的环境变量，名为 deploy_path，值为 ~/my_project
    // Deployer 会把花括号包裹起来的字符串替换成对应的环境变量
    ->set('deploy_path', '~/{{application}}');

// 定义一个名为 build 的任务
task('build', function () {
    // 这个任务的内容是执行 cd ~/my_project && build 命令
    run('cd {{release_path}} && build');
});

// 定义一个后置钩子，当 deploy:failed 任务被执行之后，Deployer 会执行 deploy:unlock 任务
after('deploy:failed', 'deploy:unlock');

// 定义一个前置钩子，在执行 deploy:symlink 任务之前先执行 artisan:migrate
before('deploy:symlink', 'artisan:migrate');
```

可以看到在这个部署文件里并没有太多涉及到部署逻辑的地方，这是因为大部分的部署逻辑都在 recipe/laravel.php 这个 Deployer 自带的脚本中写好了，我们来看看这个文件的内容，这个文件在本地的路径是 ~/.composer/vendor/deployer/deployer/recipe/laravel.php，我们也可以在 github 上直接查看 https://github.com/deployphp/deployer/blob/master/recipe/laravel.php 。

这里面大部分内容都比较浅显易懂，这里就不一一解释了。

我们查看一下这个文件的最底部，可以看到定义了一个名为 deploy 任务，这个是 Deployer 中的默认任务，当我们使用 Deployer 部署而又没有指定具体的任务时，Deployer 就会默认执行 deploy 任务。

这个 deploy 任务的内容是一个数组，这代表 deploy 是一个任务组，Deployer 会按照数组中的顺序依次执行任务。


# 开始部署

接下来我们需要根据我们的具体情况来调整一下部署脚本：

```php
<?php
namespace Deployer;

require 'recipe/laravel.php';

set('repository', 'https://github.com/leo108/laravel-shop-advanced.git');
add('shared_files', []);
add('shared_dirs', []);
add('writable_dirs', []);

host('替换成云服务器的公网 IP')
    ->user('root') // 使用 root 账号登录
    ->identityFile('~/.ssh/laravel-shop-aliyun.pem') // 指定登录密钥文件路径
    ->become('www-data') // 以 www-data 身份执行命令
    ->set('deploy_path', '/var/www/laravel-shop-deployer'); // 指定部署目录

after('deploy:failed', 'deploy:unlock');
before('deploy:symlink', 'artisan:migrate');
```
# 自定义配置部署


```php
<?php
namespace Deployer;

require 'recipe/laravel.php';

set('repository', 'https://github.com/cexll/laravel-shop.git');
add('shared_files', []);
add('shared_dirs', []);
add('copy_dirs', ['node_modules', 'vendor']);
set('writable_dirs', []);

// Hosts
host('服务器公网IP')
    ->user('root')
    ->identityFile('~/.ssh/laravel-shop-aliyun.pem') // 指定登录密钥文件路径
    ->become('www')
    ->set('deploy_path', '/www/wwwroot/test.cwj0.top');

// [Optional] if deploy fails automatically unlock.
// Migrate database before symlink new release.

//desc('Upload .env file');
//task('env:upload', function() {
//    // 将本地的 .env 文件上传到代码目录的 .env
//    upload('.env', '{{release_path}}/.env');
//});


desc('Yarn');
task('deploy:yarn', function() {
    run('cd {{release_path}} && SASS_BINARY_SITE=http://npm.taobao.org/mirrors/node-sass yarn && yarn production', ['timeout' => 600]);
});

desc('Restart Horizon');
task('horizon:restart', function() {
    run('cd {{release_path}} && php artisan horizon:terminate');
})->once();

before('deploy:symlink', 'artisan:migrate');
after('deploy:failed', 'deploy:unlock');

//after('deploy:shared', 'env:upload');
after('deploy:vendors', 'deploy:yarn');
before('deploy:vendors', 'deploy:copy_dirs');
after('artisan:config:cache', 'artisan:route:cache');
after('deploy:symlink', 'horizon:restart');
```

# 上传 .env 文件

Deployer 提供一个名为 upload 的函数用于文件上传。首先我们通过 scp 命令将上一节在云服务器上创建的 .env 文件复制到与 deploy.php 文件的同目录下：

```
$ cd ~/Code/deploy-laravel-shop
$ scp root@xxxxx:/var/www/laravel-shop/.env .
```

# 运行

```
$ dep deploy
```


  [1]: https://www.cwj0.top/usr/uploads/2020/06/4062115938.png#vwid=899&vhei=640
  [2]: https://www.cwj0.top/usr/uploads/2020/06/311965037.png#vwid=901&vhei=576
  [3]: https://www.cwj0.top/usr/uploads/2020/06/1107989602.png#vwid=876&vhei=609
  [4]: https://www.cwj0.top/usr/uploads/2020/06/2382728732.png#vwid=876&vhei=620