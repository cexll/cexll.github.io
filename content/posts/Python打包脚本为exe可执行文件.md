---
title: Python打包脚本为exe可执行文件
subtitle: ""
date: 2021-06-24 13:31:00.743
lastmod: 2021-06-24 13:31:00.743
draft: true
author: "cexll"
authorLink: ""
description: ""

tags: [
    python,打包,exe
]
categories: [
    python
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




# 安装 pyinstaller 库
```python
pip install pyinstaller
or 这里推荐`pipenv`
pipenv install pyinstaller
```

# 打包

```python
pyinstaller -F hello.py
```

打包好的exe文件在当前目录dist目录内

打包时加上`-w`参数, 执行取消小黑窗
```python
pyinstaller -F -w hello.py
```
