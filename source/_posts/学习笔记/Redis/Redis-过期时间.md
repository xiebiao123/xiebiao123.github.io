---
title: Redis 过期时间
date: 2020-10-12
categories:
	- 学习
tags:
    - redis
---

##### redis如何清理过期key
redis出于性能上的考虑，无法做到对每一个过期的key进行即时的过期监听和删除。但是redis提供了其它的方法来清理过期的key

* **被动清理**

当用户主动访问一个过期的key时，redis会将其从内存中删除

* **主动清理**

在redis的持久化中，我们知道redis为了保持系统的稳定性，健壮性，会周期性的执行一个函数。在这个过程中，会进行之前已经提到过的自动的持久化操作，同时也会进行内存的主动清理。

在内存主动清理的过程中，redis采用了一个随机算法来进行这个过程：简单来说，redis会随机的抽取N(默认100)个被设置了过期时间的key，检查这其中已经过期的key，将其清除。同时，如果这其中已经过期的key超过了一定的百分比M(默认是25)，则将继续执行一次主动清理，直至过期key的百分比在概率上降低到M以下    

* **内存不足触发主动清理**

在redis的内存不足时，也会触发主动清理。

##### redis内存不足时候，清理key的策略

redis是一个基于内存的数据库，如果存储的数据量很大，达到了内存限制的最大值，将会出现内存不足的问题。redis允许用户通过配置maxmemory-policy参数，指定redis在内存不足时的解决策略，具体参数如下：

1. volatile-lru 使用LRU算法删除一个键(只针对设置了过期时间的key
2. **allkeys-lru** 使用LRU算法删除一个键
3. volatile-lfu 使用LFU算法删除一个键(只针对设置了过期时间的键)
4. allkeys-lfu 使用LFU算法删除一个键
5. volatile-random 随机删除一个键(只针对设置了过期时间的键)
6. allkeys-random 随机删除一个键
7. volatile-ttl 删除最早过期的一个键
8. noeviction 不删除键，返回错误信息(redis默认选项)

> 注意：
>   * **LRU** 是最近最少使用置换算法(Least Recently Used),也就是首先淘汰最长时间未被使用的key
>   * **LFU** 是最近最不常用置换算法(Least Frequently Used),也就是淘汰一定时期内被访问次数最少的key