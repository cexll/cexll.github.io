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

测试代码

```go
func TestRunPubSub(t *testing.T) {
	s := NewPublisherServer()

	fmt.Println("server running...")
	// 多个订阅者
	go s.Sub("hello")
	go s.Sub("golang")
	go s.Sub("php")

	time.Sleep(time.Millisecond * 10)

	go func() {
		i := 0
		for {
			i++
			s.Pub("hello", fmt.Sprintf("hello channel %d", i))
			if i > 10 {
				break
			}
		}
	}()

	go s.Pub("golang", "golang 123")
	go s.Pub("php", "php 123")
	time.Sleep(time.Millisecond * 100)
	s.Close("hello")
	s.Close("golang")
	s.Close("php")
}
```

测试结果
```
=== RUN   TestRunPubSub
server running...
golang 123
hello channel 1
hello channel 2
hello channel 3
hello channel 4
hello channel 5
hello channel 6
hello channel 7
hello channel 8
hello channel 9
hello channel 10
hello channel 11
php 123
--- PASS: TestRunPubSub (0.13s)
PASS


进程 已完成，退出代码为 0

```

测试代码2 启动一个TCP监听9898端口 使用终端命令telnet链接测试

```go
func TestTCPPubSub(t *testing.T) {
	listen, err := net.Listen("tcp", "0.0.0.0:9898")
	if err != nil {
		panic(err)
	}
	fmt.Println("server running...")
	p := GetPublisherInstance()
	for {
		conn, err := listen.Accept()
		if err != nil {
			continue
		}
		go handleRequest(conn, p)
	}
}

var Conns = make(map[string]net.Conn)

func handleRequest(conn net.Conn, p *Publisher) {
	for {
		bytes, _, _ := bufio.NewReader(conn).ReadLine()
		fmt.Println(fmt.Sprintf("request string: [%s]", string(bytes)))
		content := strings.Split(string(bytes), " ")
		if content[0] == "subscribe" {
			Conns[conn.RemoteAddr().String()] = conn
			topic := content[1]
			Conns[topic] = conn
			go func() {
				c := p.SubscribeTopic(func(v interface{}) bool {
					if s, ok := v.(string); ok && strings.Contains(s, topic) {
						return true
					}
					return false
				})
				for v := range c {
					for k, conn2 := range Conns {
						if k == topic {
							conn2.Write([]byte(fmt.Sprintf("%s topic: %v", topic, v)))
						}
					}
				}
			}()
		} else if content[0] == "publish" {
			topic := content[1]
			go p.Publish(topic + content[2] + "\n")
		} else if content[1] == "quit" {
			topic := content[1]
			c := p.SubscribeTopic(func(v interface{}) bool {
				if s, ok := v.(string); ok && strings.Contains(s, topic) {
					return true
				}
				return false
			})
			p.Evict(c)
			break
		} else {
			fmt.Println("common chat " + string(bytes))
			break
		}
	}
}
```

测试结果
```
订阅hello
$ telnet 127.0.0.1 9898
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
subscribe hello
hello topic: hello123
hello topic: hello123
hello topic: hello123

订阅golang
telnet 127.0.0.1 9898
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
subscribe golang
golang topic: golang👍123
golang topic: golang👍123
golang topic: golang👍123

向hello发送123
telnet 127.0.0.1 9898
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
publish hello 123
publish hello 123
publish hello 123

向golang发送👍123
publish golang 👍123
publish golang 👍123
publish golang 👍123
```
# 核心原理

```go
// 订阅
type subscriber chan interface{}
// 主题函数
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