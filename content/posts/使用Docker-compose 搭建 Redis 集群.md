---
title: 使用Docker-compose 搭建 Redis 集群
subtitle: ""
date: 2021-10-30 16:23:08.451
lastmod: 2021-10-30 16:23:08.451
draft: true
author: "cexll"
authorLink: ""
description: ""

tags: [
  redis,docker,docker-compose,集群,分布式
]
categories: [
  redis,集群,docker
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


![image.png](https://cdn.jsdelivr.net/gh/cexll/cexll.github.io/images/2021/10/image-705f48bc5984487f8e6ab94a0b022470.png)
# 准备工作
### 编写`docker-compose.yml`

```yaml
version: "3.7"
services:
  rds1:
    image: redis:alpine
    container_name: rds1
    ports:
      - 16371:16371
    volumes:
      - "./conf/rds1/redis.conf:/etc/redis.conf:ro"
    entrypoint: ["redis-server", "/etc/redis.conf"]
    expose:
      - 16371
    networks:
      - default
    
  rds2:
    image: redis:alpine
    container_name: rds2
    ports:
      - 16372:16372
    volumes:
      - "./conf/rds2/redis.conf:/etc/redis.conf:ro"
    entrypoint: ["redis-server", "/etc/redis.conf"]
    expose:
      - 16372
    networks:
      - default
    
  rds3:
    image: redis:alpine
    container_name: rds3
    ports:
      - 16373:16373
    volumes:
      - "./conf/rds3/redis.conf:/etc/redis.conf:ro"
    entrypoint: ["redis-server", "/etc/redis.conf"]
    expose:
      - 16373
    networks:
      - default
    
  rds4:
    image: redis:alpine
    container_name: rds4
    ports:
      - 16374:16374
    volumes:
      - "./conf/rds4/redis.conf:/etc/redis.conf:ro"
    entrypoint: ["redis-server", "/etc/redis.conf"]
    expose:
      - 16374
    networks:
      - default
    
  rds5:
    image: redis:alpine
    container_name: rds5
    ports:
      - 16375:16375
    volumes:
      - "./conf/rds5/redis.conf:/etc/redis.conf:ro"
    entrypoint: ["redis-server", "/etc/redis.conf"]
    expose:
      - 16375
    networks:
      - default
    
  rds6:
    image: redis:alpine
    container_name: rds6
    ports:
      - 16376:16376
    volumes:
      - "./conf/rds6/redis.conf:/etc/redis.conf:ro"
    entrypoint: ["redis-server", "/etc/redis.conf"]
    expose:
      - 16376
    networks:
      - default

networks:
  default:
```

准备目录`conf` 新建多个service的配置文件
![image.png](https://cdn.jsdelivr.net/gh/cexll/cexll.github.io/images/2021/10/image-2df0b784b48e487e9bbe5930e28c0145.png)
配置文件 https://cdn.jsdelivr.net/gh/cexll/cexll.github.io/images/2021/10/redis-bb8a2ebddd77486fa7084248b9993953.conf 自行下载

核心地方在port更改
```conf
port 16371
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
appendonly yes
daemonize no
protected-mode no
pidfile  /data/redis.pid
```

# 开始

```
docker-compose up -d
```

全部启动之后我们找一个,就找`rds1`把 进入到

```
docker exec -it rds1 /bin/bash
我们进去让每个节点建立连接
## 这里的IP对应你容器的IP
redis-cli -p 16371 --cluster create \
192.168.0.6:16371 \ 
192.168.0.4:16372 \
192.168.0.7:16373 \
192.168.0.5:16374 \
192.168.0.3:16375 \
192.168.0.2:16376 \
--cluster-replicas 1

## 我们回车之后, 会告诉你节点分布 redis集群默认是1主1备所以节点都是成双的 不会出现单数
>>> Performing hash slots allocation on 6 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica 192.168.0.3:16375 to 192.168.0.6:16371
Adding replica 192.168.0.2:16376 to 192.168.0.4:16372
Adding replica 192.168.0.5:16374 to 192.168.0.7:16373
M: de50954fbef8a034162d85614f673b23181aee91 192.168.0.6:16371
   slots:[0-5460] (5461 slots) master
M: f6d715eebf2a0605045a08d58551ce14a9a2a214 192.168.0.4:16372
   slots:[5461-10922] (5462 slots) master
M: c9f662d088c8fc34df14a11cd4cacedac68cd0aa 192.168.0.7:16373
   slots:[10923-16383] (5461 slots) master
S: a9361289245ed860b4b34ba183c805b4fdc696d4 192.168.0.5:16374
   replicates c9f662d088c8fc34df14a11cd4cacedac68cd0aa
S: 28d845888ca0ef1a2b50e8c3d164a92df42f26e9 192.168.0.3:16375
   replicates de50954fbef8a034162d85614f673b23181aee91
S: 131e52485a273394f75275d3fb308ebe54282b04 192.168.0.2:16376
   replicates f6d715eebf2a0605045a08d58551ce14a9a2a214
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join
..

成功情况是
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

## 集群信息
```
127.0.0.1:16371> cluster info
cluster_state:ok
cluster_slots_assigned:16384
cluster_slots_ok:16384
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:6
cluster_size:3
cluster_current_epoch:6
cluster_my_epoch:1
cluster_stats_messages_ping_sent:101
cluster_stats_messages_pong_sent:98
cluster_stats_messages_sent:199
cluster_stats_messages_ping_received:93
cluster_stats_messages_pong_received:101
cluster_stats_messages_meet_received:5
cluster_stats_messages_received:199
```

### 集群节点

```
127.0.0.1:16371> cluster nodes
f6d715eebf2a0605045a08d58551ce14a9a2a214 192.168.0.4:16372@26372 master - 0 1635581561000 2 connected 5461-10922
c9f662d088c8fc34df14a11cd4cacedac68cd0aa 192.168.0.7:16373@26373 master - 0 1635581560000 3 connected 10923-16383
28d845888ca0ef1a2b50e8c3d164a92df42f26e9 192.168.0.3:16375@26375 slave de50954fbef8a034162d85614f673b23181aee91 0 1635581559904 1 connected
a9361289245ed860b4b34ba183c805b4fdc696d4 192.168.0.5:16374@26374 slave c9f662d088c8fc34df14a11cd4cacedac68cd0aa 0 1635581561912 3 connected
131e52485a273394f75275d3fb308ebe54282b04 192.168.0.2:16376@26376 slave f6d715eebf2a0605045a08d58551ce14a9a2a214 0 1635581562914 2 connected
de50954fbef8a034162d85614f673b23181aee91 192.168.0.6:16371@26371 myself,master - 0 1635581558000 1 connected 0-5460
```

# 连接

```
 redis-cli -p 16371
127.0.0.1:16371> set hello world
OK
127.0.0.1:16371> get hello
"world"
127.0.0.1:16371>
```
图形化连接效果
![image.png](https://cdn.jsdelivr.net/gh/cexll/cexll.github.io/images/2021/10/image-941a3039882945f99399a5695529e391.png)
