---
title: "使用Golang的Channel实现PubSub"
subtitle: "利用Go管道实现订阅"
date: 2022-07-21T11:15:53+08:00
lastmod: 2022-07-21T11:15:53+08:00
draft: true
author: "cexll"
authorLink: "github.com/cexll"
description: "使用Golang的Channel实现PubSub"

tags: [golang,channel,pubsub]
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
license: "MIT"
---

<!--more-->

# 使用Golang的Channel实现PubSub

> 核心的订阅代码借鉴于Docker "github.com/docker/docker/pkg/pubsub"

先上代码

```go
import (
	"sync"
	"time"
)

var wgPool = sync.Pool{New: func() interface{} { return new(sync.WaitGroup) }}

// NewPublisher creates a new pub/sub publisher to broadcast messages.
// The duration is used as the send timeout as to not block the publisher publishing
// messages to other clients if one client is slow or unresponsive.
// The buffer is used when creating new channels for subscribers.
func NewPublisher(publishTimeout time.Duration, buffer int) *Publisher {
	return &Publisher{
		buffer:      buffer,
		timeout:     publishTimeout,
		subscribers: make(map[subscriber]topicFunc),
	}
}

type subscriber chan interface{}
type topicFunc func(v interface{}) bool

// Publisher is basic pub/sub structure. Allows to send events and subscribe
// to them. Can be safely used from multiple goroutines.
type Publisher struct {
	m           sync.RWMutex
	buffer      int
	timeout     time.Duration
	subscribers map[subscriber]topicFunc
}

// Len returns the number of subscribers for the publisher
func (p *Publisher) Len() int {
	p.m.RLock()
	i := len(p.subscribers)
	p.m.RUnlock()
	return i
}

// Subscribe adds a new subscriber to the publisher returning the channel.
func (p *Publisher) Subscribe() chan interface{} {
	return p.SubscribeTopic(nil)
}

// SubscribeTopic adds a new subscriber that filters messages sent by a topic.
func (p *Publisher) SubscribeTopic(topic topicFunc) chan interface{} {
	ch := make(chan interface{}, p.buffer)
	p.m.Lock()
	p.subscribers[ch] = topic
	p.m.Unlock()
	return ch
}

// SubscribeTopicWithBuffer adds a new subscriber that filters messages sent by a topic.
// The returned channel has a buffer of the specified size.
func (p *Publisher) SubscribeTopicWithBuffer(topic topicFunc, buffer int) chan interface{} {
	ch := make(chan interface{}, buffer)
	p.m.Lock()
	p.subscribers[ch] = topic
	p.m.Unlock()
	return ch
}

// Evict removes the specified subscriber from receiving any more messages.
func (p *Publisher) Evict(sub chan interface{}) {
	p.m.Lock()
	_, exists := p.subscribers[sub]
	if exists {
		delete(p.subscribers, sub)
		close(sub)
	}
	p.m.Unlock()
}

// Publish sends the data in v to all subscribers currently registered with the publisher.
func (p *Publisher) Publish(v interface{}) {
	p.m.RLock()
	if len(p.subscribers) == 0 {
		p.m.RUnlock()
		return
	}

	wg := wgPool.Get().(*sync.WaitGroup)
	for sub, topic := range p.subscribers {
		wg.Add(1)
		go p.sendTopic(sub, topic, v, wg)
	}
	wg.Wait()
	wgPool.Put(wg)
	p.m.RUnlock()
}

// Close closes the channels to all subscribers registered with the publisher.
func (p *Publisher) Close() {
	p.m.Lock()
	for sub := range p.subscribers {
		delete(p.subscribers, sub)
		close(sub)
	}
	p.m.Unlock()
}

func (p *Publisher) sendTopic(sub subscriber, topic topicFunc, v interface{}, wg *sync.WaitGroup) {
	defer wg.Done()
	if topic != nil && !topic(v) {
		return
	}

	// send under a select as to not block if the receiver is unavailable
	if p.timeout > 0 {
		timeout := time.NewTimer(p.timeout)
		defer timeout.Stop()

		select {
		case sub <- v:
		case <-timeout.C:
		}
		return
	}

	select {
	case sub <- v:
	default:
	}
}
```
Server代码

```go
import (
	"fmt"
	"sync"
	"time"
)

type Server struct {
	Bucket map[string]*Publisher
	m      sync.RWMutex
}

func NewServer() *Server {
	return &Server{
		Bucket: make(map[string]*Publisher),
	}
}

// 订阅主题
func (s *Server) Sub(subName string) {
	p := s.newPublisher(subName)
	c := p.SubscribeTopic(func(v interface{}) bool {
    // 判断是不是字符串
		if _, ok := v.(string); ok {
			return true
		}
		return false
	})
  // 为了实现阻塞 需要先发送一个ping来激活
	p.Publish("ping")
	select {
	case <-c:
		for v := range c {
      // 推送的内容
			fmt.Println(v)
		}
	}

}
// 推送内容
func (s *Server) Pub(subName string, v interface{}) {
	p := s.newPublisher(subName)
	p.Publish(v)
}
// 关闭订阅
func (s *Server) Close(subName string) {
	p := s.newPublisher(subName)

	p.Close()
}

//  关闭订阅
func (s *Server) UnSub(subName string) {
  // TODO 关闭订阅
  // 这里可以自己实现相关关闭订阅的业务代码
}

// 获取一个Publisher
func (s *Server) newPublisher(subName string) *Publisher {
	s.m.RLock()
	defer s.m.RUnlock()
	p := s.Bucket[subName]
	if p == nil {
		p = NewPublisher(100*time.Millisecond, 10)
		s.Bucket[subName] = p
	}
	return p
}
```

`Main.go`

```go
func main() {
  // 获取订阅服务
    s := NewServer()

	fmt.Println("server running...")
  // 订阅hello
	go s.Sub("hello")
  // 等待订阅完成
	time.Sleep(time.Millisecond * 10)

	i := 0
	for {
		i++
    // 循环向hello发送
		s.Pub("hello", fmt.Sprintf("啊哈哈哈 golang channel %d", i))
		if i > 100 {
			break
		}
	}
  // 等待订阅消费完
  time.Sleep(time.Millisecond * 100)
  // 关闭队列
	s.Close("hello")
}
```

测试结果
```
server running...
啊哈哈哈 golang channel 1
啊哈哈哈 golang channel 2
啊哈哈哈 golang channel 3
啊哈哈哈 golang channel 4
啊哈哈哈 golang channel 5
啊哈哈哈 golang channel 6
啊哈哈哈 golang channel 7
啊哈哈哈 golang channel 8
啊哈哈哈 golang channel 9
啊哈哈哈 golang channel 10
啊哈哈哈 golang channel 11
啊哈哈哈 golang channel 12
啊哈哈哈 golang channel 13
啊哈哈哈 golang channel 14
啊哈哈哈 golang channel 15
啊哈哈哈 golang channel 16
啊哈哈哈 golang channel 17
啊哈哈哈 golang channel 18
啊哈哈哈 golang channel 19
啊哈哈哈 golang channel 20
啊哈哈哈 golang channel 21
啊哈哈哈 golang channel 22
啊哈哈哈 golang channel 23
啊哈哈哈 golang channel 24
啊哈哈哈 golang channel 25
啊哈哈哈 golang channel 26
啊哈哈哈 golang channel 27
啊哈哈哈 golang channel 28
啊哈哈哈 golang channel 29
啊哈哈哈 golang channel 30
啊哈哈哈 golang channel 31
啊哈哈哈 golang channel 32
啊哈哈哈 golang channel 33
啊哈哈哈 golang channel 34
啊哈哈哈 golang channel 35
啊哈哈哈 golang channel 36
啊哈哈哈 golang channel 37
啊哈哈哈 golang channel 38
啊哈哈哈 golang channel 39
啊哈哈哈 golang channel 40
啊哈哈哈 golang channel 41
啊哈哈哈 golang channel 42
啊哈哈哈 golang channel 43
啊哈哈哈 golang channel 44
啊哈哈哈 golang channel 45
啊哈哈哈 golang channel 46
啊哈哈哈 golang channel 47
啊哈哈哈 golang channel 48
啊哈哈哈 golang channel 49
啊哈哈哈 golang channel 50
啊哈哈哈 golang channel 51
啊哈哈哈 golang channel 52
啊哈哈哈 golang channel 53
啊哈哈哈 golang channel 54
啊哈哈哈 golang channel 55
啊哈哈哈 golang channel 56
啊哈哈哈 golang channel 57
啊哈哈哈 golang channel 58
啊哈哈哈 golang channel 59
啊哈哈哈 golang channel 60
啊哈哈哈 golang channel 61
啊哈哈哈 golang channel 62
啊哈哈哈 golang channel 63
啊哈哈哈 golang channel 64
啊哈哈哈 golang channel 65
啊哈哈哈 golang channel 66
啊哈哈哈 golang channel 67
啊哈哈哈 golang channel 68
啊哈哈哈 golang channel 69
啊哈哈哈 golang channel 70
啊哈哈哈 golang channel 71
啊哈哈哈 golang channel 72
啊哈哈哈 golang channel 73
啊哈哈哈 golang channel 74
啊哈哈哈 golang channel 75
啊哈哈哈 golang channel 76
啊哈哈哈 golang channel 77
啊哈哈哈 golang channel 78
啊哈哈哈 golang channel 79
啊哈哈哈 golang channel 80
啊哈哈哈 golang channel 81
啊哈哈哈 golang channel 82
啊哈哈哈 golang channel 83
啊哈哈哈 golang channel 84
啊哈哈哈 golang channel 85
啊哈哈哈 golang channel 86
啊哈哈哈 golang channel 87
啊哈哈哈 golang channel 88
啊哈哈哈 golang channel 89
啊哈哈哈 golang channel 90
啊哈哈哈 golang channel 91
啊哈哈哈 golang channel 92
啊哈哈哈 golang channel 93
啊哈哈哈 golang channel 94
啊哈哈哈 golang channel 95
啊哈哈哈 golang channel 96
啊哈哈哈 golang channel 97
啊哈哈哈 golang channel 98
啊哈哈哈 golang channel 99
啊哈哈哈 golang channel 100
啊哈哈哈 golang channel 101
```

# 核心原理

```go
// 订阅
type subscriber chan interface{}
// 主题方法
type topicFunc func(v interface{}) bool

type Publisher struct {
	m           sync.RWMutex          // 读写锁
	buffer      int                   // 缓冲区
	timeout     time.Duration         // 超时时间
	subscribers map[subscriber]topicFunc  // 主题
}
```

Github地址
>  https://github.com/go-ll/pubsub