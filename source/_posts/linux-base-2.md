---
title: 夯实Linux基础 - 文件系统基础命令
date: 2020-06-10 01:34:52
tags:
    - linux
---


## 目录及文件操作命令

进入指定目录

```bash
cd [directory]
```

`cd` 默认进入指定目录

`cd ~`进入当前home目录

`cd -` 返回上一层目录



创建目录

```bash
mkdir [OPTION]... DIRECTORY... 
```

常用选项：

​	-m 创建的权限码

​	-p 递归创建

​	-v 显示创建详情



创建文件或创建目录

```bash
install [OPTION]... SOURCE... DIRECTORY

```

常用选项：

​	-d 创建目录

> `install` 命令复制文件，会自动增加执行权限



一个参数创建多个目录：

```bash
# 创建 /tmp/foo/f1 /tmp/foo/f2
mkdir -pv /tmp/foo/{f1,f2}

# 创建 /tmp/foo/z1/s1 /tmp/foo/z2
mkdir -pv /tmp/foo/{z1/s1,z2}
```



查看目录内容

```bash
ls [OPTION]... [FILE]...
```

常用选项：

​	-a 显示隐藏文件

​	-r 倒序排列

   -R 递归显示所有

​	-l 使用表格式的列表格式展示



移动或重命名文件

```bash
mv [OPTION]... SOURCE... DIRECTORY
```



创建一个空白文件或更改文件时间

```bash
touch [OPTION]... FILE...
```

常用选项：

​	-c 不创建文件

​	-m 仅修改修改时间

​    -a 修改访问时间

​	

查看文件属性

```bash
stat [OPTION]... FILE...
```



查看文件的内容或类型

```
file [OPTION...] [FILE...]
```

<!--more-->



## 文本操作命令

查看文件内容

```bash
cat [OPTION]... [FILE]...
```

常用选项：

​	-n 显示行号

​    -T 显示制表符号 ^I



查看文件头内容

```bash
head [OPTION]... [FILE]...
```

常用选项：

​	-n 显示指定行数



查看文件尾内容

```bash
tail [OPTION]... [FILE]...
```

常用选项：

​	-n 显示指定行数

​	-f 监听打开的文件



打开一个文件，分页显示

```bash
less OPTION... [FILE]...
```

操作的快捷键同`man`



相比于`less` `more`命令更常用

```bash
more OPTION... [FILE]...
```

操作的快捷键同`man`



分隔文本

```bash
cut OPTION... [FILE]...
```

如下示例：

```bash
cut -d : -f 1 < /etc/passwd
```

常用选项：

   -d 设置分隔符，默认为空格

​	-f 显示第n个字段，n是1开始，非0为初始下标



```bash
sort [OPTION]... [FILE]...
```

`sort`默认是以`ASCII`码作为排序标准的

如下示例：

```bash
sort -nf -t : -k 1 < /etc/passwd
```

常用选项：

​	-f 忽略大小写

​	-n 以数字形式进行排序

​	-u 去重（注意：相邻的上下行才算是重复，如果有间隔行，则不算重复）

​	-t 指定分隔符号

​	-k 按`-t`指定的分隔符号字段进行排序



去重，相邻的上下行才算是重复，如果有间隔行，则不算重复

```bash
uniq [OPTION]... [INPUT [OUTPUT]]
```

常用选项：

​	-c 显示重复次数



```bash
wc [OPTION]... [FILE]...
```

常用选项：

​	-l 统计行数



强大又好用的搜索命令

```bash
grep [OPTION]... PATTERN [FILE]...
```

常用选项：

​	-v 去除包含的匹配项