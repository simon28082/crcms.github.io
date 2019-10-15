---
title: Mysql8下主从复制、主主复制、多源复制
date: 2019-10-15 07:27:31
tags:
  - mysql
  - replication
  - 主从复制
  - 主主复制
  - 多源复制
---

## 简述
复制的原理如下：在主库上执行的所有`DDL`和`DML`语句都会被记录到二进制日志中，也就是`binlog`。
这些日志由连接到它的从库提取。它们只是被复制到从库，并被保存为中继日志。这个过程由`IO线程`的线程负责。还有一个`SQL线程`，它按顺序执行中继日志中的语句。

## 主要步骤
不管是主主，还是多源，其基础配置都和主从差不多，所以就列出主从的基本步骤：

1. master 开启`binlog`
2. 创建复制用户
3. 设置 master 以及 slave 惟一的`server_id`
4. 在 slave 上执行`change master to`
5. 执行`start slave`

## 主从复制

### 增加binlog配置

如果不想要自定义配置，先看下系统是否开启了binlog即可
```bash
show variables like '%log_bin%'
```
![binlog](/images/mysql8-replication/log_bin.png)

增加或修改`binlog`配置，一同增加`server_id`
```
[mysqld]
log-bin=/var/lib/mysql/bin_logs/master
expire-logs-days=7
max-binlog-size=1GB

server-id=1
```
> 其它更多`binlog`配置请参考官方文档

查看`binlog`其它配置
``` bash
SHOW VARIABLES LIKE '%binlog%';
```
![binlog](/images/mysql8-replication/bin_log.png)

查看所有`binlog`及大小
```bash
show master logs(或者 show binary logs)
```
![binlog](/images/mysql8-replication/logs.png)

查看当前最新`binlog`位置
```bash
show master status;
```
![binlog](/images/mysql8-replication/master_status.png)

查看`binlog`所有位置及其它信息
```bash
show binlog events;
```
![binlog](/images/mysql8-replication/binlog_events.png)

移至下一个日志（开启新日志）
```bash
flush logs;
```
![binlog](/images/mysql8-replication/flush_logs.png)

### 创建复制用户
```bash
create user 'binlog_user'@'%' identified with mysql_native_password by 'binlog_password';
grant replication slave on *.* to 'binlog_user'@'%';
flush privileges;
```
> 需要的权限以及连接的服务器请自行修改

### 修改slave配置，增加`server_id`
```
[mysqld]
server_id=100
```

### 在slave中执行`change master to`
```bash
change master to master_host='172.22.0.2',master_user='binlog_user',master_password='binlog_password',master_log_file='master.000013',master_log_pos=155
```

> 如果配置增加未修改的话，请重启master和slave

### 开启复制
```bash
start slave
```

## 主从示例
待续。。。。

## 主主复制
待续。。。。

## 多源复制
待续。。。。