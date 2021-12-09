---
title: Linux/Win 配置使用终端代理
subtitle: ""
date: 2021-06-24 13:39:12.155
lastmod: 2021-06-24 13:39:12.155
draft: true
author: "cexll"
authorLink: ""
description: ""

tags: [
	linux,proxy
]
categories: [
	linux,proxy
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






# 在Linux终端使用SSR服务实现科学上网

## ssr 代理服务

### 下载

```bash
apt install -y git
git clone https://github.com/SAMZONG/gfwlist2privoxy.git
cd gfwlist2privoxy/
chmod +x ssr
mv ssr /usr/local/bin
```

### 安装配置

```bash
[root@localhost ~]$ ssr install
Cloning into '/usr/local/share/shadowsocksr'...
remote: Counting objects: 5490, done.
remote: Total 5490 (delta 0), reused 0 (delta 0), pack-reused 5490
Receiving objects: 100% (5490/5490), 1.71 MiB | 410.00 KiB/s, done.
Resolving deltas: 100% (3799/3799), done.

[root@localhost ~]$ ssr config 		# 配置文件路径 /usr/local/share/shadowsocksr/config.json
{
    "server": "0..0.0.0",	# ssr服务器ip
    "server_ipv6": "::",
    "server_port": 8080,	# ssr服务器端口
    "local_address": "127.0.0.1",
    "local_port": 1080,

    "password": "123456",		# 密码
    "method": "none",			# 加密方式
    "protocol": "auth_chain_a",		# 协议
    "protocol_param": "",          # 协议参数
    "obfs": "http_simple",		# 混淆
    "obfs_param": "hello.world",	# 混淆参数
    "speed_limit_per_con": 0,
    "speed_limit_per_user": 0,

    "additional_ports" : {}, // only works under multi-user mode
    "additional_ports_only" : false, // only works under multi-user mode
    "timeout": 120,
    "udp_timeout": 60,
    "dns_ipv6": false,
    "connect_verbose_info": 0,
    "redirect": "",
    "fast_open": false
}
```

### 启动/关闭

```bash
ssr start
ssr stop
```

### 卸载

```bash
ssr uninstall # 这里操作会删除/usr/local/share/shadowsocksr
```

以上，本地监听服务已经配置完成了，在填写的过程中，要注意你的本地监听地址和监听端口，默认是127.0.0.1:1080，如果你修改了设置，那么在后续配置中也要配合修改。

## Privoxy 配置

```bash
apt install -y privoxy
```

### 全局模式

代理模式同其他平台上方式，将所有http/https请求走代理服务，如果需要全局代理的话按照如下操作即可，如果要使用PAC模式，请跳过此部分。

```bash
# 添加本地ssr服务到配置文件
echo 'forward-socks5 / 127.0.0.1:1080 .' >> /etc/privoxy/config

# Privoxy 默认监听端口是是8118
export http_proxy=http://127.0.0.1:8118
export https_proxy=http://127.0.0.1:8118
export no_proxy=localhost

# 启动服务
systemctl start privoxy.service
```

### PAC模式

使用GFWList是由AutoProxy官方维护，由众多网民收集整理的一个中国大陆防火长城的屏蔽列表，这里感谢@Otokaze 为我们提供了转换shell自动转换脚本，为了方便修改，我fork了这个项目，将这篇教程所用到的资源进行了汇总，你可以在最开始git clone的目录中找到执行脚本。

```bash
[root@localhost ~]$ cd gfwlist2privoxy/
[root@localhost gfwlist2privoxy]$ ls
gfw.action  gfwlist2privoxy  README.md  ssr
[root@localhost gfwlist2privoxy]$ bash gfwlist2privoxy
proxy(socks5): 127.0.0.1:1080		# 注意，如果你修改了ssr本地监听端口是需要设置对应的
{+forward-override{forward-socks5 127.0.0.1:1080 .}}

=================================================================

"cp -af /root/gfwlist2privoxy/gfw.action /etc/privoxy/"

[root@localhost ~]$ cp -af gfw.action /etc/privoxy/
[root@localhost ~]$ echo 'actionsfile gfw.action' >> /etc/privoxy/config

# Privoxy 默认监听端口是是8118
export http_proxy=http://127.0.0.1:8118
export https_proxy=http://127.0.0.1:8118
export no_proxy=localhost

# 启动服务
systemctl start privoxy.service
```



### proxy 环境变量

```bash
# privoxy默认监听端口为8118
export http_proxy=http://127.0.0.1:8118
export https_proxy=http://127.0.0.1:8118
export no_proxy=localhost

# no_proxy是不经过privoxy代理的地址
# 只能填写具体的ip、域名后缀，多个条目之间使用','逗号隔开
# 比如: export no_proxy="localhost, 192.168.1.1, ip.cn, chinaz.com"
# 访问 localhost、192.168.1.1、ip.cn、*.ip.cn、chinaz.com、*.chinaz.com 将不使用代理
```

### 代理测试

```bash
# 访问各大网站，如果都有网页源码输出说明代理没问题
curl -sL www.baidu.com
curl -sL www.google.com
curl -sL www.google.com.hk
curl -sL www.google.co.jp
curl -sL www.youtube.com
curl -sL mail.google.com
curl -sL facebook.com
curl -sL twitter.com
curl -sL www.wikipedia.org

# 获取当前 IP 地址
# 如果使用 privoxy 全局模式，则应该显示 ss 服务器的 IP
# 如果使用 privoxy gfwlist模式，则应该显示本地公网 IP
curl -sL ip.chinaz.com/getip.aspx
curl https://httpbin.org/get
```

### 管理脚本

在以上部署操作完成后，应该已经可以正常科学上网了，但是如果需要进行管理时，需要分别管理ssr和privoxy，为了方便管理，这里写了一个shell脚本方便管理: ssr_manager

```bash
#!/bin/bash
# Author Samzong.lu

case $1 in
	start)
		ssr start &> /var/log/ssr-local.log
		systemctl start privoxy.service
		export http_proxy=http://127.0.0.1:8118
		export https_proxy=http://127.0.0.1:8118
		export no_proxy="localhost, ip.cn, chinaz.com"
		;;
	stop)
		unset http_proxy https_proxy no_proxy
		systemctl stop privoxy.service
		ssr stop &> /var/log/ssr-log.log
		;;
	autostart)
		echo "ssr start" >> /etc/rc.local
		systemctl enable privoxy.service
		echo "http_proxy=http://127.0.0.1:8118" >> /etc/bashrc
		echo "https_proxy=http://127.0.0.1:8118" >> /etc/bashrc
		echo "no_proxy='localhost, ip.cn, chinaz.com'" >> /etc/bashrc
		;;
	*)
		echo "usage: source $0 start|stop|autostart"
		exit 1
		;;
esac
```

### 使用

```bash
mv gfwlist2privoxy/ssr_manager /usr/local/bin
chmod +x ssr_manager

# 启动服务
ssr_manager start

# 关闭服务
ssr_manager stop 

# 添加开机自启动
ssr_manager autostart
```

### 局域网共享代理

```bash
vim /etc/privoxy/config # 找到list 8118 
修改 127.0.0.1:8118 为 0.0.0.0:8118 可实现局域网共享代理
ssr_manager stop
ssr_manager start
```

可通过手机代理设置 IP+端口 连接代理, 24小时随连

参考链接
> https://samzong.me/2017/11/17/howto-use-ssr-on-linux-terminal/
> https://www.zfl9.com/ss-local.html
> https://www.djangoz.com/2017/08/16/linux_setup_ssr/