---
title: Redis
date: 2018-07-08
categories:
	- 学习
tags:
	- 学习笔记
---

##### 简介
    Redis 是完全开源免费的，遵守BSD协议，是一个高性能的key-value数据库。
    Redis三个特点：
        1. Redis支持数据的持久化，可以将内存中的数据保存在磁盘中，重启的时候可以再次加载进行使用。
        2. Redis不仅仅支持简单的key-value类型的数据，同时还提供list，set，zset，hash等数据结构的存储。
        3. Redis支持数据的备份，即master-slave模式的数据备份。

##### 优势
* **性能极高** – Redis能读的速度是110000次/s,写的速度是81000次/s 。
* **丰富的数据类型** – Redis支持二进制案例的 Strings, Lists, Hashes, Sets 及 Ordered Sets 数据类型操作。
* **原子** – Redis的所有操作都是原子性的，意思就是要么成功执行要么失败完全不执行。单个操作是原子性的。多个操作也支持事务，即原子性，通过MULTI和EXEC指令包起来。
* **丰富的特性** – Redis还支持 publish/subscribe, 通知, key 过期等等特性。

<!-- more -->

##### key
* **REDIS-CLI -H host -P port -A password**  连接远程客户端
* **DEL key**  检查key是否存在
* **EXISTS key**  指定key的过期时间(秒)
* **PEXPIRE key**  指定key的过期时间(毫秒)
* **KEYS pattern(正则表达式)**  查找指定的key
* **PERSIST key**  移除key的过期时间
* **TTL key**  返回key的剩余过期时间(秒)
* **PTTL key**  返回key的剩余过期时间(毫秒)
* **TYPE key**  返回 key 所储存的值的类型

##### 字符串(string)
* **GET key**  获取指定key的值
* **SET key**  设置指定key的值
* **INCR key**  将key中存储的数字加一
* **DECR key**  将key中存储的数字减一
* **INCRBY key number**  将key中存储的数字加指定值
* **DECRBY key number**  将key中存储的数字减指定值


###### 哈希(map)
* **HSET key field**  设置哈希表中指定key指定字段的值
* **HGET key field**  获取哈希表中指定key指定字段的值
* **HMSET key field1 value1 [field2 value2 ]**  设置哈希表中指定key多个字段的值
* **HMGET key field1 [field2]**  获取哈希表中指定key多个字段的值
* **HGETALL key**  获取哈希表中指定key的所有字段和值
* **HDEL key field1 [field2]**  删除一个或多个哈希表字段
* **HEXISTS key field**  判断哈希表中指定key中指定字段是否存在
* **HKEYS key**  获取哈希表中指定key的所有字段
* **HLEN key**  获取哈希表中指定key的字段数量

###### 列表(list)
* 设置超时
    + **BLPOP key1 [key2 ] timeout** 移出并获取列表的第一个元素， 如果列表没有元素
会阻塞列表直到等待超时或发现可弹出元素为止。
	+ **BRPOP key1 [key2 ] timeout** 移出并获取列表的最后一个元素， 如果列表没有元素
会阻塞列表直到等待超时或发现可弹出元素为止。
	+ **BRPOPLPUSH listKey1 listKey2 timeout** 从列表中弹出一个值，将弹出的元素插入到另外一个列表中并返回它;如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。**循环列表 | listKey1(源) = listKey2(目的)**
* L|R
	+ **LPOP key** 移出并获取列表的头部第一个元素
	+ **RPOP key** 移出并获取列表的尾部第一个元素
	+ **LPUSH key value1 [value2]** 将一个或多个值插入到列表头部
	+ **RPUSH key value1 [value2]** 将一个或多个值插入到列表尾部
	+ **LPUSHX key value** 将一个值插入到已存在的列表头部
	+ **RPUSHX key value** 将一个值插入到已存在的列表尾部
* L
    + **LTRIM key start stop** 对一个列表进行修剪(trim)，就是说，让列表只保留指定区间内的元素，
不在指定区间之内的元素都将被删除
	+ **LLEN key** 获取列表长度
	+ **LRANGE key start stop** 获取列表指定范围内的元素
	+ **LINDEX key index** 通过索引获取列表中的元素
	+ **LINSERT key BEFORE|AFTER pivot value** 在列表指定元素前或者后插入元素
* R
	+ **RPOPLPUSH source destination** 从列表中弹出一个值，将弹出的元素插入到另外一个列表中并返回它

###### 集合(set)
* **SADD key member1 [member2]** 向集合添加一个或多个成员
* **SCARD key** 获取集合的成员个数
* **SMEMBERS key** 获取集合中所有成员
* **SDIFF key1 [key2]** 返回给定所有集合的差集|key1-key2
* **SINTER key1 [key2]** 返回给定所有集合的交集
* **SUNION key1 [key2]** 返回所有给定集合的并集
* **SDIFFSTORE destination key1 [key2]** 返回给定所有集合的差集，存储在 destination 中
* **SINTERSTORE destination key1 [key2]** 返回给定所有集合的交集，存储在 destination 中
* **SUNIONSTORE destination key1 [key2]** 返回给定所有集合的并集并，储在 destination 中
* **SISMEMBER key member** 判断 member 元素是否是集合 key 的成员
* **SMOVE source destination member** 将 member 元素从 source 集合移动到 destination 集合
* **SREM key member1 [member2]** 移除集合中一个或多个成员
* **SSCAN key cursor [MATCH pattern] [COUNT count]** 迭代集合中的元素

###### 有序集合(zset)
* **ZADD key score1 member1 [score2 member2]** 向有序集合添加一个或多个成员
* **ZCARD key** 获取有序集合的成员个数
* **ZCOUNT key min max** 计算在有序集合中指定区间分数的成员数
* **ZRANGE key start stop [WITHSCORES]** 通过索引区间返回有序集合成指定区间内的成员
* **ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT]** 通过分数返回有序集合指定区间内的成员
* **ZREM key member [member ...]** 移除有序集合中的一个或多个成员
* **ZREMRANGEBYSCORE key min max** 移除有序集合中给定的分数区间的所有成员
* **ZREVRANGEBYSCORE key max min [WITHSCORES]** 返回有序集中指定分数区间内的成员，分数从高到低排序
* **ZUNIONSTORE destination numkeys key [key ...]** 计算给定的一个或多个有序集的并集，并存储在新的 key 中
* **ZINTERSTORE destination numkeys key [key ...]** 计算给定的一个或多个有序集的交集并将结果集存储在新的有序集合 key 中
* **ZSCAN key cursor [MATCH pattern] [COUNT count]** 迭代有序集合中的元素（包括元素成员和元素分值）

##### 发布|订阅
* 订阅
	+ **PSUBSCRIBE pattern [pattern ...]** 订阅一个或多个符合给定模式的频道。
	+ **SUBSCRIBE channel [channel ...]** 订阅给定的一个或多个频道的信息。
* 发布
	+ **PUBLISH channel message** 将信息发送到指定的频道。
* 退订
	+ **PUNSUBSCRIBE [pattern [pattern ...]]** 退订所有给定模式的频道。
	+ **UNSUBSCRIBE [channel [channel ...]]** 指退订给定的频道。
* 状态 
    + **PUBSUB subcommand [argument [argument ...]]** 查看订阅与发布系统状态。

##### 事务
* **MULTI** 标记一个事务块的开始。
* **EXEC** 执行所有事务块内的命令。
* **WATCH key [key ...]** 监视一个(或多个) key ，如果在事务执行之前这个(或这些) key被其他命令所改动，那么事务将被打断。
* **UNWATCH** 取消 WATCH 命令对所有 key 的监视。
* **DISCARD** 取消事务，放弃执行事务块内的所有命令。