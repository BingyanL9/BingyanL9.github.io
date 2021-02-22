---
layout: post
title:  "Redis 命令"
date:   2021-02-18 21:18:54
categories: Redis
tags: Redis command
excerpt: Redis 命令
mathjax: true
---

* content
{:toc}

> Redis 常用命令

### 键命令

```
#key 存在时删除key
DEL key

#序列化给定的key，并返回被序列化的值
DUMP key

#判断一个key是否存在
EXISTS key [seconds]

#为key设置过期时间, 时间(以秒计)； 使用场景
# 限时优惠
# 网站数据缓存 （积分排行榜）
# 手机验证码
# 网站访客访问频率 （一分钟最多访问10次）
EXPIRE key seconds

#为key设置过期时间，时间(以毫秒计)
PEXPIRE key milliseconds

# 查看key的剩余生存时间， -1 永久有效， -2 失效
TTL Key

# 移除过期时间
persist Key

# 用通配符查key；"?" 代表任意一个字符，'*'代表所有
KEYS Pattern

# 切换数据库
select indexindex

# 从当前数据库中随机返回一个
random key

# 重命名key
raname key newname

# 将当前数据库的key移到给定的key
move key db

# 返回key的类新
type key
```




    