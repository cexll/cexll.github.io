---
title: 使用Docker-compose 搭建 MySQL 主从 集群
subtitle: ""
date: 2021-10-30 13:58:56.785
lastmod: 2021-10-30 13:58:56.785
draft: true
author: "cexll"
authorLink: ""
description: ""

tags: [
  docker,docker-compose,集群,mysql,分布式,主从
]
categories: [
  mysql,docker
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


因为Docker的存在我们可以在不需要考虑环境的情况下安装各种软件

# 准备工作
### 编写`dockercompose.yml`

```yaml
version: "3.7"
services:
  db1:
    image: mysql:5.7
    command: --default-authentication-plugin=mysql_native_password
    ports:
      - "13307:3306"
    environment:
      MYSQL_ROOT_PASSWORD: root
    volumes:
      - "./conf/db1/mysql.cnf:/etc/mysql/conf.d/mysql.cnf"
      - "./data/db1:/var/lib/mysql/"
    expose:
      - 3306
  db2:
    image: mysql:5.7
    command: --default-authentication-plugin=mysql_native_password
    ports:
      - "13308:3306"
    environment:
      MYSQL_ROOT_PASSWORD: root
    volumes:
      - "./conf/db2/mysql.cnf:/etc/mysql/conf.d/mysql.cnf"
      - "./data/db2:/var/lib/mysql/"
    expose:
      - 3306
```
在当前文件夹下创建两个文件, `conf`, `data`, 用来存放配置和数据库文件

这里会启动两个service 名为 `db1` `db2`, 对外提供端口`13307`,`13308`, `expose`是对内提供端口,让容器与容器之间可以通信

### 编辑 `mysql.cnf`
在`conf`目录创建两个目录`db1` `db2` 用来区分两个db

`db1` 我们用来当`master`即我们的主库
```conf
[client]
port                    = 3306
default-character-set   = utf8mb4

[mysqld]
user                    = mysql
port                    = 3306
sql_mode                = ""
server_id               = 1 ## 全局id 这里用来区分
log_bin                 = mysql-bin
# 需要同步的数据库，如果不配置则同步全部数据库
#binlog-do-db=test_db
expire_logs_days        = 10  ## 存放天数 超过天数丢弃
binlog_format           = ROW ## 日志格式

default-storage-engine  = InnoDB
default-authentication-plugin   = mysql_native_password
character-set-server    = utf8mb4
collation-server        = utf8mb4_unicode_ci
init_connect            = 'SET NAMES utf8mb4'


slow_query_log
long_query_time         = 3
slow-query-log-file     = /var/lib/mysql/mysql.slow.log
log-error               = /var/lib/mysql/mysql.error.log

default-time-zone       = '+8:00'

[mysql]
default-character-set   = utf8mb4
```

编写`db2`的配置 即从库 `slave`

```conf
[client]
port                    = 3306
default-character-set   = utf8mb4

[mysqld]
user                    = mysql
port                    = 3306
sql_mode                = ""
server_id               = 2 ## 全局id 不能和其他库的一样
log_bin                 = mysql-bin
binlog_format           = ROW 
log_slave_updates        = ON ## 默认没有开启, 是从库自己也更新binlog
read_only                = ON ## 从库只读模式
super_read_only            = ON ## 通用只读但是是super也一样


default-storage-engine  = InnoDB
default-authentication-plugin   = mysql_native_password
character-set-server    = utf8mb4
collation-server        = utf8mb4_unicode_ci
init_connect            = 'SET NAMES utf8mb4'

slow_query_log
long_query_time         = 3
slow-query-log-file     = /var/lib/mysql/mysql.slow.log
log-error               = /var/lib/mysql/mysql.error.log

default-time-zone       = '+8:00'

[mysql]
default-character-set   = utf8mb4
```

# 开始工作

准备工作结束之后就可以开始了
 
启动 `mysql`
```bash
docker-compose up -d 
```

运行完之后我们就启动了两个`db` 这时候我们先设置 `master`

```bash
mysql -u root -P 13307 -h db1 # 这里db1根据你们自己的ip填写
## 先创建一个从库同步用户
GRANT REPLICATION SLAVE ON *.* to 'slave_user2'@'db2' identified by '123456'; # 这里db2 换成db2的IP
FLUSH PRIVILEGES; ## 刷新并应用
```

查看`master`信息

```mysql
show master status;

+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000004 | 423      |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.02 sec)
```

接来下编辑从库
```bash
mysql -u root -P 13308 -h db2 
## 从库只需要指定master是谁就可以
change master to master_host='db1',master_port=3306,master_user='slave_user2',master_password='123456',master_log_file='mysql-bin.000004',master_log_pos=423; ## 容器内可以通过名称来指定 因为docker内部有自己的dns指定了对应的ip, 正常情况都是指定IP, master_log_file填写master上打印的信息master_log_pos也一样
start slave; ## 启动
```

这时候就搞定了 

```mysql
show slave status; ## 查看从库状态
+----------------------------------+-------------+-------------+-------------+---------------+------------------+---------------------+-------------------------------+---------------+-----------------------+------------------+-------------------+-----------------+---------------------+--------------------+------------------------+-------------------------+-----------------------------+------------+------------+--------------+---------------------+-----------------+-----------------+----------------+---------------+--------------------+--------------------+--------------------+-----------------+-------------------+----------------+-----------------------+-------------------------------+---------------+---------------+----------------+----------------+-----------------------------+------------------+--------------------------------------+----------------------------+-----------+---------------------+--------------------------------------------------------+--------------------+-------------+-------------------------+--------------------------+----------------+--------------------+--------------------+-------------------+---------------+----------------------+--------------+--------------------+
| Slave_IO_State                   | Master_Host | Master_User | Master_Port | Connect_Retry | Master_Log_File  | Read_Master_Log_Pos | Relay_Log_File                | Relay_Log_Pos | Relay_Master_Log_File | Slave_IO_Running | Slave_SQL_Running | Replicate_Do_DB | Replicate_Ignore_DB | Replicate_Do_Table | Replicate_Ignore_Table | Replicate_Wild_Do_Table | Replicate_Wild_Ignore_Table | Last_Errno | Last_Error | Skip_Counter | Exec_Master_Log_Pos | Relay_Log_Space | Until_Condition | Until_Log_File | Until_Log_Pos | Master_SSL_Allowed | Master_SSL_CA_File | Master_SSL_CA_Path | Master_SSL_Cert | Master_SSL_Cipher | Master_SSL_Key | Seconds_Behind_Master | Master_SSL_Verify_Server_Cert | Last_IO_Errno | Last_IO_Error | Last_SQL_Errno | Last_SQL_Error | Replicate_Ignore_Server_Ids | Master_Server_Id | Master_UUID                          | Master_Info_File           | SQL_Delay | SQL_Remaining_Delay | Slave_SQL_Running_State                                | Master_Retry_Count | Master_Bind | Last_IO_Error_Timestamp | Last_SQL_Error_Timestamp | Master_SSL_Crl | Master_SSL_Crlpath | Retrieved_Gtid_Set | Executed_Gtid_Set | Auto_Position | Replicate_Rewrite_DB | Channel_Name | Master_TLS_Version |
+----------------------------------+-------------+-------------+-------------+---------------+------------------+---------------------+-------------------------------+---------------+-----------------------+------------------+-------------------+-----------------+---------------------+--------------------+------------------------+-------------------------+-----------------------------+------------+------------+--------------+---------------------+-----------------+-----------------+----------------+---------------+--------------------+--------------------+--------------------+-----------------+-------------------+----------------+-----------------------+-------------------------------+---------------+---------------+----------------+----------------+-----------------------------+------------------+--------------------------------------+----------------------------+-----------+---------------------+--------------------------------------------------------+--------------------+-------------+-------------------------+--------------------------+----------------+--------------------+--------------------+-------------------+---------------+----------------------+--------------+--------------------+
| Waiting for master to send event | db1         | slave_user2 |        3306 |            60 | mysql-bin.000004 |            11760423 | 258ee9abda4b-relay-bin.000002 |      11760153 | mysql-bin.000004      | Yes              | Yes               |                 |                     |                    |                        |                         |                             |          0 |            |            0 |            11760423 |        11760367 | None            |                |             0 | No                 |                    |                    |                 |                   |                |                     0 | No                            |             0 |               |              0 |                |                             |                1 | 16e6b1d7-393e-11ec-a290-0242ac190003 | /var/lib/mysql/master.info |         0 | NULL                | Slave has read all relay log; waiting for more updates |              86400 |             |                         |                          |                |                    |                    |                   |             0 |                      |              |                    |
+----------------------------------+-------------+-------------+-------------+---------------+------------------+---------------------+-------------------------------+---------------+-----------------------+------------------+-------------------+-----------------+---------------------+--------------------+------------------------+-------------------------+-----------------------------+------------+------------+--------------+---------------------+-----------------+-----------------+----------------+---------------+--------------------+--------------------+--------------------+-----------------+-------------------+----------------+-----------------------+-------------------------------+---------------+---------------+----------------+----------------+-----------------------------+------------------+--------------------------------------+----------------------------+-----------+---------------------+--------------------------------------------------------+--------------------+-------------+-------------------------+--------------------------+----------------+--------------------+--------------------+-------------------+---------------+----------------------+--------------+--------------------+
```

如果`Last_IO_Error`存在数据 或者 同步不成功等问题可以先查看`server_id`是否设置, or 手动到从库查看能否连接到主库

主节点查看所有从节点

```
SHOW SLAVE HOSTS;
+-----------+------+------+-----------+--------------------------------------+
| Server_id | Host | Port | Master_id | Slave_UUID                           |
+-----------+------+------+-----------+--------------------------------------+
|         2 |      | 3306 |         1 | 16d57f3c-393e-11ec-bbc7-0242ac190002 |
+-----------+------+------+-----------+--------------------------------------+
1 row in set (0.03 sec)
```

## 常见问题

同步不生效
 - `show variables like 'server_id` 查看server_id是否设置成功了
 - 查看从库能否访问主库
 - 查看从库信息 `Last_IO_Error`是否正常