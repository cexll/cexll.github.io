---
title: "OpenResty 应用场景"
subtitle: "介绍OpenResty的相关应用场景 例 限流 限速 缓存 IP访问限制 鉴权 网关 灰度 WAF"
date: 2022-08-19T09:40:09+08:00
lastmod: 2022-08-19T09:40:09+08:00
draft: true
author: "cexll"
authorLink: ""
description: ""

tags: [openresty, nginx, lua, gateway]
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

# OpenResty 应用场景

OpenResty 是什么 , 基于 Nginx + Lua 实现的一个全新产品 , 或者叫生态

## 功能介绍

- lua-resty-core

利用 core 我们可以实现很多操作 在 Nginx 就将一些业务等等操作提前初始化或者完成了也可以配合其他组件实现更复杂的功能

```lua
// 哈希
ngx.md5 
ngx.md5_bin
ngx.sha1_bin
// base64
ngx.encode_base64
ngx.decode_base64
// request
ngx.req.get_headers
ngx.req.get_uri_args
ngx.req.get_method
```
更多了解 https://github.com/openresty/lua-resty-core

- lua-resty-redis

利用 redis 我们可以实现鉴权 缓存 分布式锁等等操作 我们可以在 Nginx 链接 Redis 提前做到读取库存 订单之类的高并发操作然后提前做处理

```lua
local redis = require "resty.redis"
                local red = redis:new()

                red:set_timeouts(1000, 1000, 1000) -- 1 sec

                -- or connect to a unix domain socket file listened
                -- by a redis server:
                --     local ok, err = red:connect("unix:/path/to/redis.sock")

                -- connect via ip address directly
                local ok, err = red:connect("127.0.0.1", 6379)

                -- or connect via hostname, need to specify resolver just like above
                local ok, err = red:connect("redis.openresty.com", 6379)

                if not ok then
                    ngx.say("failed to connect: ", err)
                    return
                end

                ok, err = red:set("dog", "an animal")
                if not ok then
                    ngx.say("failed to set dog: ", err)
                    return
                end

                ngx.say("set result: ", ok)

                local res, err = red:get("dog")
                if not res then
                    ngx.say("failed to get dog: ", err)
                    return
                end

                if res == ngx.null then
                    ngx.say("dog not found.")
                    return
                end

                ngx.say("dog: ", res)

                red:init_pipeline()
                red:set("cat", "Marry")
                red:set("horse", "Bob")
                red:get("cat")
                red:get("horse")
                local results, err = red:commit_pipeline()
                if not results then
                    ngx.say("failed to commit the pipelined requests: ", err)
                    return
                end

                for i, res in ipairs(results) do
                    if type(res) == "table" then
                        if res[1] == false then
                            ngx.say("failed to run command ", i, ": ", res[2])
                        else
                            -- process the table value
                        end
                    else
                        -- process the scalar value
                    end
                end

                -- put it into the connection pool of size 100,
                -- with 10 seconds max idle time
                local ok, err = red:set_keepalive(10000, 100)
                if not ok then
                    ngx.say("failed to set keepalive: ", err)
                    return
                end
```

更多了解 https://github.com/openresty/lua-resty-redis

- lua-resty-websocket

利用 websocket 我们可以提前处理相关信息进行违规判断或者压力分担

```lua
local server = require "resty.websocket.server"

    local wb, err = server:new{
        timeout = 5000,  -- in milliseconds
        max_payload_len = 65535,
    }
    if not wb then
        ngx.log(ngx.ERR, "failed to new websocket: ", err)
        return ngx.exit(444)
    end

    local data, typ, err = wb:recv_frame()

    if not data then
        if not string.find(err, "timeout", 1, true) then
            ngx.log(ngx.ERR, "failed to receive a frame: ", err)
            return ngx.exit(444)
        end
    end

    if typ == "close" then
        -- for typ "close", err contains the status code
        local code = err

        -- send a close frame back:

        local bytes, err = wb:send_close(1000, "enough, enough!")
        if not bytes then
            ngx.log(ngx.ERR, "failed to send the close frame: ", err)
            return
        end
        ngx.log(ngx.INFO, "closing with status code ", code, " and message ", data)
        return
    end

    if typ == "ping" then
        -- send a pong frame back:

        local bytes, err = wb:send_pong(data)
        if not bytes then
            ngx.log(ngx.ERR, "failed to send frame: ", err)
            return
        end
    elseif typ == "pong" then
        -- just discard the incoming pong frame

    else
        ngx.log(ngx.INFO, "received a frame of type ", typ, " and payload ", data)
    end

    wb:set_timeout(1000)  -- change the network timeout to 1 second

    bytes, err = wb:send_text("Hello world")
    if not bytes then
        ngx.log(ngx.ERR, "failed to send a text frame: ", err)
        return ngx.exit(444)
    end

    bytes, err = wb:send_binary("blah blah blah...")
    if not bytes then
        ngx.log(ngx.ERR, "failed to send a binary frame: ", err)
        return ngx.exit(444)
    end

    local bytes, err = wb:send_close(1000, "enough, enough!")
    if not bytes then
        ngx.log(ngx.ERR, "failed to send the close frame: ", err)
        return
    end
```

更多了解 https://github.com/openresty/lua-resty-websocket

- lua-resty-limit-traffic

利用 limit 可以实现限速 限流 控流 可以说是比较常用的功能了


```lua
local limit_req = require "resty.limit.req"

                -- limit the requests under 200 req/sec with a burst of 100 req/sec,
                -- that is, we delay requests under 300 req/sec and above 200
                -- req/sec, and reject any requests exceeding 300 req/sec.
                local lim, err = limit_req.new("my_limit_req_store", 200, 100)
                if not lim then
                    ngx.log(ngx.ERR,
                            "failed to instantiate a resty.limit.req object: ", err)
                    return ngx.exit(500)
                end

                -- the following call must be per-request.
                -- here we use the remote (IP) address as the limiting key
                local key = ngx.var.binary_remote_addr
                local delay, err = lim:incoming(key, true)
                if not delay then
                    if err == "rejected" then
                        return ngx.exit(503)
                    end
                    ngx.log(ngx.ERR, "failed to limit req: ", err)
                    return ngx.exit(500)
                end

                if delay >= 0.001 then
                    -- the 2nd return value holds the number of excess requests
                    -- per second for the specified key. for example, number 31
                    -- means the current request rate is at 231 req/sec for the
                    -- specified key.
                    local excess = err

                    -- the request exceeding the 200 req/sec but below 300 req/sec,
                    -- so we intentionally delay it here a bit to conform to the
                    -- 200 req/sec rate.
                    ngx.sleep(delay)
                end
```

更多了解 https://github.com/openresty/lua-resty-limit-traffic

- lua-resty-mysql

看名字就知道是用来操作 MySQL 的我们在 Nginx 提前链接MySQL 进行一些前置处理 

```lua
 local cjson = require "cjson"
    local mysql = require "resty.mysql"

    local db = mysql:new()
    local ok, err, errcode, sqlstate = db:connect({
        host = "127.0.0.1",
        port = 3306,
        database = "world",
        user = "monty",
        password = "pass"})

    if not ok then
        ngx.log(ngx.ERR, "failed to connect: ", err, ": ", errcode, " ", sqlstate)
        return ngx.exit(500)
    end

    res, err, errcode, sqlstate = db:query("select 1; select 2; select 3;")
    if not res then
        ngx.log(ngx.ERR, "bad result #1: ", err, ": ", errcode, ": ", sqlstate, ".")
        return ngx.exit(500)
    end

    ngx.say("result #1: ", cjson.encode(res))

    local i = 2
    while err == "again" do
        res, err, errcode, sqlstate = db:read_result()
        if not res then
            ngx.log(ngx.ERR, "bad result #", i, ": ", err, ": ", errcode, ": ", sqlstate, ".")
            return ngx.exit(500)
        end

        ngx.say("result #", i, ": ", cjson.encode(res))
        i = i + 1
    end

    local ok, err = db:set_keepalive(10000, 50)
    if not ok then
        ngx.log(ngx.ERR, "failed to set keepalive: ", err)
        ngx.exit(500)
    end
```

更多了解 https://github.com/openresty/lua-resty-mysql

- lua-resty-logger-socket

cloudflare 实现的日志模块 可以在 Nginx 收集日志 适合 CDN 网关 相关的使用

```lua
local logger = require "resty.logger.socket"
                if not logger.initted() then
                    local ok, err = logger.init{
                        host = 'xxx',
                        port = 1234,
                        flush_limit = 1234,
                        drop_limit = 5678,
                    }
                    if not ok then
                        ngx.log(ngx.ERR, "failed to initialize the logger: ",
                                err)
                        return
                    end
                end

                -- construct the custom access log message in
                -- the Lua variable "msg"

                local bytes, err = logger.log(msg)
                if err then
                    ngx.log(ngx.ERR, "failed to log message: ", err)
                    return
                end
```

更多了解 https://github.com/cloudflare/lua-resty-logger-socket


应用场景还有很多 可以 操作MongoDB 操作ETCD HTTP请求 链接kafka投递消息 链接RabbitMQ  直播流rtmp grpc网关 

可以用lua来做很多事 , 但是也不能很多事都用它来做 毕竟它是动态语言 bilibili 2021-07-13 就是lua脚本死循环导致nginx百分百占用导致崩溃
## Lua
介绍强大的 Lua 语言, 在各个领域能发挥的强度

1. 在游戏服务端混得不错
2. 在HTTP层混得不错
3. 嵌入脚本
  
列举一些案例 

- Connmix
> OpenMix 作者做的一个全新产品 ConnMix `面向消息编程的分布式长连接引擎` 运行在 Go 上, Lua 来处理 HTTP WebSocket TCP UDP 等, 这些协议支持多少在于你能利用 Lua 写出来多少 

APISIX 网关 
> 云原生生态内部含有非常多功能 性能强劲 
