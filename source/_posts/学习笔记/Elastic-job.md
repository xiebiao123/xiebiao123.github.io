---
title: elastic-job
date: 2019-07-19
categories:
    - 学习
tags:
    - elastic-job
---

### 分片策略

数据分片的目的在于把一个任务分散到不同的机器上运行，既可以解决单机计算能力上限的问题，也能降低部分任务失败对整体系统的影响。elastic-job并不直接提供数据处理的功能，框架只会将分片项分配至各个运行中的作业服务器（其实是Job实例，部署在一台机器上的多个Job实例也能分片），开发者需要自行处理分片项与真实数据的对应关系。框架也预置了一些分片策略：平均分配算法策略，作业名哈希值奇偶数算法策略，轮转分片策略。同时也提供了自定义分片策略的接口。

#### 分片原理

elastic-job的分片是通过zookeeper来实现的。节点的分片由主节点分配，如下三种情况都会触发主节点上的分片算法执行：

1. 新的Job实例加入集群
2. 现有的Job实例下线（如果下线的是leader节点，那么先选举然后触发分片算法的执行）
3. 主节点选举
上述三种情况，会让zookeeper上leader节点的sharding节点上多出来一个necessary的临时节点，主节点每次执行Job前，都会去看一下这个节点，如果有则执行分片算法。

分片的执行结果会存储在zookeeper上，5个分片，每个分片应该由哪个Job实例来运行都已经分配好。分配的过程就是上面触发分片算法之后的操作。分配完成之后，各个Job实例就会在下次执行的时候使用上这个分配结果。

每个job实例任务触发前都会获取本任务在本实例上的分片情况，然后封装成shardingContext，传递给调用任务的实际执行方法：

<!-- more -->

``` java
/**
 * 执行作业.
 *
 * @param shardingContext 分片上下文
 */
void execute(ShardingContext shardingContext);
```

### 分片算法

所有的分片策略都继承JobShardingStrategy接口。根据当前注册到ZK的实例列表和在客户端配置的分片数量来进行数据分片。最终将每个Job实例应该获得的分片数字返回出去。 方法签名如下：

``` java
/**
 * 作业分片.
 * 
 * @param jobInstances 所有参与分片的单元列表
 * @param jobName 作业名称
 * @param shardingTotalCount 分片总数
 * @return 分片结果
 */
Map<JobInstance, List<Integer>> sharding(List<JobInstance> jobInstances, String jobName, int shardingTotalCount);
```

分片函数的触发，只会在leader选举的时候触发，也就是说只会在刚启动和leader节点离开的时候触发，并且是在leader节点上触发，而其他节点不会触发。

#### 1. 基于平均分配算法的分片策略

基于平均分配算法的分片策略对应的类是：AverageAllocationJobShardingStrategy。它是默认的分片策略。它的分片效果如下：

如果有3个Job实例, 分成9片, 则每个Job实例分到的分片是: 1=[0,1,2], 2=[3,4,5], 3=[6,7,8].
如果有3个Job实例, 分成8片, 则每个Job实例分到的分片是: 1=[0,1,6], 2=[2,3,7], 3=[4,5].
如果有3个Job实例, 分成10片, 则个Job实例分到的分片是: 1=[0,1,2,9], 2=[3,4,5], 3=[6,7,8].

#### 2.作业名的哈希值奇偶数决定IP升降序算法的分片策略

这个策略的对应的类是：OdevitySortByNameJobShardingStrategy，它内部其实也是使用AverageAllocationJobShardingStrategy实现，只是在传入的节点实例顺序不一样，也就是上面接口参数的List<JobInstance>。AverageAllocationJobShardingStrategy的缺点是一旦分片数小于Job实例数，作业将永远分配至IP地址靠前的Job实例上，导致IP地址靠后的Job实例空闲。而OdevitySortByNameJobShardingStrategy则可以根据作业名称重新分配Job实例负载。如：

如果有3个Job实例，分成2片，作业名称的哈希值为奇数，则每个Job实例分到的分片是：1=[0], 2=[1], 3=[]
如果有3个Job实例，分成2片，作业名称的哈希值为偶数，则每个Job实例分到的分片是：3=[0], 2=[1], 1=[]
实现比较简单：

``` java
long jobNameHash = jobName.hashCode();
if (0 == jobNameHash % 2) {
    Collections.reverse(jobInstances);
}
return averageAllocationJobShardingStrategy.sharding(jobInstances, jobName, shardingTotalCount);
```

#### 3.根据作业名的哈希值对Job实例列表进行轮转的分片策略

这个策略的对应的类是：RotateServerByNameJobShardingStrategy，和上面介绍的策略一样，内部同样是用AverageAllocationJobShardingStrategy实现，也是在传入的List<JobInstance>列表顺序上做文章。

#### 4.自定义分片策略

除了可以使用上述分片策略之外，elastic-job还允许自定义分片策略。我们可以自己实现JobShardingStrategy接口，并且配置到分片方法上去，整个过程比较简单，下面仅仅列出通过配置spring来切换自定义的分片算法的例子：

``` xml
<job:simple id="MyShardingJob1" class="nick.test.elasticjob.MyShardingJob1" registry-center-ref="regCenter" cron="0/10 * * * * ?" 
sharding-total-count="5" sharding-item-parameters="0=A,1=B,2=C,3=D,4=E" job-sharding-strategy-class="nick.test.elasticjob.MyJobShardingStrategy"/>
```

### 分片的执行*

同一时间点会触发每个分片的任务，可根据 shardingContext.getShardingItem() 来判断具体在那个分片执行

``` java
switch (shardingContext.getShardingItem()) {
   case 0:
        break;
   default:
        break;
}
```
