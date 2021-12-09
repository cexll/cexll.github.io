---
title: Github上fork项目与源项目同步
subtitle: ""
date: 2021-06-24 13:42:04.113
lastmod: 2021-06-24 13:42:04.113
draft: true
author: "cexll"
authorLink: ""
description: ""

tags: [
    git,github
]
categories: [
    git,github
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





fork项目之后将fork后的克隆到本地

比如 项目`https://github.com/test/test` fork 之后 在你的项目内 `https://github.com/youname/test`

```bash
# 克隆到本地
git clone https://github.com/youname/test
cd test
```

添加test的项目地址,也就是主项目的remote地址

```bash
git remote add test https://github.com/test/test # 这里 test 为别名也可以 v1.0 v2.0
git fetch test # test 相当于一个别名
git remote -v # 查看本地项目目录
git checkout master # 切换分支 
git merge test/master # 合并分支
```

如果有冲突,需要先丢掉本地的

```bash
git reset -hard test/master
```

然后就将更新后的项目提交到自己github

```bash
git commit -am "update"
git push origin
git push -u origin master -f # 强制提交
```
