---
title: 夯实Linux基础 - 用户和组管理
date: 2020-06-11 18:34:52
tags:
    - linux
---


## 用户管理命令

添加用户

```bash
useradd [options] USERNAME
```

常用选项：

​	-c 基本的注释信息
​	-d 指定创建的家目录
​	-m 创建时自动创建目录，通常配合`-k`选项使用
​	-k 会自动把`/etc/skel`中的基本`bash`配置文件拷贝到当前创建用户的家目录
​	-u 指定用户ID
​	-g 指定的基本组ID号（组必须存在）
​	-G 指定的附加组，格式如 `1,2` 或组名 `user,game
​	-r 创建系统用户
​	-s 指定运行的SHELL

> 使用`-r`时系统会创建系统用户，系统用户的ID标识，如果未指定，是在`100-999`并且未被使用范围内，具体范围可参考`/etc/login.defs`中的`SYS_UID_MIN`和`SYS_UID_MAX`



修改用户

```bash
usermod [options] LOGIN
```

常用选项：

​	-c 基本的注释信息
​	-d 指定创建的家目录
​	-m 移动已存在的用户家目录到指定位置
​	-u 指定用户ID
​	-g 指定的基本组ID号（组必须存在）
​	-G 指定的附加组，格式如 `1,2` 或组名 `user,game` ， `-a`选项表示在原附加组上进行追加
​	-s 指定运行的SHELL
​	-L 锁定该用户
​	-U 解锁该用户



删除用户

```bash
userdel [options] LOGIN
```

常用选项：

​	-r 删除用户的家目录

<!--more-->

查看用户ID信息

```bash
id [OPTION]... [USER]
```

常用选项：

​	-u 查看用户ID
​	-g 查看用户基本组
​	-G 查看用户附加组
​	-n 组合 `ugG` 选项，给出当前用户名，或组名，及附加组名



修改用户`shell`

```bash
chsh [options] [LOGIN]
```

常用选项：

​	-s 设置用户`shell`



查看用户的帐号信息 包括执行shell，home，以及有没有用户的cron

```bash
finger [LOGIN]
```



修改密码

```bash
passwd [options] [LOGIN]
```

常用选项：

​	-l 锁定用户
​	-u 解锁用户
​	-e 设置过期时间



管理密码时效

```bash
chage [options] LOGIN
```

常用选项：	

​	-d 设置最后一次密码更改的日期
​	-E 设置密码过期时间
​	-I 过期后设置密码无效
​	-m 设置密码最小修改天数
​	-M 设置密码最大修改天数



## 组命令管理

添加用户组

```bash
groupadd [options] GROUP
```

常用选项：

​	-g 指定gid
​	-p 为组设置一个密码
​	-r 设置为系统组



修改组

```bash
groupmod [options] GROUP
```

常用选项：

​	-g 指定gid
​	-n 修改新的组名称
​	-p 设置密码



删除组

```bash
groupdel [options] GROUP
```

常用选项：

​	-f 强制删除



组管理工具

```bash
gpasswd [option] GROUP
```

常用选项：

​	-a 添加一个用户到指定组
​	-d 从组中移除用户
​	-r 删除组密码
​	-R 限制对GROUP的访问
​	-M 为当前组设置用户多个用户



临时登录切换用户的基本组（需要密码，所以才有gpasswd这个为组设置密码）

```bash
newgrp
```





## 涉及的文件

```text
/etc/passwd                     ->1
/etc/shadow
/etc/group
/etc/gshadow
/etc/default/useradd
/etc/skel
/etc/shells
/etc/login.defs
```



### 基本说明

当使用命令`useradd`添加一个用户时，如果未指定组，系统会自动生成一个主组（基本组）并且在`/etc/passwd`下面新生成一条记录，然后会在`/etc/group`写入用户对应的主组记录

当使用`passwd`修改密码时，系统会修改`/etc/shadow`来设置用户密码

在使用`useradd`添加一个用户时，默认如果不指定`-d -m`等选项时，系统会默认有一个选项值 ，这个选项值保存在`/etc/default/useradd`中

当`useradd`饮食`-m -k`选项时，系统会自动拷贝`/etc/skel/*`中的所有相关配置到用户家目录中去

当使用`-s`选项设定`shell`时，我们需要知道系统支持的`shell`类别，则在文件`/etc/shells`中查看

使用`groupadd`表示添加一个组，如果使用了`-p`选项，则系统会修改`/etc/gshadow`文件保存对应的组密码

`/etc/login.defs`包含了登录的配置控制定义。



### 文件格式

`/etc/passwd` 用户信息保存文件

```text
1:2:3:4:5:6:7
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
```

1. 用户名

2. 密码占位

3. UID

4. GID

5. 注释信息(对应useradd -c)

6. 主目录路径

7. 执行shell

   

`/etc/shadow` 用户密码保存文件

```text
1:2:3:4:5:6:7:8:9
root:$6$XkKHd3gi$mB7EJmIheafUbdS9gN0R2pfMvwBJmyvNV.BV4FGvz47wQIbeVZP1NdpNoGagLRstXUvPAXQxxMLb1uKc6NqQt/:18378:0:99999:7:::
daemon:*:17647:0:99999:7:::
```

1. 用户名
2. 密码，!或者*的话，表示此帐户不可登录
3. 密码最近一次修改时间，注意：这个是从1970年1月1日算起的总的天数
4. 最少多少天后才能改密码的天数(默认为0)
5. 最多多少天后一定要修改密码的天数(默认为99999)
6. 过期前多少天时间会被警告
7. 过期后多少天内账号变为inactive状态，可登陆，但不能操作
8. 多少天后账号会过期，无法登陆
9. 保留参数



`/etc/group`用户组管理

```text
1:2:3:4
root:x:0:
daemon:x:1:
bin:x:2:
sys:x:3:
adm:x:4:
tty:x:5:
disk:x:6:
```

1. 组名称
2. 组密码
3. 组ID
4. 当前组对应的其它用户（对于用户来说是附加组）,主组用户不会写入此字段，多个值使用`,`隔开

> `Linux`中一个用户可以有多个组，有主组(基本组)和附加组的区分



`/etc/gshadow`组密码文件

```text
1:2:3:4
root:*::
daemon:*::
```

1. 组名称
2. 组密码
3. 组管理员
4. 组成员和`/etc/group`中的第4列相同



`/etc/skel` 默认的`shell`配置文件目录

```text
drwxr-xr-x 2 root root 4096 Mar 11 21:05 .
drwxr-xr-x 1 root root 4096 Jun 12 06:10 ..
-rw-r--r-- 1 root root  220 Apr  4  2018 .bash_logout
-rw-r--r-- 1 root root 3771 Apr  4  2018 .bashrc
-rw-r--r-- 1 root root  807 Apr  4  2018 .profile
```



`/etc/shells`默认系统支持的shell

```text
/bin/bash
/bin/csh
/bin/ksh
/bin/sh
/bin/tcsh
/bin/zsh
```



`/etc/login.defs`用户登录的相关配置文件

```text
PASS_MAX_DAYS   99999　　#密码最大有效期
PASS_MIN_DAYS   0 　　  #两次修改密码的最小间隔时间
PASS_MIN_LEN    5     #密码最小长度，对于root无效
PASS_WARN_AGE   7      #密码过期前多少天开始提示

# Min/max values for automatic uid selection in useradd
#创建用户时不指定UID的话自动UID的范围
UID_MIN                   500 #用户ID的最小值
UID_MAX                 60000   #用户ID的最大值

# Min/max values for automatic gid selection in groupadd
#自动组ID的范围
GID_MIN                   500  #组ID的最小值
GID_MAX                 60000  #组ID的最大值
```


