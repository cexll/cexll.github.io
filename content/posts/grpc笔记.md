---
title: "GRPC笔记"
subtitle: "GRPC笔记"
date: 2022-08-02T11:15:53+08:00
lastmod: 2022-08-02T11:15:53+08:00
draft: true
author: "cexll"
authorLink: "github.com/cexll"
description: "grpc一些开发中可能会用到的技巧"

tags: [grpc, protobuf]
categories: []

hiddenFromHomePage: false
hiddenFromSearch: false

featuredImage: ""
featuredImagePreview: ""

toc:
  enable: true
  auto: true
code:
  copy: true
  maxShownLines: 50
math:
  enable: false
share:
  enable: true
lightgallery: false
license: "MIT"
seo: 
  images: []
---

<!--more-->

# GRPC笔记


### 1. grpc验证
- 实现credentials.PerRPCCredentials接口就可以把数据当做 gRPC 中的 Credential 在添加到 metadata
- 服务端从 ctx 中解析 metadata，然后从 metadata 中获取 授权信息并进行验证
- 可以借助 Interceptor 实现全局身份验证
- 客户端可以通过 DialOption 为所有请求统一指定授权信息，或者通过 CallOption 为每一个请求分别指定授权信息

### 2. 拦截器Interceptor
- 拦截器分类与定义
 - 一元拦截器
    - 预处理
    - 调用RPC方法
    - 后处理
 - 流拦截器
    - 预处理
    - 调用RPC方法 获取 Streamer
    - 后处理
        - 调用 SendMsg 、RecvMsg 之前
        - 调用 SendMsg 、RecvMsg
        - 调用 SendMsg 、RecvMsg 之后
 - 服务端拦截器和客户端拦截器
 - 拦截器使用及执行顺序
    - 配置多个拦截器时，会按照参数传入顺序依次执行
```
grpc.UnaryServerInterceptor
grpc.StreamServerInterceptor
grpc.UnaryClientInterceptor
grpc.StreamClientInterceptor
```

### 3. 利用gateway同时提供HTTP和RPC
1. 安装protoc-gen-grpc-gateway插件
2. .proto文件添加注解
```proto
syntax = "proto3";
option go_package = "github.com/lixd/grpc-go-example/features/proto/echo";
package helloworld;
// 引入 annotations https://github.com/grpc-ecosystem/grpc-gateway/tree/master/third_party/googleapis/google/api
import "google/api/annotations.proto";

service Greeter {
  rpc SayHello (HelloRequest) returns (HelloReply) {
    // 添加注解
    option (google.api.http) = {
      // method: route 
      get: "/v1/greeter/sayhello"
      body: "*"
    };
  }
}

message HelloRequest {
  string name = 1;
}

message HelloReply {
  string message = 1;
}
```
3. 启动grpc同时启动http
```go
// 启动 HTTP 服务
conn, err := grpc.Dial(
    "localhost:50051",
    grpc.WithInsecure(),
)
if err != nil {
    log.Fatalln("Failed to dial server:", err)
}

gwmux := runtime.NewServeMux()
// Register Greeter
err = pb.RegisterGreeterHandler(context.Background(), gwmux, conn)
if err != nil {
    log.Fatalln("Failed to register gateway:", err)
}

gwServer := &http.Server{
    Addr:    fmt.Sprintf(":%d", *restful),
    Handler: gwmux,
}

log.Println("Serving gRPC-Gateway on http://0.0.0.0"+fmt.Sprintf(":%d", *restful))
log.Fatalln(gwServer.ListenAndServe())
```

### 4. 使用context进行超时控制

控制超时时间在高并发下尽可能的控制资源

- context.WithDeadline()
- context.WithTimeout()

### 5. 配置retry自动重试

- 客户端建立连接时通过grpc.WithDefaultServiceConfig(retryPolicy)指定重试策略
- 环境变量中开启重试:export GRPC_GO_RETRY=on

### 6. 客户端负载均衡
- Name Resolver(默认)
- Load Balancing Policy
  - pick_first：尝试连接到第一个地址，如果连接成功，则将其用于所有RPC，如果连接失败，则尝试下一个地址（并继续这样做，直到一个连接成功）
  - round_robin：连接到它看到的所有地址，并依次向每个后端发送一个RPC。例如，第一个RPC将发送到backend-1，第二个RPC将发送到backend-2，第三个RPC将再次发送到backend-1

### 7. k8s grpc负载均衡
```go
// Import the module
import "github.com/sercand/kuberesolver/v3"
	
// Register kuberesolver to grpc before calling grpc.Dial
kuberesolver.RegisterInCluster()
// if schema is 'kubernetes' then grpc will use kuberesolver to resolve addresses
cc, err := grpc.Dial("kubernetes:///service.namespace:portname", opts...)
```

- kubernetes:///service.namespace:portname

#### 7.1 服务端负载均衡

服务端负载均衡主要是在 Pod 之前增加一个 中间组件，一般为 7 层负载均衡。

client 请求中间组件，由中间组件再去请求后端的 Pod。

常见的组件比如 Linkerd，或者 ServiceMesh 如 istio 中的 envoy 也能实现同样的效果。


### 8.gRPC压测工具ghz
https://github.com/bojand/ghz/releases

#### 基本参数
```
--config：指定配置文件位置
--proto：指定 proto 文件位置
        会从 proto 文件中获取相关信息
--call：指定调用的方法。
        具体格式为包名.服务名.方法名
        如：--call helloworld.Greeter.SayHello
-c：并发请求数
-n：最大请求数，达到后则结束测试
-d：请求参数
        JSON格式，如-d '{"name":"Bob"}'
-D：以文件方式指定请求参数，JSON文件位置
        如-D ./file.json
-o：输出路径
        默认输出到 stdout
-O/--format：输出格式，有多种格式可选
        便于查看的：csv、json、pretty、html：
        便于入库的：influx-summary、influx-details：满足InfluxDB line-protocol 格式的输出
```
#### 负载参数

负载参数主要控制ghz每秒发起的请求数（RPS）。
```
-r/--rps：指定RPS
        ghz以恒定的RPS进行测试
--load-schedule：负载调度算法，取值如下：
        const：恒定RPS，也是默认调用算法
        step：步进增长RPS，需要配合load-start，load-step，load-end，load-step-duration，和load-max-duration等参数
        line：线性增长RPS，需要配合load-start，load-step，load-end，和load-max-duration等参数，其实line就是 step 算法将load-step-duration时间固定为一秒了。
--load-start：step、line 的起始RPS
--load-step：step、line 的步进值或斜率值
--load-end：step、line 的负载结束值
--load-max-duration：最大持续时间，到达则结束



示例:
从50RPS开始，每5秒钟增加10RPS，一直到完成10000请求为止。
-n 10000 -c 10 --load-schedule=step --load-start=50 --load-step=10 --load-step-duration=5s

从50RPS开始，每5秒钟增加10RPS，最多增加到150RPS，一直到完成10000请求为止。
-n 10000 -c 10 --load-schedule=step --load-start=50 --load-end=150 --load-step=10 --load-step-duration=5s

从200RPS开始，每1秒钟降低2RPS，一直降低到50RPS，一直到完成10000请求为止。
-n 10000 -c 10 --load-schedule=line --load-start=200 --load-step=-2 --load-end=50
```

#### 并发参数

```
-c：并发woker数，

        注意：不是并发请求数
--concurrency-schedule：并发调度算法，和--load-schedule类似

        const：恒定并发数，默认值
        step：步进增加并发数
        line：线性增加并发数
--concurrency-start：起始并发数

--concurrency-end：结束并发数

--concurrency-step：并发数步进值

--concurrency-step-duration：在每个梯段需要持续的时间

--concurrency-max-duration：最大持续时间

示例:

固定RPS200，worker数从5开始，每5秒增加5，最大增加到50。

-n 100000 --rps 200 --concurrency-schedule=step --concurrency-start=5 --concurrency-step=5 --concurrency-end=50 --concurrency-step-duration=5s
```

#### 配置文件   

所有参数都可以通过配置文件来指定，这也是比较推荐的用法。

```json
{
    "proto": "/path/to/greeter.proto",
    "call": "helloworld.Greeter.SayHello",
    "total": 2000,
    "concurrency": 50,
    "data": {
        "name": "Joe"
    },
    "metadata": {
        "foo": "bar",
        "trace_id": "{{.RequestNumber}}",
        "timestamp": "{{.TimestampUnix}}"
    },
    "import-paths": [
        "/path/to/protos"
    ],
    "max-duration": "10s",
    "host": "0.0.0.0:50051"
}
```


#### 使用方法

```
ghz -c 10 -n 1000 \
   --insecure \
   --proto ./hello_world.proto \
   --call helloworld.Greeter.SayHello \
   -d '{"name":"Joe"}' \
   0.0.0.0:50051
```
`--call helloworld.Greeter.SayHello`: 说明，具体 proto 文件如下

```proto
// 省略其他代码...
package helloworld;
service Greeter {
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}
```

结果

```bash
Summary:
  Count:        1000
  Total:        87.65 ms
  Slowest:      6.97 ms
  Fastest:      0.12 ms
  Average:      0.75 ms
  Requests/sec: 11409.21

Response time histogram:
  0.118 [1]     |
  0.803 [801]   |∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎
  1.487 [131]   |∎∎∎∎∎∎∎
  2.172 [27]    |∎
  2.857 [18]    |∎
  3.542 [12]    |∎
  4.226 [0]     |
  4.911 [0]     |
  5.596 [0]     |
  6.281 [0]     |
  6.966 [10]    |

Latency distribution:
  10 % in 0.35 ms 
  25 % in 0.43 ms 
  50 % in 0.57 ms 
  75 % in 0.75 ms 
  90 % in 1.23 ms 
  95 % in 1.62 ms 
  99 % in 3.31 ms 

Status code distribution:
  [OK]   1000 responses  
```

#### 负载参数)使用方法

```
ghz -c 10 -n 1000 \
   --insecure \
   --proto ./hello_world.proto \
   --call helloworld.Greeter.SayHello \
   -d '{"name":"Joe"}' \
   --load-schedule=step --load-start=50 --load-step=10 --load-step-duration=5s \
   -o report.html -O html \
   0.0.0.0:50051
```
这次指定使用HTML方式输出结果，执行完成后可以在当前目录看到输出的HTML文件
```bash
$ ls
report.html
```
#### 并发参数)使用方法

```
ghz -c 10 -n 10000 \
   --insecure \
   --proto ./hello_world.proto \
   --call helloworld.Greeter.SayHello \
   -d '{"name":"Joe"}' \
   --rps 200 --concurrency-schedule=step --concurrency-start=5 --concurrency-step=5 --concurrency-end=50 --concurrency-step-duration=5s \
   -o report.json -O pretty \
   0.0.0.0:50051
```

本次以CSV格式打印输出

```
duration (ms),status,error
1.05,OK,
0.32,OK,
0.30,OK,
0.36,OK,
0.34,OK,
0.29,OK,
0.40,OK,
0.40,OK,
0.62,OK,
0.31,OK,
0.30,OK,
0.48,OK,
```

#### 编程方式

```go
package main

import (
	"log"
	"os"

	"github.com/bojand/ghz/printer"
	"github.com/bojand/ghz/runner"
	"github.com/golang/protobuf/proto"
	pb "github.com/lixd/grpc-go-example/helloworld/helloworld"
)

// 官方文档 https://ghz.sh/docs/intro.html
func main() {
	// 组装BinaryData
	item := pb.HelloRequest{Name: "lixd"}
	buf := proto.Buffer{}
	err := buf.EncodeMessage(&item)
	if err != nil {
		log.Fatal(err)
		return
	}
	report, err := runner.Run(
		// 基本配置 call host proto文件 data
		"helloworld.Greeter.SayHello", //  'package.Service/method' or 'package.Service.Method'
		"localhost:50051",
		runner.WithProtoFile("../helloworld/helloworld/hello_world.proto", []string{}),
		runner.WithBinaryData(buf.Bytes()),
		runner.WithInsecure(true),
		runner.WithTotalRequests(10000),
		// 并发参数
		runner.WithConcurrencySchedule(runner.ScheduleLine),
		runner.WithConcurrencyStep(10),
		runner.WithConcurrencyStart(5),
		runner.WithConcurrencyEnd(100),
	)
	if err != nil {
		log.Fatal(err)
		return
	}
	// 指定输出路径
	file, err := os.Create("report.html")
	if err != nil {
		log.Fatal(err)
		return
	}
	rp := printer.ReportPrinter{
		Out:    file,
		Report: report,
	}
	// 指定输出格式
	_ = rp.Print("html")
}
```

运行测试会在当前目录输出report.html文件

```bash
$ go run ghz.go
$ ls
ghz.go  report.html
```