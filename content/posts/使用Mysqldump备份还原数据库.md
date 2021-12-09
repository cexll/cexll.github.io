---
title: 使用Mysqldump备份还原数据库
subtitle: ""
date: 2021-06-24 13:30:04.046
lastmod: 2021-06-24 13:30:04.046
draft: true
author: "cexll"
authorLink: ""
description: ""

tags: [
    mysql,mysqldump,备份,还原
]
categories: [
    mysql
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


# 备份数据库


```mysql
mysqldump -uroot -p123456 -P3306 test -t > test.sql
-u 登录数据库账号
-p 登录数据库密码
-P 连接数据库端口
test 为 test 数据库
test.sql 输出到当前目录
```

# 备份此链接下所有数据库
使用`-A`参数,表示备份所有数据库(结构和数据)
```mysql
mysqldump -uroot -p123456 -P3306 -A > all.sql
```

# 备份所有数据库(仅结构)
加入`-d`参数表示只备份结构
```mysql
mysqldump -uroot -p123456 -P3306 -A -d > all.sql
```

# 还原数据库
需要先登录数据库中
```mysql
mysql -uroot -p123456
# 执行还原
source all.sql
```
