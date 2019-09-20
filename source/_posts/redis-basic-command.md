---
title: Redis基础数据类型和常用操作命令
date: 2019-09-20 06:59:14
tags:
---
## 简述
经常用的都是通过代码包装好的`redis`调用程式，时间有些常用命令又模糊了，故拿出来记录下。

## 基础数据类型

### String
#### Add|Set
- `SET key value [EX seconds|PX milliseconds] [NX|XX]` NX:相当于SETNX值不存在时增加，XX值存在覆盖
- `MSET key value key value ...` 一次设置多个值
- `SETNX key value` 原子性操作，当key不存在设置成功返回1，否则返回0
- `STRLEN key` 返回key的长度，不存在返回0，非string类型返回错误
- `APPEND key value` 在key的末尾追加字符串，返回字符串长度，原来的key不存在，则会增加（相当于SET）
- `SETRANGE key offset value` 替换字符串，从offset下标开始，替换strlen(value)位字符串

#### Get
- `EXISTS key ...` 判断key是否存在，返回存在key的个数，单个key不存在返回0
- `GET key` 获取key值
- `MGET key ...` 获取多个key的值

#### Delete
- `DEL key ...` 删除指定key，成功返回删除个数

> `string`类型在redis中有三种存储方式
> - int 64位有符号位整数
> - embstr 长度小于44个字节的字符串
> - raw 长度大于44个字节的字符串
> 其中raw效率相对最差，可以通过 `OBJECT ENCODING key`的命令来查看string的存储类型

### List

#### Add|Set
- `LPUSH key value ...` 左侧写入返回总列表个数
- `RPUSH key value ...` 右侧写入返回总列表个数
- `LINSERT key [BEFORE|AFTER] item value` 从左侧开始，指定元素(item)前或后插入新元素，返回总列表个数
- `LSET key index` 覆盖或增加指定索引的值，返回OK，超过最大list范围则会报错

#### Get
- `LRANGE key start stop` 获取指定范围的list, `lrange key 0 -1` 标识获取整个list
- `LINDEX key index` 获取指定索引处的元素，不存在返回nil

#### Delete
- `LPOP key` 左侧弹出，返回总列表个数，当list不存在时返回nil
- `RPOP key` 右侧弹出，返回总列表个数，当list不存在时返回nil
- `LTRIM key start stop` 删除指定区间外的元素，返回OK
- `BLPOP key` LPOP阻塞式方法
- `BRPOP key` RPOP阻塞式方法



### Hash
类似于`Go`中的`map`或`PHP`中的键值数组，用于存储字段的映射关系

#### Add|Set
- `HMSET key field value ...` 成功返回 `OK`
- `HSET key field value ...` 增加field时返回新增field值的个数，覆盖时返回0(覆盖成功也是0)
- `HSETNX key field value` 当field不存在时增加成功返回1，如果field存则返回0,不可同时进行多个field操作

#### Get
- `HKEYS key` 返回当前hash中所有的field
- `HEXISTS key field` 判断当前key中是否存在field，存在返回1，否则返回0
- `HGETALL key` 返回当前Key的所有值，当HASH巨大的时候不适合使用此函数
- `HMGET key field ...` 返回当前key指定的field值
- `HSCAN key cursor [MATCH field] [COUNT number]` 通过指针移动来获取MATCH匹配的值，COUNT代表的是返回的元素个数，但即使设置了count也并不一定代表就返回count个元素，因此不可作为分页式数据

#### Delete
- `HDEL key field ...` 删除一个或多个field，当key全部删除后，key会被自动清除

### Set

#### Add|Set
- `SADD key member ...` 添加集合元素，返回添加的的元素数量

#### Get
- `SISMEMBER key member` 判断是否存在于集合，0表示不存在1存在
- `SCARD key` 获取集合元素数量
- `SMEMBERS key` 返回集合所有元素
- `SSCAN key cursor [MATCH member] [COUNT number]` 通过指针移动来获取MATCH匹配的值，COUNT代表的是返回的元素个数，但即使设置了count也并不一定代表就返回count个元素，因此不可作为分页式数据
- `SRANDMEMBER key count` 随机返回指定count元素
- `SUNION key key ...` 集合并集，返回合并后的值
- `SUNIONSTORE storeKey key key ...` 并集，并存储至storeKey中，索引相同则覆盖，返回合并个数
- `SINTER` 集合交集，返回合交集值
- `SINTERSTORE storeKey key key ...` 交集，并存储至storeKey中
- `SDIFF` 集合差集，返回合差集值
- `SDIFFSTORE diffKey key key ...` 差集，并存储至diffKey中

#### Delete
- `DEL key` 删除集合