---
title: 分布式唯一ID详解
date: 2019-09-10
categories:
    - 学习
tags:
    - 唯一ID
---

### 业界常用ID解决方案

* **UUID** 性能非常高，没有网络消耗。缺点就是无序的字符串，不具备有趋势自增的特性，UUID太长也不便于存储，浪费存储空间，所以很多的场景是不使用的
* **Redis发号器** 利用Redis的INCR和INCRBY来实现，原子操作，线程安全，性能比Mysql强劲。缺点就是需要占用网络资源，增加系统复杂度
* **Snowflake雪花算法** 它是twitter开源式的分布式ID生成算法，代码实现简单，而且不占用宽带，数据的迁移不会受到影响，生成的id里包含有时间戳，所以生成的id按照时间进行递增，部署多台服务器的话，需要保证系统时间是一样的。缺点就是依赖系统时钟
* **百度UidGenerator** UidGenerator跟雪花算法不一样，它可以自定义时间戳，工作机器id与序列号各部位的位数，用于不同的场景

#### Snowflake雪花算法

雪花算法是默认使用主键生成方案，生成一个64bit的长整型数据。sharding-jdbc中雪花算法生成的主键主要是由四个部分组成，1bit符号位、41bit时间戳位、10bit机器id，12bit序列号。

![雪花算法结构](/images/java/雪花算法结构.png)

* 符号位（1bit）
  * 在Java当中，Long型最高的位是符号位，也就是正数是0，负数是1，一般id生成都为正数，所以的话默认是0
* 时间戳（41bit）
  * 41位的时间戳占41个bit，毫秒数是2的41次幂，而一年的总毫秒数就为1000L * 60 * 60 * 24 * 365，为服务上线的时间毫秒级时间戳（当前时间-服务第一次上线的时间）
* 工作机器id（10bit）
  * 表示一个唯一的工作进程，它的默认值是0，可以通过key-generator.props.worker.id来进行设置
* 序列号（12bit）
  * 可以允许同一毫秒生成2^12=4096个id，理论上一秒就可以生成400万个id。

##### 为啥会出现时钟回拨这种现象

有一种网络的时间协议叫ntp，专门是用来进行同步或者是用来校准网络计算机的时间。这就是手机现在不用手动校对时间的原因。当硬件因为某一些原因导致时间快或者是慢了，这个时候就需要使用ntp服务来对时间进行校准，在做校准的时候就有可能发生服务器时钟的跳跃或者是回拨这些问题

##### 雪花算法怎么样解决时钟回拨问题

上面提到服务器时钟回拨问题可能会导致重复的id产生，所以在Snowflake方案对雪花算法进行了改进，添加了一个最大的容忍时钟回拨毫秒数

当时钟回拨的时间超过最大容忍毫秒数的阀值的话，就直接程序报错。如果在可以容忍的范围内的话，就默认使用了分布式的主键生成器，等待时钟同步到最后一次生成主键时间后才继续开始工作

最大容忍的时钟回拨毫秒数默认是0，max.tolerate.time.difference.milliseconds来设置

```java

publicfinalclassSnowflakeShardingKeyGeneratorimplementsShardingKeyGenerator{
  @Getter
  @Setter
  privatePropertiesproperties=newProperties();

  publicStringgetType(){
    return"SNOWFLAKE";
  }

  publicsynchronizedComparable<?>generateKey(){
    /**
    *当前系统时间毫秒数
    */
    longcurrentMilliseconds=timeService.getCurrentMillis();
    /**
    *判断是否需要等待容忍时间差，如果需要，则等待时间差过去，然后再获取当前系统时间
    */
    if(waitTolerateTimeDifferenceIfNeed(currentMilliseconds)){
    currentMilliseconds=timeService.getCurrentMillis();
  }
  /**
  *如果最后一次毫秒与当前系统时间毫秒相同，即还在同一毫秒内
  */
  if(lastMilliseconds==currentMilliseconds){
    /**
    *&位与运算符：两个数都转为二进制，如果相对应位都是1，则结果为1，否则为0
    *当序列为4095时，4095+1后的新序列与掩码进行位与运算结果是0
    *当序列为其他值时，位与运算结果都不会是0
    *即本毫秒的序列已经用到最大值4096，此时要取下一个毫秒时间值
    */
    if(0L==(sequence=(sequence+1)&SEQUENCE_MASK)){
      currentMilliseconds=waitUntilNextTime(currentMilliseconds);
    }
  }else{
    /**
    *上一毫秒已经过去，把序列值重置为1
    */
    vibrateSequenceOffset();
    sequence=sequenceOffset;
  }
  lastMilliseconds=currentMilliseconds;

  /**
  *XX......XXXX0000000000000000000000时间差XX
  *XXXXXXXXXX000000000000机器IDXX
  *XXXXXXXXXXXX序列号XX
  *三部分进行|位或运算：如果相对应位都是0，则结果为0，否则为1
  */
  return((currentMilliseconds-EPOCH)<<TIMESTAMP_LEFT_SHIFT_BITS)|
      (getWorkerId() <<WORKER_ID_LEFT_SHIFT_BITS)|sequence;
  }

  /**
  *判断是否需要等待容忍时间差
  */
  @SneakyThrows
  privatebooleanwaitTolerateTimeDifferenceIfNeed(finallongcurrentMilliseconds){
    /**
    *如果获取ID时的最后一次时间毫秒数小于等于当前系统时间毫秒数，属于正常情况，则不需要等待
    */
    if(lastMilliseconds<=currentMilliseconds){
      returnfalse;
    }
    /**
    *===>时钟回拨的情况（生成序列的时间大于当前系统的时间），需要等待时间差
    */
    /**
    *获取ID时的最后一次毫秒数减去当前系统时间毫秒数的时间差
    */
    longtimeDifferenceMilliseconds=lastMilliseconds-currentMilliseconds;
    /**
    *时间差小于最大容忍时间差，即当前还在时钟回拨的时间差之内
    */
    Preconditions.checkState(timeDifferenceMilliseconds<getMaxTolerateTimeDifferenceMilliseconds(),
    "Clockismovingbackwards,lasttimeis%dmilliseconds,currenttimeis%dmilliseconds",lastMilliseconds,currentMilliseconds);
    /**
    *线程休眠时间差
    */
    Thread.sleep(timeDifferenceMilliseconds);
    returntrue;
  }

  //配置的机器ID
  privatelonggetWorkerId(){
    longresult=Long.valueOf(properties.getProperty("worker.id",String.valueOf(WORKER_ID)));
    Preconditions.checkArgument(result>=0L&&result<WORKER_ID_MAX_VALUE);
    returnresult;
  }

  privateintgetMaxTolerateTimeDifferenceMilliseconds(){
    returnInteger.valueOf(properties.getProperty("max.tolerate.time.difference.milliseconds",String.valueOf(MAX_TOLERATE_TIME_DIFFERENCE_MILLISECONDS)));
  }

  privatelongwaitUntilNextTime(finallonglastTime){
    longresult=timeService.getCurrentMillis();
    while(result<=lastTime){
    result=timeService.getCurrentMillis();
    }
    returnresult;
  }
}
```

可以看出最后的主键生成时间（lastMilliseconds）与当前（currentMilliseconds）做比较，如果生成的时间大于当前时间的话，这就说明了时钟回调了。那么这时会接着判断两个时间的差，看下是否在设置在最大的容忍时间范围内，在范围内就睡眠差值的时间，大于差值那就直接报异常

#### 百度UidGenerator

UidGenerator算法是对雪花算法的改进版。UidGenerator组成是由：sign(1bit)+delta seconds(28bits)+worker node id(22bits)+sequence(13bits)

1. sign 固定的符号标识，生成的UID为正数
2. delta seconds：当前的时间
3. worker id：机器id，内置实现是在启动的时候由数据库分配，默认分配策略为：用后就弃掉
4. sequence：每秒下的并发序列，13bits可以支持每一秒8192个并发

UidGenerator可以保证指定的机器同一时间某一并发序列是唯一的，并由此生成一个64bits的唯一id。
UidGenerator跟雪花算法不一样，它可以自定义时间戳，工作机器id与序列号各部位的位数，用于不同的场景。

##### UidGenerator两种方式
* DefaultUidGenerator 对于时钟回拨问题比较的简单,抛出UidGenerateException异常，根据业务情况来调整字段占用的位数
* CachedUidGenerator 是在DefaultUidGenerator进行改进的，利用了RingBuffer，它的本质是一个数组，数组里的每一个项都叫slot。而CachedUidGenerator是设计两个RingBuffer，一个是用于保存唯一id，一个保存flag

![CachedUidGenerator结构](/images/java/CachedUidGenerator结构.png)

* RingBuffer Of Flag

其中，保存flag这个RingBuffer的每个slot的值都是0或者1，0是CAN_PUT_FLAG的标志位，1是CAN_TAKE_FLAG的标识位。每个slot的状态要么是CAN_PUT，要么是CAN_TAKE。以某个slot的值为例，初始值为0，即CAN_PUT。接下来会初始化填满这个RingBuffer，这时候这个slot的值就是1，即CAN_TAKE。等获取分布式ID时取到这个slot的值后，这个slot的值又变为0，以此类推。

* RingBuffer Of UID

保存唯一ID的RingBuffer有两个指针，Tail指针和Cursor指针。

1. Tail指针表示最后一个生成的唯一ID。如果这个指针追上了Cursor指针，意味着RingBuffer已经满了。这时候，不允许再继续生成ID了。用户可以通过属性rejectedPutBufferHandler指定处理这种情况的策略。
2. Cursor指针表示最后一个已经给消费的唯一ID。如果Cursor指针追上了Tail指针，意味着RingBuffer已经空了。这时候，不允许再继续获取ID了。用户可以通过属性rejectedTakeBufferHandler指定处理这种异常情况的策略。

另外，如果你想增强RingBuffer提升它的吞吐能力，那么需要配置一个更大的boostPower值：

```xml
<!-- RingBuffer size扩容参数, 可提高UID生成能力.即每秒产生ID数上限能力 --> 
<!-- 默认:3，原bufferSize=2^13, 扩容后bufferSize = 2^13 << 3 = 65536 -->
<property name="boostPower" value="3"/>
```

CachedUidGenerator主要是通过以下的集中方式规避了时钟的问题和增强了唯一性
1、自增列：在每一次重启的时候workerid都会初始化，且它就是数据库自增的id，这样完美的实现每一个实例获取到的workerid都不一样，不会造成冲突
2、RingBuffer：不需要在每次取得ID都要计算分布式ID，而是利用RingBuffer数据结构预先生成若干个分布式ID来保存
3、时间递增：像雪花算法都是通过currentTimeMillis来获取时间，并比较，这样的话是很依赖服务器的时间，但是CachedUidGenerator的时间类型是AtomicLong，通过incrementAndGet方法来获取下一次的时间，这样就避免了对服务器时间的依赖，也就是时钟回拨的问题可以得到解决。

CachedUidGenerator通过缓存的这种方式来预先生成唯一ID列表，这种可以解决唯一ID所消耗的时间，但是也有不好的点就是，需要耗费内存来缓存这一部分ID的数据，如果访问量不是很大的情况下，提前生成的UID的时间戳可能是很早以前的。