---
title: "利用Redis的Sub和Pub实现Im通讯"
subtitle: "利用sub/pub实现分布式im通讯,只需要增加redis集群数量和服务数量即可"
date: 2022-03-17T13:53:28+08:00
lastmod: 2022-03-17T13:53:28+08:00
draft: true
author: "cexll"
authorLink: ""
description: "Redis的Pub/Sub作为核心,WebSocket作为通讯实现分布式Im通讯"

tags: [分布式, redis, pub, sub]
categories: [php,分布式,redis]

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

使用WebSocket进行通讯(发送/接受)信息, Redis作为信息传递, Redis集群可参考之前的文章

理论上可无限扩容, 机器扛不住了直接加机器,Redis使用集群

这里讲解一下核心的功能

- 创建链接

![image.png](https://cdn.jsdelivr.net/gh/cexll/cexll.github.io/images/2022/03/1.png)

```php
public function onOpen($server, Request $request): void
{
    $server->push($request->fd, 'OpenSuccess');
    // 这里可鉴权 也可在基类进行操作, 我这里只作为演示
    $channel = $request->get['channel'] ?? null;
    if (! empty($channel)) {
        var_dump('fd:' . $request->fd . ' channel: ' . $channel);
        // 创建链接 订阅传入的订阅名称 正式环境可使用用户ID作为唯一Id 群聊可使用hashId 也可以直接使用群聊唯一ID 
        $sub = new Subscriber('127.0.0.1', 6379, '', 5);
        $sub->subscribe($channel);
        $chan = $sub->channel();
        // 订阅直接循环读取内容
        while (true) {
            $data = $chan->pop();
            if (empty($data)) { // 手动close与redis异常断开都会导致返回false
                if (! $sub->closed) {
                  // redis异常断开处理
                    var_dump('Redis connection is disconnected abnormally');
                }
                break;
            }
            try {
              // 拿到数据之后正常数据需要进行Json处理
                $payload = Json::decode($data->payload);
            } catch (\Throwable $e) {
                $payload = $data->payload;
            }

            // 有fd数据的直接通过Ws发送
            if (! empty($payload['fd'])) {
                $server->push((int) $payload['fd'], $payload['payload']);
            } else {
              // 没有fd则通过WebSocketClient走一遍
                $this->wsClient->push(Json::encode([
                    'channel' => $channel,
                    'payload' => mb_convert_encoding($data->payload, 'UTF-8', 'UTF-8'),
                    'fd' => $request->fd,
                ]));
            }
        }
    }
}
```
- 发送消息

![image.png](https://cdn.jsdelivr.net/gh/cexll/cexll.github.io/images/2022/03/220220317141458.png)

![image.png](https://cdn.jsdelivr.net/gh/cexll/cexll.github.io/images/2022/03/320220317141544.png)
```php
public function onMessage($server, Frame $frame): void
{
    // 接收到消息Json处理一下
    try {
        $data = Json::decode($frame->data);
    } catch (\Throwable $e) {
        $data = $frame->data;
    }
    // 正常的数据会走这里
    if (is_array($data)) {
        if (! empty($data['fd']) && $frame->fd === $data['fd']) {
            // 有fd直接发送
            $server->push($frame->fd, $data['payload']);
        } else {
            // 没有fd通过redis走一遍
            $this->redis->publish($data['channel'], $frame->data);
        }
    // 可能是其他或者测试就走这里
    } else {
        $server->push($frame->fd, $frame->data);
    }
}
```
- 断开链接
```php
public function onClose($server, int $fd, int $reactorId): void
{
  // 这里演示代码 断开链接之后取消订阅
  $channel = $request->get['channel'] ?? null;
  if (!empty($channel)) {
      $sub = new Subscriber('127.0.0.1', 6379, '', 5);
      $sub->unsubscribe($channel); // 断开订阅
  }
}
```

使用组件
- Mix/Subscriber
- Hyperf/WebSocket


