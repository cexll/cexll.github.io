---
title: 记录Docker日志将磁盘占满导致服务器崩掉
subtitle: ""
date: 2021-11-03 13:49:44.323
lastmod: 2021-11-03 13:49:44.323
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


# 问题
突然服务器站点访问异常, 数据库无法连接,好在ssh能够连接上去

定睛一看 发现磁盘被吃满了, 不知道为啥..
运行部分命令无响应, 有些提示
```bash
-bash: cannot create temp file for here-document: No space left on device ls -bash
```
? 为什么磁盘满了? 
使用 `df -h` 一看果然都没有了

# 解决 
查看一下大文件

```
du -sh /* | grep G
...
/var/lib/docker/containers
```
? `docker` 难道是docker日志的问题吗 

通过 `docker ps`  好像没问题 `docker stats` 好像掉了一个服务

查看一下日志呢

```bash
du -d1 -h /var/lib/docker/containers | sort -h

32K    /var/lib/docker/containers/d607c06e475191fff1abd0c2b4b672e7fe8a96cb197f4e8557b18600de2e60af
36K    /var/lib/docker/containers/0d4321106721b9d26335fefef7b9e8e23629691684a4da2f953ac8223c8240c3
36K    /var/lib/docker/containers/7525aab4aa917aa1016169114762261726ac7b9cc712bef35cdc7035b50d20ce
36K    /var/lib/docker/containers/9252e1c373d59ef5613c2b6122eb6e43aa2bd822bd2c199aa67d6eb659c4adb7
10G    /var/lib/docker/containers
1525M    /var/lib/docker/containers/15700ee92cd2831554b9a1e78127df0f07248c1498d35c17525407bc8a98bc1a
```

.. 离谱 

清理文件
```
sh -c "cat /dev/null > ${log_file_name}"
```
应急情况推荐`rm -rf /var/lib/docker/containers`

后续百度解决方案

- 单一容器限制日志大小
- 全局限制日志大小(只针对新创建的容器)

## 单一容器配置
启动
```
docker run -it --log-opt max-size=10m --log-opt max-file=3 redis
```
但是启动后的怎么办呢

只能通过脚本去定时清理了..
```shell
#!/bin/sh 
echo "======== start clean docker containers logs ========"  
logs=$(find /var/lib/docker/containers/ -name *-json.log)  
for log in $logs  
        do  
                echo "clean logs : $log"  
                cat /dev/null > $log  
        done  
echo "======== end clean docker containers logs ========"
```
## 全局配置

vim `/etc/docker/daemon.json`
```json
{
    "log-driver":"json-file",
    "log-opts":{
        "max-size" :"50m",
        "max-file":"3"
    }
}
```
重启 Docker 服务
```bash
systemctl daemon-reload
systemctl restart docker
```