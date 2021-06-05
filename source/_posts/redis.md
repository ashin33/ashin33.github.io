---
title: redis整理（一）
date: 2021-04-16 15:58:42
tags: [2021,redis]
categories: [代码]
top_img: /img/redis.jpeg
cover: /img/redis.jpeg
---
>最近一直在面试，发现redis的知识还是太薄弱了，慢慢梳理一下。

# redis为何物
redis是基于内存的可持久化的key-value型数据库。
# redis的优点
* 读写速度快
* 支持持久化（AOF，RDB两种方式）
* 支持事务
* 数据类型丰富（string，hash，set，zet，list）
* 支持主从，读写分离

# redis的缺点

# redis数据类型

| 类型     | 数据类型                |
|--------|---------------------|
| [string](#string) | 字符串，整数，浮点型          |
| [hash](#hash)   | 有键值对的列表             |
| [set](#set)    | 字符串的无序集合，每个字符串都是唯一的 |
| [zet](#zet)    | 有序集合，由浮点key和字符串值组成  |
| [list](#list)   | list链式表，每个节点上都一个值   |
## <a id="string">string</a>
1. 获取值 `get key`
2. 设置值 `set key`
3. 删除值 `del key`
4. 自增 `incr key`
5. 按值自增 `incrby key`

![-w510](/images/16187382102057.jpg)

## <a id="hash">hash</a>
1. 设置值(可同时设置多个，空格分隔)`hset key field1 [field2]`返回入库条数
2. 获取值`hget key field`，返回值
3. 获取所有值`hgetall key`，返回所有键值
4. 删除值(可同时删除过个)`hdel key field1 [field2]`，返回删除条数
5. 删除整个hash`del`，返回删除的hash表数

![-w600](/images/16187431843823.jpg)

## <a id="set">set</a>
1. 向集合添加一个（多个成员）`sadd key field1 [field2]`,返回操作条数
2. 获取集合的成员数 `scard key`
3. 获取第一个集合独有的元素，只传第一个参数则与库中所有的其他集合比较`sdiff key1 [key2]`
4. 获取第一个集合的交集，只传第一个集合则与库中所有的其他集合比较`sinter key1 [key2]`
5. 移除集合中的一个或多个元素 `srem key field1 [field2]`

![-w505](/images/16212344696690.jpg)

## <a id="zet">zet</a>
1. 向集合中添加元素 `zadd key index1 value1 [index2 value2]`,返回新增的元素数，不包括被更新的元素
2. 获取集合中的元素数 `zcard key`
3. 获取集合中在指定区间中的元素数 `zcount key min max`
4. 移除集合中的一个或多个元素 `zrem key value1 [value2]`
![-w387](/images/16212394834115.jpg)

## <a id="list">list</a>
1. 将一个或多个元素插入列表头 `lpush key value1 [value2]`,返回列表长度
2. 将一个或多个元素插入列表尾 `rpush key value1 [value2]`,返回列表长度
3. 从头部移除并获取一个元素 `lpop key`，返回元素值，不存在返回null
4. 从尾部移除并获取一个元素 `rpop key`,返回元素值，不存在返回null
![-w379](/images/16212429789072.jpg)

# redis的持久化
## 两种持久化类型
1. 全量的RDB持久化
    指定时间间隔内，fork一个子进程，将数据写入临时文件，最后一次性替换之前的文件
2. 增量的AOF持久化
    以文本的日志形式，记录每一个写、删除操作
    
##  优缺点
### RDB优点
1. 备份频率和周期可以灵活自定义
2. 可以保持redis的高性能，不影响redis读写
3. 恢复快

### RDB缺点
1. 无法保证数据的高可用性
2. 数据较大时，子进程可能会使服务停止几百毫秒甚至1秒

### AOF优点
1. 有更高的数据可用性
2. 宕机不会破坏已存储的日志内容
3. 有一个可读的日志文件

### AOF缺点
1. 恢复速度比RDB慢
2. 运行效率往往比RDB慢

## 选择
当使用bgsave全量持久化时，选择AOF来进行增量的持久化。
在对数据要求没有那么严格时，推荐RDB持久化，恢复速度更快

# 如何使用redis实现消息队列
Redis 的 list(列表) 数据结构常用来作为异步消息队列使用，使用rpush/lpush操作入队列，使用lpop 和 rpop来出队列。rpush 和 lpop 结合 或者lpush 和rpop 结合。

客户端是通过队列的 pop 操作来获取消息，然后进行处理。处理完了再接着获取消息，再进行处理。如此循环往复，这便是作为队列消费者的客户端的生命周期。

# hash表的使用场景

>添加一个雲放同学提供的hash使用场景


```
#购物车

#用户1001添加1件10091商品
hset cart:1001 10091 1 

#用户1001添加2件10021商品
hset cart:1001 10021 2 

#用户1001增加1件10091商品
hincrby cart:1001 10091 1 

#获取用户1001的购物车商品总数
hlen cart:1001 

#删除用户1001的商品10091
hdel cart:1001 10091 

#查看用户1001的购物车
hgetall cart:1001

#清空用户1001的购物车
del cart:1001
```


