---
title: Pycharm搭建Docker环境运行Python
tags:
  - docker
  - pycharm
  - python
url: 61.html
id: 61
categories:
  - Python
date: 2018-05-30 16:38:16
---

部署环境：
-----

**platform**： `win10` **docker**： `docker for windows` ![](http://blog.crcms.cn/wp-content/uploads/2018/05/6-1024x291.png) 

设置步骤
----

### 1、允许本机2375端口连接

![docker http://localhost:2375](http://blog.crcms.cn/wp-content/uploads/2018/05/1.png)

### 2、测试连接状态

    settings->Build->Docker
    

![](http://blog.crcms.cn/wp-content/uploads/2018/05/2.png) 当出现如图Connection successful，则表示连接成功

### 3、选择镜像

    settings->Project->Interpreter
    

![](http://blog.crcms.cn/wp-content/uploads/2018/05/3.png) 选择Docker镜像，如果没有则点击`Add`，如图： ![](http://blog.crcms.cn/wp-content/uploads/2018/05/4.png)

### 4、测试

至此已完成搭建，新建test.py测试即可 ![](http://blog.crcms.cn/wp-content/uploads/2018/05/5.png)