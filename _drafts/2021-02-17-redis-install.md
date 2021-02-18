---
layout: post
title:  "Redis 安装"
date:   2021-02-18 21:18:54
categories: Redis
tags: Redis install
excerpt: Redis install
mathjax: true
---

* content
{:toc}

> Redis 简介和安装

## Redis

Redis 是开源的免费的，遵守BSD协议，是一个高性能（NOSQL）的key-value数据库。 Redes 是一个开源的使用ANSI C语言编写、支持网络，基于内存，也可持久化的日志型数据库，并提供多种语言的API。

NOSQL： 非关系型数据库，数据和数据之间没有关联关系。为了解决大规模数据集合多重数据种类带来的挑战，尤其是大数据应用的难题。

NOSQL类型： 键值型数据库（redis, oracle BDB）, 列存储数据库（分布式存储的海量数据，HBase），文档型数据库（MongoDb），图形数据库（Neo4J）

NOSQL类型使用场景：

    1. 数据模型比较简单 

    2. 需要灵活更强的IT系统 

    3. 对数据库性能要求高 

    4. 不需要高度的数据一致性 

    5. 对于给定的key，比较容易映射复杂值的环境

Redis 特点：

    1. 支持数据持久化，可以将内存中的数据保存在磁盘中，重启的时候可以再次加载进行使用。

    2. 不仅仅支持简单的key-value类型的数据，还提供list，set，zset，hash等数据结构的存储

    3. 支持数据备份和集群（最多支持15个库）等高可用功能。

    4. 性能极高

    5. 所有操作都是原子性。但操作是原子性的，多操作也支持事务。

    6. 单个键存入512M大小

    7. 单线程（最新不是单线程）

    8. 可以作为数据库、二级缓存和消息中间件

    9. 没有锁

缺点： Redis 对内存占用过高

总结： Redis 是一个简单的、高效的、分布式的。 可以作为基于内存的缓存工具。

## 安装 - linux

```
## 安装gcc
yum -y install gcc automake autoconf libtool make

## 如果出现/var/run/yum.pid已经被占用，需要执行下面命令
rm -f /var/run/yum.pid

wget http://download.redis.io/releases/redis-4.0.1.tar.gz
tar xzf redis-4.0.1.tar.gz
cd redis-4.0.1

# 执行以下其中一个
make 
make MALLOC=lib

# 安装编译后的文件
make PREFIX=/usr/local/redis install

# 查看
cd /usr/local/redis
```

## 启动Redis

1. 启动服务端

    ```
    cd /usr/local/redis/bin
    ./redis-server
    ```

12. 启动客户端

    ```
    cd /usr/local/redis/bin
    ./redis-cli
    ```

## Redis配置

Redis 核心配置文件在安装目录下，名字为redis.conf

- 进入下载完之后解压的安装包 

    ```
    cd /redis-4.0.1
    ll // 就可以看到redis.conf文件了
    ```

- 将以上配置文件复制到安装目下 ```/usr/local/redis```

    ```
    cp redis.conf /usr/local/redis/
    ```

-  查看配置文件

    ```
    cd /usr/local/redis
    less -mN redis.conf
    # 搜索 /[search text]
    ```

    如果需要配置生效，起服务
    
    ```
    ./redis-server /path/to/redis.conf
    ```


- 解读配置文件

    ```

    daemonize yes # 值yes/no 是否为守护线程，默认不是，需要改成yes。 
    pidfile /var/run/redis.pid # 表示当redis以守护线程运行时，redis默认会把pid写入到/var/run/redis.pid文件
    port 6379 # 端口号
    bind 127.0.0.1 # 绑定本地主机 注释它
    timeout 300 # 当客户端闲置多长时间后关闭连接，如果指定为0，表示关闭该功能
    loglevel verbose # 日志几乎级别，有四种 debug verbose notice warning。 默认为verbose
    logfile stdout # 日志记录方式，默认为标准输出。如果redis为守护线程，而这里又配置了标准输出作为日志记录方式，日志将发送给 /dev/null
    databases 16 # 设置数据库的数量，默认为16。可以使用 SELECT <dbid> 命令在连接上指定数据库id

    save <seconds> <changes> # 多长时间内，有多少次操作，就将数据同步到数据文件，可以多个条件配合，默认提供一下几种
    save 900 1
    save 300 10
    save 60 10000

    rdbcompression yes # 指定存储至数据库时是否是压缩数据，默认是。Redis采用LZF压缩，如果为了节约CPU资源，可以关闭该选项。但会导致数据库文件变得巨大
    dbfilename dump.rdb # 指定本地数据库文件名。默认值为 dump.db 
    dir ./ #指定本地数据库的存放目录 ./ 表示当前目录
    slaveof <masterip> <mastrport> # 设置本地为slave机器，masterip masterport 指定master的ip和端口
    masterauth <master-password> # 当master设置了安全认证时，通过这个参数传入密码
    
    requirepass foobared  #设置Redis连接密码，如果配置密码，客户端在连接Redis时需要通过 AUTH <password> 命令提供密码，默认关闭

    maxclients 128 # 这是同一时间最大客户端连接数量，默认无限制，redis可以同时打开的客户端连接数为redis进程可以打开的最大文件描述符数。如果设置为0，表示不做限制。当客户端连接数达到上限时，redis会关闭新的连接并给客户端返回 max number of clients reached 错误信息

    maxmemory <bytes> # redis最大内存限制。默认256M。建议256-512M 不超过1G。达到最大后，redis会先尝试清楚已过期或即将过期的key，做完这个操作之后，还是达到最大设置，将无法执行写入操作，但仍然可以读取操作。 新的vm 会把key放在内存，value放在swap区
    #为了处理内存不足导致性能下降的问题 redis 有两种解决方案：
    # 1. 为数据设置超时时间 
    # 2. 采用LRU算法动态将不用的数据清除：
          volatile-lru: 设置超时间的数据中，删除最不常用的数据
          allkeys-lru: 查看所有的key中最近最不常用的数据进行删除，广泛应用
          volatile-lfu：查询所有配置超时时间的数据，删除使用频率最少的数据
          allkeys-lfu: 查看所有的key中使用频率最少的数据进行删除
          volatile-random: 在设置已超时的数据中随机删除
          allkeys-random: 查询所有key 之后随机删除
          volatile-ttl：查询所有设置超时时间的数据，之后排序，将马上将要过期的数据进行删除
          Noeviction: 如果设置该属性，将不会进行删除操作，如果内存溢出，报错

    appendonly no # 指定是否在每次更新操作后进行日志记录，redis在默认情况下异步的把数据写入磁盘，如果不开启，将会在断电后导致一部分数据丢失。因为redis本身同步数据是按 save <seconds> <changes>策略来同步数据的。默认为no
    appendfilename appendonly.aof # 指定更新日志文件名，默认为appendonly.aof
    appendfsysnc everysec # 指定更新日志的条件 有三个可选值 no always everysec

    ```

## 自定义配置

```
daemonize yes 
#bind 127.0.0.1
requirepass 123456
```

1. 再次启动，会以守护启动，并且客户端连接需要密码

    ```
    ./bin/redis-server redis.conf
    ```

2. 启动客户端

    ```
    ./bin/redis-cli -a 123456 # ./bin/redis-cli -h host -p port -a password

    #加入key
    set gradeName java1802

    # 查看key
    get gradeName
    ```

3. 关闭redis

    ·1 断电 非正常，会丢数据

    ```
    #查询redis服务端的pid
    pid -ef | grep -i redis
    kill -9 <pid>
    ```

    ·2 在客户端中执行shutdown命令。关闭也会做持久化，不会丢失数据

    ```
    ./bin/redis-cli shutdown
    ```

