---
layout: post
title:  "Redis hash"
date:   2021-03-01 21:18:54
categories: Redis
tags: Redis hash
excerpt: Redis hash
mathjax: true
---

* content
{:toc}

> Redis hash

#### 简介

redis hash是一个string类型的field和value的映射表，hash特别适合用于存储对象。redis中每个hash可以存储 2^32-1 (40亿)键值对。可以看成具有key和value的map容器，该类型非常适合存储键值对信息。

#### hash 命令

1. 赋值语法

    1.1 hset key field value 

    eg: hset h1 name zhangsan

    1.2 hmset key field value [field1 value2] 对一个值设置多个属性

    eg: hmset users:1 id 1 name songkai age 32

2. 取值语法

    2.1 hget key field 获取一个key指定的属性

    eg: hget h1 name 

    2.2 hmget key field [field2 field3]: 获取hash表中给定字段的值

    2.3 hmgetall key: 获取hash表中所有的字段和值

    2.4 hkeys key 获取所有哈希表的字段

    2.5 hlen key 获取哈希表中的字段数量

3. 删除

    3.1 hdel key field [field1 field2] 删除一个或者多个哈希表字段

    3.2 del key 删除整个表

4. 其他

    4.1 hsetnx key field value
    只有在字段field不存在时，设置哈希表字段的值

    4.2 hincrby key field increment
    为哈希表key中的指定字段的整数数值加上增量increment

    4.3 hincrbyfloat key field increment
    为哈希表key中的指定字段的浮点数数值加上增量increment

    4.4 hexists key field 查看哈希表key中，指定字段是否存在

#### hash 应用场景

1. 常用来存储一个对象

2. 为什么不用string？ 

    2.1 key 为string，把其他信息封装成一个对象以序列化的方式存储，这种方式的缺点在于增加了序列化和反序列化的开销，并且在需要修改其中一项信息时，需要把整个对象取回，修改操作需要进行并发保护，引入CAS等复杂问题。

    2.2 用户有多少成员就存储多少个key-value对，用用户ID+对应属性值的名称作为唯一标识来取得对应属性的值，虽然省掉序列化的开销和并发问题，但是用户ID为重复存储，如果大量存储这样的数据，内存浪费是很多的。


#### 使用java客户端

1. Jedis

    ```
    Jedis jedis = new Jedis("192.168.46.130", 6379);

    jedis.auth("passw0rd");

    System.out.println(jedis.ping());
    ```

    
    ```
    #测试string类型
    # 获得数据库的连接
    Jedis jedis = new Jedis("192.168.46.130", 6379);

    jedis.auth("passw0rd");

    jedis.set("strName", "字符串名称");
    String strName = jedis.get("strName");

    System.out.println(strName);
    # 关闭数据库连接
    jedis.close();
    ```

    ```
    /*
     * redis作用：为了减轻数据库(mysql)的访问压力，做个缓存
     * 需求: 判断某KEY是否存在，如果存在，就从redis中查询
     *       如果不存在，就查询数据库，且要将查询出的数据存入redis
     *
    */

    public void test() {
        Jedis jedis = new Jedis("192.168.46.130", 6379);
        jedis.auth("passw0rd");

        String key = "applicationName";
        if (jedis.exists(key)) {
            String value = jedis.get(key);
            System.out.println("redis缓存中查询：", value);
        } else {
            //假设从数据库中查询
            String result = "微信开发会议达人"；
            jedis.set(key, result);
            System.out.println("Mysql 数据库中查询得到" + result);
        }
    }
    ```

#### jedis工具类以简单示例

```
# jedis 连接池优化
public void test2() {
    //1. 设置jedis连接池配置
    JedisPoolConfig config = new JedisPoolConfig();
    //设置最大连接数
    config.setMaxTotal(100);
    //空闲时连接数
    config.setMaxIdle(10);

    //2. 设置连接池
    JedisPool pool = new JedisPool(config, "192.168.20.31", 6379);
    // 在池中得到redis
    Jedis jedis = pool.getResource();

    //3. 关闭连接，返回连接池
    jedis.close();
}
```

```
public class RedisPoolUtil {

    private static JedisPool pool;

    static{
        JedisPoolConfig poolConfig = new JedisPoolConFig();
        poolConfig.setMaxTotal(5);
        poolConfig.setMaxIdle(1);//设置最大空闲数
        //...
        pool = new JedisPool(config, "192.168.20.31", 6379);
        // 在池中得到redis
        Jedis jedis = pool.getResource();
    }

    public static Jedis getJedis() {
        Jedis jedis = pool.getResource();
        jedis.auth("passw0rd");
        return jedis;
    }

    public static void close() {
        jedis.close();
    }
}
```

```
# jedis hash 使用

Jedis jedis = RedisPoolUtil.getJedis();
String key = "users";

if (jedis.exists(key)) {
    Map<String, String> map = jedis.hgetAll(key);
    System.out.println("--redis查询的结果：");
    System.out.println(map.get("id") + "\t" + map.get("name") + "\t" + map.get("age") + "\t" + map.get("remark"));
} else {
    jedis.set(key, "id", "1");
    jedis.set(key, "name", "宋凯");
    jedis.set(key, "age", "22");
    jedis.set(key, "remark", "这是为男同学");
}
RedisPoolUtil.close();
```




