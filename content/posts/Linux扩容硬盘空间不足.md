---
title: Linux扩容硬盘空间不足
subtitle: ""
date: 2021-06-24 13:43:46.385
lastmod: 2021-06-24 13:43:46.385
draft: true
author: "cexll"
authorLink: ""
description: ""

tags: [
    linux,扩容
]
categories: [
    linux
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




**Linux 在多个小硬盘下非常难受，如2个128G的固态，使用非常难受，今天就让它们组合起来使用，让多个硬盘组合成一个使用而不需要归成一个，我平常喜欢让一个硬盘挂载系统，另一个挂再home目录，组合使用更加舒服**

## 以下操作全部需要root执行: sudo

```bash
su root | sudo
```

## 1、查看硬盘信息

```bash
fdisk -l
```

## 2、进入磁盘，对磁盘进行分区

- 删除旧分区

```bash
fdisk [磁盘路径] | 例: /dev/sdb
m   // 帮助
d   // 删除分区
n   // 添加分区
p   // 创建逻辑分区
```

### 输入1，进入逻辑分区阶段

```bash
artion number(1-4)：1
这里需要一点fdisk基础，默认全部分区直接回车就行
```

### 保存 w

```bash
w
```

## 3、格式化分区&挂载

- 手动卸载分区

```bash
umount [磁盘路径]
```

- 将 `/dev/sdb1` 格式化为 `ext4` 类型

```bash
mkfs.ext4 /dev/sdb1
```

- 在根目录下创建一个目录

```bash
mkdir /[目录名]
```

- 挂载分区

```bash
mount /dev/sdb1 /[目录名]
```

- 查看硬盘和挂载分区等信息

```bash
df -h
```

## 4、设置开机自动挂载

- 查询磁盘UUID

```bash
ls -al /dev/disk/by-uuid
```

- 编辑 `/dev/fstab`

```bash
vim /etc/fstab
```

- 加入挂载磁盘的信息

```bash
UUID=xxx /[挂载目录] ext4(文件格式) defaults 0 0
```

- 重启

```bash
reboot
```

# 问题表

## 1、重启失败(命令行界面)

- 输入登录密码
- 检查修改磁盘挂载信息

```bash
vim /etc/fstab
```

注释掉之前添加的信息，再次重启

```bash
reboot
```
