---
title: 夯实Linux基础 - 入门
date: 2020-06-10 01:34:33
tags: 
    - linux
---

## 文件目录结构

```text
├── bin                                     可执行文件，用户命令，系统启动相关的
├── boot                                    系统启动文件
├── dev                                     设备文件，设备数据的访问入口(块设备,字符设备)
├── etc                                     配置文件
├── home
├── lib                                     库文件
├── lib64                                   库文件64位
├── media                                   挂载点目录，通常用于挂载移动设备
├── mnt                                     额外的临时文件系统，如第二块硬盘
├── opt                                     可选目录，早期通常用于第三方安装程序的安装目录，现在基本安装在/usr/local中
├── proc                                    伪文件系统，内核的映射文件
├── root                                    root用户目录
├── run                                     系统运行目录
├── sbin                                    可执行文件，系统命令，系统启动相关的
├── sys                                     伪文件系统 和硬件设备相关的属性映射文件
├── tmp                                     系统临时文件
├── usr                                     Unix操作系统软件资源所放置的目录(Unix Software Resource)
│   ├── bin                                 主要是用于系统启动后提供的一系列用户命令
│   ├── local                               第三方软件
│   │   ├── bin
│   │   ├── lib
│   │   ├── share
│   │   └── sbin
│   ├── lib                                 不包含于系统核心库的库
│   └── sbin                                主要是用于系统启动后提供的一系列系统命令
└── var                                     可变目录
```

<!--more-->

## 开始第一个命令`type`

`type`主要是用于区别命令类型，格式如下：

```bash
type <cmd>
```

如下示例：

```bash
type cd
# output: cd is a shell builtin -> 系统内置命令

type chmod
# output: chmod is /bin/chmod -> 外部命令路径

type ls
# output: ls is aliased to `ls --color=auto' -> 外部命令描述
```



## 命令如何找到的

环境变量PATH，在terminal中我们输入`echo $PATH`可以看到，命令的查找存储路径，如下示例：

```text
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

`PATH`中多个路径使用`:`分隔

我们所有执行的命令查找都是系统通过`PATH`环境变量来查找，系统会从`PATH`路径左侧目录开始查找，找到第一个即会停止。

系统对于已使用的命令，并不会每次都去重新查找真实路径，而是会放到内存缓存中来提高查找效率。

通过`hash`命令可以查看缓存区命中的命令集合

```bash
hash
```

输出如下示例：

```txt
hits	command
   1	/bin/cat
   3	/usr/sbin/useradd
   6	/bin/ls
```



## 命令帮助

通过`type <cmd>`可以判断是否是系统内置命令，系统内置命令可以使用，`help`命令来查找帮助使用说明。

```bash
help cd
```

非系统内置命令，基本上使用`--help`选项，来查找帮助。

```bash
ls --help
```

远程帮助文档，可以使用如下`info`命令，如：

```bash
info ls
```

另外，在Linux中大部分文档都存储在`/usr/share/doc`目录中。



## 超级`man`

`man`手册(`manual`)的简写，`Linux`几乎所有的命令都可以通过`man`来查找说明和使用。

```bash
man cd
```

![image-20200609221059943](/Users/simon/Documents/公司/Linux/Basic/Linux基础一.assets/image-20200609221059943.png)



上图可以看到类似于`BUILTIN(1)`这种字样，这是表示`cd`是内置命令，并且`(1)`表示当前在`man`的第几章节中。

`man`的几个主要章节为：

- 1 用户命令(/bin:/usr/bin:/usr/local/bin)

- 2 系统调用

- 3 库调用

- 4 特殊文件，如设备文件

- 5 文件格式，解释配置文件的语法 如man 5 passwd

- 6 游戏

- 7 杂项

- 8 管理命令 （/sbin:/usr/sbin:/usr/local/sbin）

可以使用另一个命令`whatis`来查看命令在哪个章节中，如：

```bash
whatis chmod
```

![image-20200609221655831](/Users/simon/Documents/公司/Linux/Basic/Linux基础一.assets/image-20200609221655831.png)

可以看到`chmod`在第一章节中



`man`的快捷键

向后翻一屏：`space`

向前翻一屏：`b`

向后翻一行：`enter`

向前翻一行：`k`

`/keyword` 向后搜索

`?keyword` 向前搜索

`n`: 下一个

`N`: 前一个


