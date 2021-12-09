---
title: 使用Golang交叉编译
subtitle: ""
date: 2021-06-24 13:37:32.544
lastmod: 2021-06-24 13:37:32.544
draft: true
author: "cexll"
authorLink: ""
description: ""

tags: [
    go
]
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



# golang编译命令

```bash
GOOS=linux GOARCH=amd64 go build -ldflags "-w -s" -o name
```
- GOOS: 要运行的操作系统
- GOARCH: 要运行的系统位数(amd64/386)
- name: 编译后软件名称
