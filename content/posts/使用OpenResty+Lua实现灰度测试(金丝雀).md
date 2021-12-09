---
title: 使用OpenResty+Lua实现灰度测试(金丝雀)
subtitle: ""
date: 2021-11-18 11:14:17.353
lastmod: 2021-11-18 11:14:17.353
draft: true
author: "cexll"
authorLink: ""
description: ""

tags: [
    openresty,lua,灰度测试,redis,nginx,gateway
]
categories: [
    nginx,openresty
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



> https://openresty.org/en/ 下载地址 如何安装部署不是本节内容

在实际项目中遇到重构或者新版本发布, 新老系统如何高效的切换,现目前的答案就是`Gateway` 网关,有很多开源的网关`Kong` `Apisix` 但是这里来教如何自己实现一个api网关

# 介绍
`openresty` 基于 `Nginx` 开发 使用 `Lua` 让程序更加灵活, `Lua`基于`C`开发,脚本语言 拥有原生协程, 性能强 自己简单看一下[lua的语法](https://www.runoob.com/lua/lua-tutorial.html)我们就开始上手了

# 思路

网关实现思路其实很简单,请求来到`openresty`然后通过`Lua`脚本解析相关`route`/ `queryParmes` / `Body` 内容,然后做出想要的结果, 比如我想要`/abc`返回 
```json
{
    "msg": "helloworld"
}
```
或者`GET`返回`1`, `POST`返回`2`, 因为是 `luaJit`耗时很低

# 代码
将通过 `redis`来实现访问指定路由后请求新项目, 没有则访问老项目

```lua
local re_uri = ngx.var.request_uri // 获取请求的uri
local re_method = ngx.var.request_method // 获取请求的方式
if re_method == "GET" then // 这里判断如果是get请求就判断一下有没有传参 有传参就把后面的参数去掉 因为只需要保存 uri
    local cat_len = string.find(re_uri, "?")
    -- ngx.say(type(cat_len))
    if type(cat_len) == "number" and cat_len > 0 then
        re_uri = string.sub(re_uri, 0, cat_len-1)
    end
end
// 开始连接redis
local function close_redis(redis_instance)
    if not redis_instance then
        return
    end
    local ok,err = redis_instance:close();
    if not ok then
        ngx.say("close redis error : ",err);
    end
end

-- 连接redis
local redis = require("resty.redis");
-- 创建redis 对象
local redis_instance = redis:new();
redis_instance:set_timeout(1000)

local ok,err = redis_instance:connect('127.0.0.1',6379)
if not ok then
    ngx.say("connect redis error : ",err)
    return close_redis(redis_instance);
end

--Redis身份验证
local auth,err = redis_instance:auth(""); // 如果没有密码可以注释掉这一段代码 
if not auth then
    ngx.say("failed to authenticate : ",err)
end

-- 连接成功
local resp,err = redis_instance:set("lua","hello world") // 连接成功写入一个 lua helloworld
if not resp then
    ngx.say("set msg error : ",err)
    return close_redis(redis_instance)
end


-- 获取API是否存在 
local resp, err = redis_instance:get("gateway:"..re_uri)
if not resp then
    ngx.say("get msg error : ", err)
    return close_redis(redis_instance)
end

// 如果不存在则访问老项目
if resp == nil then
    ngx.exec("@old");
end
if resp == "1" then
    ngx.exec("@new"); // 如果存在则访问新项目
else
    ngx.exec("@old");
end
close_redis(redis_instance) // 结束之后关闭redis连接
```

# 配置 openresty
`lua`脚本写好之后需要在`openresty`配置加上, 因为是基于nginx开发的所以配置和nginx是差不多的

vim nginx.conf
```conf
location / { // 访问路由则加载lua脚本
      add_header content-type application/json;
      content_by_lua_file /www/lua/gateway.lua;
    }
    
    location @old{ // 旧项目
      try_files $uri =404;
      fastcgi_pass  unix:/tmp/php-cgi-72.sock;
      fastcgi_index index.php;
      include fastcgi.conf;
      include pathinfo.conf;
    }
    
    location @new{ // 新项目
      proxy_http_version 1.1;
      proxy_set_header Connection "keep-alive";
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header REMOTE-HOST $remote_addr;
      proxy_pass http://127.0.0.1:13001;
    }
```

# 重载配置
```
openresty reload
```
# 结束

```
curl http://localhost/

hello OpenResty
```