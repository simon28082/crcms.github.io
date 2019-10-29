---
title: Mysql8下GTID复制
date: 2019-10-29 14:39:31
tags:
  - mysql
  - replication
  - GTID复制
---

## 简述
之前介绍了通过`binlog Pos`来进行主从复制，基于复制点(pos)来进行复制，在迁移或因故障未能及时同步
等问题下必须重新定位复制点，非常麻烦，特别是对于多台mysql而言。而GTID就可以很好的解决这个问题。
GTID自动检测二进制日志的位置。

## 什么是GTID?
全局事务标识符（Global Transaction Identifier, GTID ）是在程序中创建的唯一标识符，并与主库上提交的每个事务相关联。
此标识符是唯一的，不仅在其主库上，在给定的复制设置中的所有数据库上，它都是唯一的。
所有事务和所有GTID之间都是一对一的映射关系。
GTID用一对坐标表示，用冒号（：）分隔：

```
GTID = source_id:transaction_id
```

source_id是主库的标识。通常，服务器的server_uuid选项就代表此标识。transaction_id 是一个序列号，由在该服务器上提交事务的顺序决定。

## 主要步骤
不管是主主，还是多源，其基础配置都和主从差不多，所以就列出主从的基本步骤：

1. master 开启`binlog`
2. 创建复制用户
3. 设置 master 以及 slave 惟一的`server_id`,`gtid_mode`,`enforce_gtid_consistency`
4. 在 slave 上执行`change master to MASTER_AUTO_POSITION=1`
5. 执行`start slave`

## 完整示例
开启3台mysql，分别命令为master,slave,slave2，初始配置如下：
其中slave2我们使用延时同步，主要是为了更好的灾备。

master:
```
[mysqld]
skip-host-cache
skip-name-resolve

[mysql]
```

slave:
```
[mysqld]
skip-host-cache
skip-name-resolve

[mysql]

```

slave2:
```
[mysqld]
skip-host-cache
skip-name-resolve

[mysql]

```
运行Mysql
```bash
docker run -d -it -v /tmp/mysql/master/conf:/etc/mysql/conf.d -v /tmp/mysql/slave/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=root --name master mysql
docker run -d -it -v /tmp/mysql/slave/conf:/etc/mysql/conf.d -v /tmp/mysql/master/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=root --name slave mysql
docker run -d -it -v /tmp/mysql/slave2/conf:/etc/mysql/conf.d -v /tmp/mysql/slave2/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=root --name slave2 mysql
# 创建桥接网络
docker network create mysql
docker network connect --alias master mysql master
docker network connect --alias slave mysql slave
docker network connect --alias slave2 mysql slave2

```
docker ps查看保证mysql都已经启动，然后增加binlog和gtid的配置

master:
```
log_bin=/var/lib/mysql/bin_logs/master
server_id=100
gtid_mode=on
enforce_gtid_consistency=on
```
slave:
```
log_bin=/var/lib/mysql/bin_logs/slave
server_id=101
gtid_mode=on
enforce_gtid_consistency=on
```
slave2:
```
log_bin=/var/lib/mysql/bin_logs/slave
server_id=103
gtid_mode=on
enforce_gtid_consistency=on
```

在master上增加复制帐户
```
create user 'binlog_user'@'%' identified with mysql_native_password by 'binlog_password';
grant replication slave on *.* to 'binlog_user'@'%';
flush privileges;
```

重启master和slave
```bash
docker restart master
docker restart slave
docker restart slave2
```

在slave中增加`change master to` 并 `start slave`
```bash
change master to master_host='master',master_user='binlog_user',master_password='binlog_password',master_port=3306,master_auto_position=1;
start slave;
```

在slave中增加`change master to` 并 `start slave`并开启延时
```bash
change master to master_host='master',master_user='binlog_user',master_password='binlog_password',master_port=3306,master_auto_position=1,master_delay=10;
start slave;
```
> 这里注意，master_delay=10表示开启了10秒延时

## 查看slave和master状态
```mysql
show master status\G;
```
![binlog](/images/mysql8-gtic-replication/master_status.png)

```mysql
show slave status\G;
```
可以看到slave的Slave_IO_State: Waiting for master to send event
![binlog](/images/mysql8-gtic-replication/slave_status.png)

## 测试
在主库中创建数据库并写入数据
```mysql
create database tv default charset utf8mb4 collate utf8mb4_unicode_ci;
create table t1 (
  id int(10) unsigned primary key auto_increment,
  content text default null
);
#写入数据
use tv;
insert into t1(content) values('1'),('2'),('3'),('4'); 
```

## 结果
登录slave和slave2分别查看可以看到，数据已经同步，其中slave2是延时同步，也已经生效。
![binlog](/images/mysql8-gtic-replication/slave1_demo.png)
![binlog](/images/mysql8-gtic-replication/slave1_demo.png)

## 最后
master正常写入数据
重启slave和slave2随后也是可以正常同步的
> 如果希望从库重启后不进行自动同步，需要在配置中增加`skip_slave_start`
> 还要注意的是，这里的示例是全部新库，如果原来是使用pos位置同步这里并不完全适用
> 需要先设置master为read only，等待所有从库全部同步完成才可以进行切换