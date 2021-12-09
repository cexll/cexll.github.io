---
title: Docker-compose 部署微服务 Hyperf
subtitle: ""
date: 2021-06-23 09:04:30.92
lastmod: 2021-06-23 09:04:30.92
draft: true
author: "cexll"
authorLink: ""
description: ""

tags: [
    docker,docker-compose,hyperf,微服务
]
categories: [
    docker,hyperf
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


> Hyperf 最近更新了2.2, 对php8做了兼容并且实测性能提升1倍

# 编写 docker-compose-yml
> 对于不熟悉yaml的自己先查阅一下文档
```yaml
version: '3.7'
services:  // 服务, 我这里启动了4个服务 分别用Tab对其
    hyclient: // hyperf-client 即服务消费者
        build: hyclient
        ports: 
            - 9501:9501 // 映射9501端口
        volumes:
            - ./hyclient:/opt/www // 目录映射,让容器内和本地同步
        restart: always // 自动重启规则
        networks:
            - backnet
    hyserver: // hyperf-server 即服务提供者
        build: hyserver
        volumes:
            - ./hyserver:/opt/www
        ports: 
            - 9502:9502
        restart: always
        networks:
            - backnet
    consul: // 这里使用了consul来做服务发现
        image: consul
        ports: 
            - 8500:8500
        restart: always
        networks: 
            - backnet
    redis:  // 使用了redis
        image: redis
        ports: 
            - 6379:6379
        restart: always
        networks: 
            - backnet
networks:
    backnet:
```

# Run
> 运行需要在yml目录
```
docker-compose up -d
```

启动之后就可以在docker看见容器启动了, 也可以使用`docker-compose ps` 查看

# Test
我这里只需要请求`9501`端口就可以查看服务是否正常

> 因为我将端口映射出来了,所以可以直接使用`localhost`访问
```linux
curl http://localhost:9501/
```

~~代码后续开源到`Github`, 持续关注~~

https://github.com/cexll/docker-compose-hyperf

项目已开源