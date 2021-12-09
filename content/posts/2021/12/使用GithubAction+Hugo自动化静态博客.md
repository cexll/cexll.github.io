---
title: "使用GithubAction+Hugo自动化静态博客"
subtitle: ""
date: 2021-12-04T16:45:37+08:00
lastmod: 2021-12-04T16:45:37+08:00
draft: true
author: "cexll"
authorLink: "cexll"
description: ""

tags: [
  hugo,action,github
]
categories: [
  blog
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
license: "MIT"
---

<!--more-->

> 综合情况考虑决定不续费服务器使用github.io来写博客,因为服务器基本处于一个闲置状态,如果不是因为写博客可能完全就荒废了,日常需要的各种特殊情况都能通过内网穿透解决,所以决定研究静态博客

# 安装 Hugo

进入 [官网](https://gohugo.io/getting-started/quick-start/)

ubuntu安装
```bash
apt install hugo
```

# 新建站点

```
hugo new site hello
```
进入站点 `cd hello`

站点目录结构
```
ls
 ▸ archetypes/
  ▸ content/
  ▸ layouts/
  ▸ static/
    config.toml
```

# 创建文章

```bash
hugo new hello.md
```

正文内容
```
+++
date = "2015-10-25T08:36:54-07:00"
draft = true
title = "about"

+++

正文内容
```

# 编译静态文件

```bash
hugo -D
```

# 启动http预览

```bash
hugo server -D
```

# 安装皮肤/主题

```bash
cd theme
git clone https://github.com/spf13/hyde.git
vim config.toml

theme = "hyde"
```
更多皮肤在这里 https://themes.gohugo.io/

# 配置GitHubAction
在`.github/workflows`创建文件 `hg-action.yml`
```yml
name: GitHub Pages

on:
  push:
    branches:
      - main  # Set a branch to deploy
  pull_request:

jobs:
  deploy:
    runs-on: ubuntu-20.04
    concurrency:
      group: ${{ github.workflow }}-${{ github.ref }}
    steps:
      - uses: actions/checkout@v2
        with:
          submodules: true  # Fetch Hugo themes (true OR recursive)
          fetch-depth: 0    # Fetch all history for .GitInfo and .Lastmod

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: '0.85.0'
          # extended: true

      - name: Build
        run: hugo -D

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        if: ${{ github.ref == 'refs/heads/main' }}
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
```
具体内容自行了解一下  https://github.com/peaceiris/actions-gh-pages

# 配置Gtihub变量

```yml
with:
  github_token: ${{ secrets.GITHUB_TOKEN }} //在个人设置SSH里面添加一个名称为GITHUB_TOKEN的密钥
  publish_dir: ./public
```
![演示](https://github.com/cexll/cexll.github.io/raw/main/images/2021/12/20211209104835.png)

# 完成
配置完成后 后面每次推送都会通过GithubAction编译好静态文件到分支下 `gh-pages`, 在将仓库设置静态访问分支
