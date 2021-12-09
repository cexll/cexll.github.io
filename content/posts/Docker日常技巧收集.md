---
title: Docker日常技巧收集
subtitle: ""
date: 2021-06-24 13:41:50.413
lastmod: 2021-06-24 13:41:50.413
draft: true
author: "cexll"
authorLink: ""
description: ""

tags: [
  docker
]
categories: [
  docker
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




# 查看正在运行容器

```bash
docker ps
```

# 查看所有运行容器 

```bash
docker ps -a
```

# 进入容器

```bash
docker exec -it 容器ID bash
```

# 停止全部运行中的容器

```bash
docker stop $(docker ps -a)
```

# 删除全部容器

```bash
docker rm $(docker ps -aq)
```

# 停止并删除容器

```bash
docker stop $(docker ps -a) & docker rm $(docker ps -aq)
```

# 清理所有处理终止状态的容器

用 `docker container ls -a` 命令可以查看所有已经创建的包括终止状态的容器，如果数量太多要一个个删除可能会很麻烦，用下面的命令可以清理掉所有处于终止状态的容器。
```bash
$ docker container prune
```

# 删除容器

可以使用 `docker container rm` 来删除一个处于终止状态的容器。例如

```bash
$ docker container rm  trusting_newton
trusting_newton
```

# 容器配置DNS
所有 Docker 容器的 DNS 配置通过 /etc/resolv.conf 文件立刻得到更新。

配置全部容器的 DNS ，也可以在 /etc/docker/daemon.json 文件中增加以下内容来设置。

```bash
{
  "dns" : [
    "114.114.114.114",
    "8.8.8.8"
  ]
}
```
