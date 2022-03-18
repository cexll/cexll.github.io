---
title: 使用docker-swarm集群管理
subtitle: ""
date: 2021-09-22 21:43:58.421
lastmod: 2021-09-22 21:43:58.421
draft: true
author: "cexll"
authorLink: ""
description: ""

tags: [
  docker,swarm,集群
]
categories: [
  docker,swarm
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


> 对于热门的k8s，docker也有相应的集群管理工具，虽然现在是k8s的时代，但是对于中小型公司，docker其实已经够用

# Docker-Machine
Docker Machine 是一种可以让您在虚拟主机上安装 Docker 的工具，并可以使用 docker-machine 命令来管理主机。
![68747470733a2f2f646f63732e646f636b65722e636f6d2f6d616368696e652f696d672f6c6f676f2e706e67](https://cdn.jsdelivr.net/gh/cexll/cexll.github.io/images/2021/09/68747470733a2f2f646f63732e646f636b65722e636f6d2f6d616368696e652f696d672f6c6f676f2e706e67-183afe9a648847688090f455d4899dda-thumbnail.png)
Docker Machine 也可以集中管理所有的 docker 主机，比如快速的给 100 台服务器安装上 docker。

Docker Machine 管理的虚拟主机可以是机上的，也可以是云供应商，如阿里云，腾讯云，AWS，或 DigitalOcean。

使用 docker-machine 命令，您可以启动，检查，停止和重新启动托管主机，也可以升级 Docker 客户端和守护程序，以及配置 Docker 客户端与您的主机进行通信。
![machine](https://cdn.jsdelivr.net/gh/cexll/cexll.github.io/images/2021/09/machine-5f64b86f780b4854801f825f5cc31780.png)
# Install

- Linux
```
$ base=https://github.com/docker/machine/releases/download/v0.16.0 &&
  curl -L $base/docker-machine-$(uname -s)-$(uname -m) >/tmp/docker-machine &&
  sudo mv /tmp/docker-machine /usr/local/bin/docker-machine &&
  chmod +x /usr/local/bin/docker-machine
```
- Windows
注意 Windows 需要使用`GitBash`
```
$ base=https://github.com/docker/machine/releases/download/v0.16.0 &&
  mkdir -p "$HOME/bin" &&
  curl -L $base/docker-machine-Windows-x86_64.exe > "$HOME/bin/docker-machine.exe" &&
  chmod +x "$HOME/bin/docker-machine.exe"
```
- MacOS
```
$ base=https://github.com/docker/machine/releases/download/v0.16.0 &&
  curl -L $base/docker-machine-$(uname -s)-$(uname -m) >/usr/local/bin/docker-machine &&
  chmod +x /usr/local/bin/docker-machine
```

查看是否安装成功：

```
$ docker-machine version
docker-machine version 0.16.0, build 9371605
```


> 注意 本地需要提前安装 virtuabox

# 使用

## 1. 列出可用机器

```
docker-machine ls
```
![docker-machine1](https://cdn.jsdelivr.net/gh/cexll/cexll.github.io/images/2021/09/docker-machine1-4c0ffc1f196f43529f24112fc6652eee.png)

## 2. 创建机器

```
$ docker-machine create --driver virtualbox test
```
> --driver：指定用来创建机器的驱动类型，这里是 virtualbox。
![docker-machine2](https://cdn.jsdelivr.net/gh/cexll/cexll.github.io/images/2021/09/docker-machine2-99c1e47e3e11445bb518c1dcc75a6340.png)

## 3. 查看机器的 ip

```
$ docker-machine ip test
```
![docker-machine3](https://cdn.jsdelivr.net/gh/cexll/cexll.github.io/images/2021/09/docker-machine3-601a4ef336174b4c88b174eaa982369c.png)

## 4. 停止机器

```
$ docker-machine stop test
```
![docker-machine4](https://cdn.jsdelivr.net/gh/cexll/cexll.github.io/images/2021/09/docker-machine4-f26e1e153cfc49d881ffd822b2b91eac.png)

## 5. 启动机器
```
$ docker-machine start test
```
![docker-machine5](https://cdn.jsdelivr.net/gh/cexll/cexll.github.io/images/2021/09/docker-machine5-f55e3b9bea614e9ca3c0775116dc65b6.png)

## 6. 进入机器

```
$ docker-machine ssh test
```
![docker-machine6](https://cdn.jsdelivr.net/gh/cexll/cexll.github.io/images/2021/09/docker-machine6-7e653ae483764c6dbcb36c8546a9bfab.png)

# Swarm 集群
Docker Swarm 是 Docker 的集群管理工具。它将 Docker 主机池转变为单个虚拟 Docker 主机。 Docker Swarm 提供了标准的 Docker API，所有任何已经与 Docker 守护程序通信的工具都可以使用 Swarm 轻松地扩展到多个主机。

## 原理
如下图所示，swarm 集群由管理节点（manager）和工作节点（work node）构成。
- swarm mananger：负责整个集群的管理工作包括集群配置、服务管理等所有跟集群有关的工作。
- work node：即图中的 available node，主要负责运行相应的服务来执行任务（task）。

![services-diagram](https://cdn.jsdelivr.net/gh/cexll/cexll.github.io/images/2021/09/services-diagram-bddba1e829434efcb3864d0ef714da09.png)

## 1. 创建 swarm 集群管理节点（manager）

```
$ docker-machine create -d virtualbox swarm-manager
```
![swarm1](https://cdn.jsdelivr.net/gh/cexll/cexll.github.io/images/2021/09/swarm1-d5fb270cf2b8403f9cf1e7548e1babc0.png)

初始化 swarm 集群，进行初始化的这台机器，就是集群的管理节点。

```
$ docker-machine ssh swarm-manager
$ docker swarm init --advertise-addr 192.168.99.107 #这里的 IP 为创建机器时分配的 ip。
```
![swarm2](https://cdn.jsdelivr.net/gh/cexll/cexll.github.io/images/2021/09/swarm2-e29f3e1f9dec4ca6a1e6e7a2b446f6d7.png)
以上输出，证明已经初始化成功。需要把以下这行复制出来，在增加工作节点时会用到：

```
docker swarm join --token SWMTKN-1-4oogo9qziq768dma0uh3j0z0m5twlm10iynvz7ixza96k6jh9p-ajkb6w7qd06y1e33yrgko64sk 192.168.99.107:2377
```

## 2. 创建 swarm 集群工作节点（worker）

这里直接创建好俩台机器，swarm-worker1 和 swarm-worker2 
```
$ docker-machine create -d virtualbox swarm-worker1
$ docker-machine create -d virtualbox swarm-worker2
```
![swarm3](https://cdn.jsdelivr.net/gh/cexll/cexll.github.io/images/2021/09/swarm3-e4ae4503562b44218fe4e25e002f4d28.png)

分别进入两个机器里，指定添加至上一步中创建的集群，这里会用到上一步复制的内容。

![swarm4](https://cdn.jsdelivr.net/gh/cexll/cexll.github.io/images/2021/09/swarm4-08ead86529254488b5df9762d79dbedb.png)

以上数据输出说明已经添加成功。

上图中，由于上一步复制的内容比较长，会被自动截断，实际上在图运行的命令如下：

```
docker@swarm-worker1:~$ docker swarm join --token SWMTKN-1-4oogo9qziq768dma0uh3j0z0m5twlm10iynvz7ixza96k6jh9p-ajkb6w7qd06y1e33yrgko64sk 192.168.99.107:2377
```

## 3. 查看集群信息

进入管理节点，执行：docker info 可以查看当前集群的信息。

```
$ docker info
```
![swarm5](https://cdn.jsdelivr.net/gh/cexll/cexll.github.io/images/2021/09/swarm5-a6a0c5bc1286482e90b115cb18708130.png)

通过画红圈的地方，可以知道当前运行的集群中，有三个节点，其中有一个是管理节点。

## 4. 部署服务到集群中

注意：跟集群管理有关的任何操作，都是在管理节点上操作的。

以下例子，在一个工作节点上创建一个名为 helloworld 的服务，这里是随机指派给一个工作节点：

```
docker@swarm-manager:~$ docker service create --replicas 1 --name helloworld alpine ping docker.com
```

![swarm6](https://cdn.jsdelivr.net/gh/cexll/cexll.github.io/images/2021/09/swarm6-a831a842068f4a38ac4d67ebb963c19b.png)


## 5. 查看服务部署情况
查看 helloworld 服务运行在哪个节点上，可以看到目前是在 swarm-worker1 节点：

```
docker@swarm-manager:~$ docker service ps helloworld
```

![swarm7](https://cdn.jsdelivr.net/gh/cexll/cexll.github.io/images/2021/09/swarm7-1d056e4da6e3461b86359f2aef050152.png)

查看 helloworld 部署的具体信息：

```
docker@swarm-manager:~$ docker service inspect --pretty helloworld
```

![swarm8](https://cdn.jsdelivr.net/gh/cexll/cexll.github.io/images/2021/09/swarm8-a45e6d68bff94770aa3bb94a8c24334e.png)

## 6. 扩展集群服务

我们将上述的 helloworld 服务扩展到俩个节点。

```
docker@swarm-manager:~$ docker service scale helloworld=2
```

![swarm9](https://cdn.jsdelivr.net/gh/cexll/cexll.github.io/images/2021/09/swarm9-12b8e13314814fac877c6d6924b70fb6.png)

可以看到已经从一个节点，扩展到两个节点。

![swarm10](https://cdn.jsdelivr.net/gh/cexll/cexll.github.io/images/2021/09/swarm10-d06d247bfd044c5d89f1c451edbac3b0.png)


## 7. 删除服务

```
docker@swarm-manager:~$ docker service rm helloworld
```
![swarm11](https://cdn.jsdelivr.net/gh/cexll/cexll.github.io/images/2021/09/swarm11-cf84fd6cfefc46bfae25194df3c77a43.png)

查看是否已删除：

![swarm12](https://cdn.jsdelivr.net/gh/cexll/cexll.github.io/images/2021/09/swarm12-1158c287564943ceb8dc96eb2d56e4b8.png)

## 8. 滚动升级服务

以下实例，我们将介绍 redis 版本如何滚动升级至更高版本。

创建一个 3.0.6 版本的 redis。

```
docker@swarm-manager:~$ docker service create --replicas 1 --name redis --update-delay 10s redis:3.0.6
```
![swarm13](https://cdn.jsdelivr.net/gh/cexll/cexll.github.io/images/2021/09/swarm13-c07bf48d1e6c48c890a011922f85a43c.png)

滚动升级 redis 。

![swarm14](https://cdn.jsdelivr.net/gh/cexll/cexll.github.io/images/2021/09/swarm14-cd9108cdbecd44a9846a7daef102917d.png)

看图可以知道 redis 的版本已经从 3.0.6 升级到了 3.0.7，说明服务已经升级成功。

## 9. 停止某个节点接收新的任务

查看所有的节点：

```
docker@swarm-manager:~$ docker node ls
```
![swarm16](https://cdn.jsdelivr.net/gh/cexll/cexll.github.io/images/2021/09/swarm16-a77384df67054bcfbf030d1cc9e290ec.png)

可以看到目前所有的节点都是 Active, 可以接收新的任务分配。

停止节点 swarm-worker1：

![swarm17](https://cdn.jsdelivr.net/gh/cexll/cexll.github.io/images/2021/09/swarm17-09c34283ef7a44bf9a9bdabdde6c984f.png)
注意：swarm-worker1 状态变为 Drain。不会影响到集群的服务，只是 swarm-worker1 节点不再接收新的任务，集群的负载能力有所下降。

可以通过以下命令重新激活节点：
```
docker@swarm-manager:~$  docker node update --availability active swarm-worker1
```
![swarm19](https://cdn.jsdelivr.net/gh/cexll/cexll.github.io/images/2021/09/swarm19-6bf905c072f54808afcf58ba70697b02.png)