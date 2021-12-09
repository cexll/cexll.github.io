---
title: Composer 国内加速镜像
subtitle: ""
date: 2021-06-24 13:35:15.013
lastmod: 2021-06-24 13:35:15.013
draft: true
author: "cexll"
authorLink: ""
description: ""

tags: [
    php,composer,proxy
]
categories: [
    php,composer
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


# 选项一、全局配置（推荐）
```bash
$ composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/
```

# 选项二、单独使用

如果仅限当前项目使用镜像，去掉 -g 即可，如下：

```bash
$ composer config repo.packagist composer https://mirrors.aliyun.com/composer/
```

# 取消镜像

```bash
$ composer config -g --unset repos.packagist
```

# 打印调试信息

`composer` 命令后面加上 -vvv （是 3 个 v）可以打印出调错信息，命令如下：

```bash
$ composer -vvv create-project laravel/laravel blog
$ composer -vvv require psr/log
```

## 增加 Composer Cache大小

缓存大小看本地内存大小
```bash
composer config --global cache-files-maxsize 1024MiB   
```

## 增加`php.ini`memory_limit内存
```bash
memory_limit = -1
php -r "echo ini_get('memory_limit').PHP_EOL;"
```