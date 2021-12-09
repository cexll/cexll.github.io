---
title: Docker容器内重启其他容器或者重启自己
subtitle: ""
date: 2021-08-27 09:10:51.389
lastmod: 2021-08-27 09:10:51.389
draft: true
author: "cexll"
authorLink: ""
description: ""

tags: [
    docker,docker in docker
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


> 其实道理一看就明白很简单,只需要将命令和docker运行后的文件映射进容器内就可以实现, 具体看命令

# 启动一个Nginx容器,将Docker映射进去
```bash
docker run --name nginx -v /etc/localtime:/etc/localtime:ro -v /var/run/docker.sock:/var/run/docker.sock  -v /usr/bin/docker:/usr/bin/docker -d nginx
```

启动容器后我们进入容器就可以相当于在容器内操作容器
```bash
进入容器
$ docker exec -it nginx /bin/bash
$ root@nginx:/# docker ps
CONTAINER ID   IMAGE                 COMMAND                  CREATED          STATUS          PORTS                                                                                                                             NAMES
f3d73c45399e   nginx                 "/docker-entrypoint.…"   35 seconds ago   Up 32 seconds   80/tcp                                                                                                                            nginx
```

芜湖~