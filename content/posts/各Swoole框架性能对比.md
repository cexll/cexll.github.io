---
title: 各Swoole框架性能对比
subtitle: ""
date: 2021-06-24 13:36:34.283
lastmod: 2021-06-24 13:36:34.283
draft: true
author: "cexll"
authorLink: ""
description: ""

tags: [
  php,swoole
]
categories: [
  php
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



***时间截止 2020/02/23***

```bash
电脑环境: Windows10 LTSC WSL Ubuntu18.04LTS (X5660 6h12 @2.8 16G DDR3 1333)
运行环境: PHP7.2 Swoole4.4.14

测试页面均为安装后默认
测试不严谨,但有参考价值
```

# Swoft v2.0.8

  -  第一次

```bash
➜  Swoft wrk -c 100 -t 100 -d 60s http://127.0.0.1:18306
Running 1m test @ http://127.0.0.1:18306
  100 threads and 100 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    17.29ms    2.53ms  41.09ms   80.99%
    Req/Sec    57.88      6.04   120.00     64.10%
  347184 requests in 1.00m, 1.87GB read
Requests/sec:   5778.05
Transfer/sec:     31.95MB
```

  -  第二次

```bash
➜  php wrk -c 100 -t 100 -d 60s http://127.0.0.1:18306
Running 1m test @ http://127.0.0.1:18306
  100 threads and 100 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    16.98ms    2.35ms  44.75ms   82.33%
    Req/Sec    58.93      5.83   282.00     69.65%
  353479 requests in 1.00m, 1.91GB read
Requests/sec:   5881.24
Transfer/sec:     32.52MB
```

  -  第三次

```bash
➜  php wrk -c 100 -t 100 -d 60s http://127.0.0.1:18306
Running 1m test @ http://127.0.0.1:18306
  100 threads and 100 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    17.11ms    2.46ms  51.08ms   83.15%
    Req/Sec    58.52      5.90   171.00     66.99%
  350933 requests in 1.00m, 1.89GB read
Requests/sec:   5840.62
Transfer/sec:     32.30MB
```


# EasySwoole v3.3.4

  -  第一次

```bash
➜  php wrk -c 100 -t 100 -d 60s http://127.0.0.1:9501
Running 1m test @ http://127.0.0.1:9501
  100 threads and 100 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    14.80ms    2.31ms  57.43ms   71.99%
    Req/Sec    67.66      7.27   450.00     55.52%
  405597 requests in 1.00m, 637.85MB read
Requests/sec:   6749.03
Transfer/sec:     10.61MB
```

  -  第二次

```bash
➜  php wrk -c 100 -t 100 -d 60s http://127.0.0.1:9501
Running 1m test @ http://127.0.0.1:9501
  100 threads and 100 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    14.29ms    2.07ms  35.04ms   70.22%
    Req/Sec    70.05      6.82   464.00     61.23%
  415687 requests in 1.00m, 653.71MB read
Requests/sec:   6918.06
Transfer/sec:     10.88MB
```

  -  第三次

```bash
➜  php wrk -c 100 -t 100 -d 60s http://127.0.0.1:9501
Running 1m test @ http://127.0.0.1:9501
  100 threads and 100 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    14.53ms    2.13ms  37.30ms   70.33%
    Req/Sec    68.90      7.26   575.00     59.26%
  413033 requests in 1.00m, 649.54MB read
Requests/sec:   6871.81
Transfer/sec:     10.81MB

```


# Hyperf v1.1

  -  第一次

```bash
➜  php wrk -c 100 -t 100 -d 60s http://127.0.0.1:9501
Running 1m test @ http://127.0.0.1:9501
  100 threads and 100 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     4.23ms    1.81ms  84.42ms   79.98%
    Req/Sec   238.89     53.71     0.89k    75.80%
  1430318 requests in 1.00m, 259.17MB read
Requests/sec:  23797.41
Transfer/sec:      4.31MB

```

 - 第二次

```bash
➜  php wrk -c 100 -t 100 -d 60s http://127.0.0.1:9501
Running 1m test @ http://127.0.0.1:9501
  100 threads and 100 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     4.41ms    2.86ms 113.49ms   83.12%
    Req/Sec   235.72    105.72     1.17k    72.61%
  1409861 requests in 1.00m, 255.46MB read
Requests/sec:  23456.44
Transfer/sec:      4.25MB

```

 - 第三次

```bash
➜  php wrk -c 100 -t 100 -d 60s http://127.0.0.1:9501
Running 1m test @ http://127.0.0.1:9501
  100 threads and 100 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     5.00ms   22.00ms 998.36ms   99.75%
    Req/Sec   240.64     77.35     0.89k    75.48%
  1438216 requests in 1.00m, 260.60MB read
Requests/sec:  23928.53
Transfer/sec:      4.34MB
```

# Mix v2.1.8

 - 第一次

```bash
➜  php wrk -c 100 -t 100 -d 60s http://127.0.0.1:9501
Running 1m test @ http://127.0.0.1:9501
  100 threads and 100 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    11.67ms    1.30ms  69.86ms   96.39%
    Req/Sec    85.98      6.78   363.00     92.94%
  515068 requests in 1.00m, 88.91MB read
Requests/sec:   8569.74
Transfer/sec:      1.48MB

```

 - 第二次

```bash
➜  php wrk -c 100 -t 100 -d 60s http://127.0.0.1:9501
Running 1m test @ http://127.0.0.1:9501
  100 threads and 100 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    11.53ms  520.29us  19.26ms   96.10%
    Req/Sec    86.93      5.58   454.00     69.55%
  520715 requests in 1.00m, 89.88MB read
Requests/sec:   8663.78
Transfer/sec:      1.50MB
```

 - 第三次

```bash
➜  php wrk -c 100 -t 100 -d 60s http://127.0.0.1:9501
Running 1m test @ http://127.0.0.1:9501
  100 threads and 100 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    11.80ms    1.01ms  25.55ms   92.47%
    Req/Sec    84.90      7.22     0.91k    94.35%
  508519 requests in 1.00m, 87.78MB read
Requests/sec:   8464.00
Transfer/sec:      1.46MB

```

# iMi v1.0.29

 - 第一次

```bash
➜  php wrk -c 100 -t 100 -d 60s http://127.0.0.1:8080
Running 1m test @ http://127.0.0.1:8080
  100 threads and 100 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    18.34ms   51.83ms   1.96s    99.68%
    Req/Sec    62.61     10.78   393.00     83.96%
  362616 requests in 1.00m, 70.89MB read
  Socket errors: connect 0, read 0, write 0, timeout 78
Requests/sec:   6033.35
Transfer/sec:      1.18MB
```

- 第二次

```bash
➜  php wrk -c 100 -t 100 -d 60s http://127.0.0.1:8080
Running 1m test @ http://127.0.0.1:8080
  100 threads and 100 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    16.05ms    2.91ms  47.86ms   71.69%
    Req/Sec    62.38      7.18   171.00     57.90%
  374154 requests in 1.00m, 73.15MB read
Requests/sec:   6226.31
Transfer/sec:      1.22MB

```

- 第三次

```bash
➜  php wrk -c 100 -t 100 -d 60s http://127.0.0.1:8080
Running 1m test @ http://127.0.0.1:8080
  100 threads and 100 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    16.27ms    2.97ms  41.05ms   71.52%
    Req/Sec    61.52      7.30   212.00     52.50%
  369050 requests in 1.00m, 72.15MB read
Requests/sec:   6140.92
Transfer/sec:      1.20MB

```