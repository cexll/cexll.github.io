---
title: 使用代理解决go get龟速,go mod教程
subtitle: ""
date: 2021-06-24 13:29:25.115
lastmod: 2021-06-24 13:29:25.115
draft: true
author: "cexll"
authorLink: ""
description: ""

tags: [
	go,proxy
]
categories: [
	go
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


这里的代理为代理上游服务代理非本地代理

**推荐的代理**
```
Goproxy中国 https://goproxy.cn/
goproxy.io https://goproxy.io/zh/
```

# 使用
> 更多操作可以查看网站文档
```bash
 goproxy 
go env -w GO111MODULE=on
go env -w GOPROXY=https://goproxy.cn,direct
 goproxy.io
go env -w GO111MODULE=on
go env -w GOPROXY=https://goproxy.io,direct
```

这个使用使用`go get -u xxx` 会发现之前下不动的已经飞速了

# go mod
GOMODEL绝对是一个非常棒的,这里使用小案例讲解如何使用`go mod`
```bash
mkdir hello
cd hello
touch main.go
# 初始化go mod
go mod init go.mod
vim main.go

  package main
  import "github.com/gin-gonic/gin"
  func main() {
  	  r := gin.Default()
	  r.GET("/ping", func(c *gin.Context) {
		  c.JSON(200, gin.H{
			  "message": "pong",
		  })
	  })
	  r.Run() // listen and serve on 0.0.0.0:8080
  }

go run main.go
这里就会自动去下载需要的库,并且在`go.mod` require 包的版本路径,生成`go.sum` 生成依赖库的信息

```