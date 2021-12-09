---
title: NPS内网穿透
subtitle: ""
date: 2021-06-24 13:41:35.759
lastmod: 2021-06-24 13:41:35.759
draft: true
author: "cexll"
authorLink: ""
description: ""

tags: [
    nps,内网穿透
]
categories: [
   nps,内网穿透
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





## 一、NPS的GitHub页面地址

GitHub：https://github.com/cnlh/nps

## 二、进入releases安装页面

releases github:https://github.com/cnlh/nps/releases

找到你需要的版本，我这里用的服务器是ubuntu16.04，那么也就是Linux的64位版本，所以我需要下载对应的客户端和服务端。

![1.png](http://192.168.1.41:8090/upload/2020/1/1-745f51b0788b4708b13907263ac56ec3.png#vwid=1024&vhei=736)

## 三、登录你的服务器的SSH配置NPS服务端

1.我这里用的阿里云，没有安装wget，就安装一下。

```bash
# Debian/Ubuntu: apt install wget -y
# Centos/Rhel: yum install wget -y
```

2.从github上获取最新版本的nps服务端

```bash
wget https://github.com/cnlh/nps/releases/download/v0.23.1/linux_amd64_server.tar.gz
```

3.进行解压

```bash
tar xvf linux_amd64_server.tar.gz
```

4.启动服务端

```bash
./nps start
```

## 四、输入服务器的IP地址加8080端口号，即可进入NPS的后台界面

![2.png](http://192.168.1.41:8090/upload/2020/1/2-9785ff895bdc42e9b846ebc2d2308303.png#vwid=1024&vhei=901)

默认用户名：admin 默认密码：123


## 五、登录后台，添加一条客户端

![3.png](http://192.168.1.41:8090/upload/2020/1/3-23202b4b51f84d0a98148b7ba7ff0ecc.png#vwid=1024&vhei=884)![4.png](http://192.168.1.41:8090/upload/2020/1/4-789758682c4f4765954941201b0fe6b1.png#vwid=1024&vhei=615)![5.png](http://192.168.1.41:8090/upload/2020/1/5-37bcbc083fc84a27bc794a6d2c0ec364.png#vwid=1024&vhei=216)


## 六、配置客户端

1. SFTP功能 (这里使用香橙派)

```bash
apt update
apt install openssh-sftp-server
```

2. 下载NPS客户端(自己是啥就下载啥)

![6.png](http://192.168.1.41:8090/upload/2020/1/6-29fbcd6e08ce4a4da73e6dc6f1b8b39c.png#vwid=819&vhei=152)

```bash
wget https://github.com/cnlh/nps/releases/download/v0.23.1/linux_arm64_client.tar.gz
```

3. 解压

```bash
tar xvf linux_arm64_client.tar.gz
```

## 七、启动客户端

1.临时启动客户端测试

```bash
./npc -server=(ip:port) -vkey=(web界面中显示的密钥)
```

例子：

```bash
./npc -server=35.221.192.140:8024 -vkey=123456
```

![7.png](http://192.168.1.41:8090/upload/2020/1/7-74f18a9fff8d47a099af6d9e4eb7fb80.png#vwid=1024&vhei=204)

> 临时连接成功 客户端显示online

2.常驻客户端后台

```bash
nohup ./npc -server=(ip:port) -vkey=(web界面中显示的密钥)
```

## 八、解析域名并绑定

1.进入域名后台解析一个域名到你的服务端的IP上

![8.png](http://192.168.1.41:8090/upload/2020/1/8-cdd1735f0b6d404ba4cb43bd28fabbb9.png#vwid=1024&vhei=119)

2.进入NPS后台绑定域名以及设置内网IP及端口号

![9.png](http://192.168.1.41:8090/upload/2020/1/9-0c2fdb8cc79a42f3b14082cfd00560e7.png#vwid=1024&vhei=414)

![10.png](http://192.168.1.41:8090/upload/2020/1/10-097e42d817b64237ad599315146d42d6.png#vwid=1024&vhei=721)
![11.png](http://192.168.1.41:8090/upload/2020/1/11-15d1f9dd1b5146948604f89ada5bf2f8.png#vwid=1024&vhei=203)

> 域名这里显示online，就说明绑定成功，已经可以穿透访问了