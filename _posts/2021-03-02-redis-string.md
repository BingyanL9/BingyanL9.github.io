---
layout: post
title:  "Redis string"
date:   2021-03-01 21:18:54
categories: Redis
tags: Redis string
excerpt: Redis string
mathjax: true
---

* content
{:toc}

> Redis string

#### Redis Key 的命名建议

1. key不要太长，尽量不要超过1024字节，这不仅消耗内存，而且会降低查找效率；

2. key不要太短，太短的话，key的可读性会降低。

3. 在一个项目中，key最好使用统一的命名模式(解决数据之间关系问题)，例如： user:id:properties

4. 区分大小写

#### String 

1. string 是redis最基本的类型，一个key对应一个value。

2. string类型是二进制安全的。意思是redis的string可以包含任何数据。比如jpg图片或者序列化的对象。

3. string类型是redis最基本的数据类型，一个键可以存储512MB（不建议超过1024字节）

二进制安全是指：在传输数据时，保证二进制数据的信息安全，也就是不被篡改、破译等，如果被攻击，能够及时监测出来。
特点：
    1. 编码、解码发生在客户端完成，执行效率高
    2. 不需要频繁的编解码，不会出现乱码

####　命令

1. 赋值语法

    1.1 set key_name value

    redis set命令用于设置给定key的值。如果key已经存储值，set就覆写旧值，且无视类型

    1.2 SETNX key value
    只有在key不存在时设置key的值。 setnx（set if not exists）命令在指定的key不存在时，为key设置指定值。

    1.3 MSET key value [key value ...]
    同时设置一个或多个key-value对

2. 取值语法

    2.1 get key_name

    redis get命令用于设置给定key的值。如果key不存在，返回nil。如果key存储的不是字符串类型，返回一个错误。

    2.2 getrange key start end 
    eg: getrange key 0 3
    用于获取存储指定key中字符串的子字符串。字符串的截取范围由start和end两个偏移量决定（包括start和end在内）

    2.3 getbit key offset
    对key所存储的字符串值，获取指定偏移量上的位

    2.4 mget key1 [key2 key3....]
    获取所有（一个或多个）给定key的值

    2.5 getset key_name value
    getset 命令用于设置指定key的值，并返回key的旧值，当key不存在，返回nil

    2.6 strlen key
    返回key所存储的字符串值的长度 

3. 删除语法

    3.1 del key_name 删除key，如果存在，返回数字类型 

4. 自增/自减

    4.1 INCR key_name 
    incr 命令将key中存储的数字值增1.如果key不存在，那么key的值会先被初始化为0，然后执行 INCR 操作

    4.2 INCRBY Key_name 增量值
    将key中存储的数字加上指定的增量值

    4.3 DECR key_name/DECRBY key_name 减值

    4.4 append key_name value
    append 命令用于为指定的key追加至末尾，如果不存在，为其赋值

    


