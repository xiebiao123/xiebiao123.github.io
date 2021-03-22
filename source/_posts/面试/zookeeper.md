---
title: zookeeper
date: 2021-01-05
categories:
    - 面试
tags:
    - zookeeper
---

#### 说一说zookeeper有哪些使用场景？
- 1.分布式协调

![分布式系统顺序性](/images/面试/zookeeper-分布式协调.png)
 
- 2.分布式锁

![分布式系统顺序性](/images/面试/zookeeper-分布式锁.png)

* 3.元素据、配置信息管理

![分布式系统顺序性](/images/面试/zookeeper-元素据、配置信息管理.png)

* 4.HA高可用性(主备切换)

![分布式系统顺序性](/images/面试/zookeeper-HA高可用性.png)


#### redis分布式锁 和 ZK分布式锁的对比 ？
1. redis分布式锁，其实需要自己不断去尝试获取锁，比较消耗性能；zk分布式锁，获取不到锁，**注册一个监听器**即可，不需要不断主动尝试获取锁，性能开销较小
2. redis获取锁的那个客户端如果bug了或挂掉，那么只能等待超时时间之后才能释放；而ZK因为创建的是临时节点，只要客户端挂了，znode就没有了，此时自动释放锁



