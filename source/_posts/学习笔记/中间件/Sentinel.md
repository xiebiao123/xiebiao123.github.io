---
title: Sentinel
date: 2019-07-19
categories:
    - 学习
tags:
    - sentinel
---

### Sentinel介绍

Sentinel是阿里中间件团队开源、面向分布式服务架构的轻量级的高可用的限流控制组件，主要以流量为切入点，从 **流量控制、熔断降级、系统负载保护** 等多个维度来帮助用户保护服务的稳定性

### Sentinel 与 Netflix Hystrix 直接差异

|对比内容 | Sentinel | Hystrix |
|---|---|---|
|隔离策略 | 型号量隔离 | 线程池隔离/信号量隔离 |
|熔断降级策略 | 基于响应时间或失败比率 | 基于失败比率|
|实时指标实现 | 滑动窗口| 滑动窗口(RxJava)|
|规则配置 | 支持多种数据源 | 支持多种数据源 |
|扩展性 | 多个扩展点 | 插件的形式 |
|基于注解的支持 | 支持 | 支持 |
|限流 | 基于QPS,支持基于调用关系的限流 | 不支持 |
|流量整形 | 支持慢启动、均速器模式 | 不支持|
|系统负载保护 | 支持 | 不支持 |
|控制台 | 开箱即用、可配置规则、查看秒级监控、机器发现等 | 不完善 |
|常见框架的适配 | Servlet、Spring Cloud、Dubbo、gRPC等 | Servlet、Spring Cloud、Netflix|

<!-- more -->
### 原理图

![原理图](/images/java/sentinel原理图.png)

### 框架

sentinel通过Entry 对象进行流量管理，每个Entry创建的时候会创建7个slot，通过责任链的模式处理slot。

1. **NodeSelectorSlot** 把资源的调用路径，以树状结构存储起来
2. **ClusterBuilderSlot** 资源的统计信息以及调用者信息，例如该资源的RT,QPS,thread count 等等
3. **StatisticSlot** 记录、统计不同维度的runtime指标监控信息
4. **SystemSlot** 通过判断系统相关指标来进行限流，主要的指标为qps、总线程数、rt、系统负载
5. **AuthoritySlot** 黑白名单
6. **FlowSlot** 限流规则
7. **DegradeSlot** 熔断降级规则

### Sentinel 控制台

Sentinel 控制台是流量控制、熔断降级规则统一配置和管理的入口，它为用户提供了机器自发现、簇点链路自发现、监控、规则配置等功能。在 Sentinel 控制台上，我们可以配置规则并实时查看流量控制效果。

#### 1. 如何编译

1. 下载源码 [Sentinel](https://github.com/alibaba/Sentinel/tree/master)
2. mvn clean install

#### 2. 如何启动

``` shell
java -Dserver.port=8080 \
-Dcsp.sentinel.dashboard.server=localhost:8080 \
-Dproject.name=sentinel-dashboard \
-jar target/sentinel-dashboard.jar
```

从 Sentinel 1.6.0 开始，Sentinel 控制台支持简单的登录功能，默认用户名和密码都是 sentinel。用户可以通过如下参数进行配置：

- Dsentinel.dashboard.auth.username=sentinel 用于指定控制台的登录用户名为 sentinel；
- Dsentinel.dashboard.auth.password=123456 用于指定控制台的登录密码为 123456；如果省略这两个参数，默认用户和密码均为 sentinel；
- Dserver.servlet.session.timeout=7200 用于指定 Spring Boot 服务端 session 的过期时间，如 7200 表示 7200 秒；60m 表示 60 分钟，默认为 30 分钟；

[配置详细](https://github.com/alibaba/Sentinel/wiki/%E5%90%AF%E5%8A%A8%E9%85%8D%E7%BD%AE%E9%A1%B9)

#### 3. 客户端接入

##### 引入Jar包

``` xml
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-transport-simple-http</artifactId>
    <version>x.y.z</version>
</dependency>
```

##### 配置启动参数

启动时加入 JVM 参数 -Dcsp.sentinel.dashboard.server=consoleIp:port 指定控制台地址和端口。若启动多个应用，则需要通过 -Dcsp.sentinel.api.port=xxxx 指定客户端监控 API 的端口（默认是 8719）。

除了修改 JVM 参数，也可以通过配置文件取得同样的效果。更详细的信息可以参考 [启动配置项](https://github.com/alibaba/Sentinel/wiki/%E5%90%AF%E5%8A%A8%E9%85%8D%E7%BD%AE%E9%A1%B9)

##### 触发客户端初始化

**确保确保客户端有访问量，Sentinel会在客户端首次调用的时候进行初始化，开始向控制台发送心跳包**

>注意：您还需要根据您的应用类型和接入方式引入对应的 适配依赖，否则即使有访问量也不能被 Sentinel 统计。

##### 规则配置

- [熔断降级](https://github.com/alibaba/Sentinel/wiki/%E7%86%94%E6%96%AD%E9%99%8D%E7%BA%A7)
  - **平均响应时间 (DEGRADE_GRADE_RT):** 1秒内进入5个请求的平均响应时间，超过阀值(count,以ms为单位)，进行熔断
  - **异常比例(DEGRADE_GRADE_EXCEPTION_RATIO):** 当资源的每秒请求量>=5，并且异常总数占比超过阀值[0.0,1.0],代表0% ~ 100%，进行熔断
  - **异常数(DEGRADE_GRADE_EXCEPTION_COUNT):** 当资源近一分钟内的异常数目超过阀值之后就会进行熔断，**若timeWindow 值小于60s,则熔断结束后，仍可能再进入熔断**
- [流量控制](https://github.com/alibaba/Sentinel/wiki/%E6%B5%81%E9%87%8F%E6%8E%A7%E5%88%B6)
  - 限流规则主要由下面几个因素组成
    - resource：资源名，即限流规则的作用对象
    - count：限流阀值
    - grade：限流阀值类型(QPS或并发线程数)
      - **并发线程数** 统计当前请求中上下文中的线程数，超过阀值直接拒绝，效果类似于信号量隔离
      - **QPS** 当QPS超过某个阀值的时候，则进行流量控制（对应FlowRule中的controlBehavior字段）
        - **直接拒绝** 默认
        - **Warm Up**  冷启动，在一定时间内逐渐增加到阀值上限，给冷系统一个预热的时间，避免冷系统压垮
        - **均速排队** 严格控制请求通过的间隔时间(如QPS为2，则每隔500ms才允许通过下一个请求)，对应的是漏桶算法
    - limitApp：流控针对的调用来源 可以通过 <http://localhost:8719/origin?id=nodeA> 查看
      - **default** 不区分调用者
      - **{some_origin_name}** 表示针对特定的调用者才会进行流量控制
      - **other** 表示针对除{some_origin_name}以外的其余调用方的流量控制
    - strategy：调用关系限流策略
    - controlBehavior：流量控制效果(直接拒绝、Warm Up、均速排队)
- [热点参数限流](https://github.com/alibaba/Sentinel/wiki/%E7%83%AD%E7%82%B9%E5%8F%82%E6%95%B0%E9%99%90%E6%B5%81)
  - Sentinel利用LRU策略统计最近常访问的热点参数，结合令牌桶算法来进行参数级别的限流
- [系统自适应限流](https://github.com/alibaba/Sentinel/wiki/%E7%B3%BB%E7%BB%9F%E8%87%AA%E9%80%82%E5%BA%94%E9%99%90%E6%B5%81)
  - **Load** 当系统load1超过阀值，且系统当前的并发线程数超过系统容量的时候，才会触发保护。系统的容量由系统 maxQps\*minRt  计算得出。设置参考值一般是CPU cores\*2.5
  - **RT** 当单台机器上所有入口流量的平均RT达到阀值，即进行系统保护单位是毫秒
  - **线程数** 当单台机器上所有入口流量的并发线程数达到阀值即触发系统保护
  - **入口QPS** 当单台机器上所有入口流量的QPS达到阀值即触发系统保护
- [黑白名单控制](https://github.com/alibaba/Sentinel/wiki/%E9%BB%91%E7%99%BD%E5%90%8D%E5%8D%95%E6%8E%A7%E5%88%B6)
- [实时监控](https://github.com/alibaba/Sentinel/wiki/%E5%AE%9E%E6%97%B6%E7%9B%91%E6%8E%A7) 
  - 簇点监控
  - 链路监控

### Sentinel基本使用方法

``` java
Entry entry = null;
// 务必保证finally会被执行
try{
    // 资源名可使用任意有业务语义的字符串
    entry = SphU.entry("自定义资源名");
    //被保护的业务逻辑
    //do something...
}catch(BlockException e1){
    //资源访问阻止，被限流或者降级
    //进行相应的处理操作
}finally{
    if(entry != null){
        entry.exit();
    }
}
```

### Sentinel使用案例

- 某台机器由于请求过多，线程数创建庞大，导致上下文切换频繁，降低了响应时间
  - 通过利用sentinel监控并发线程数，进行阈值降级
- 某台计算服务器，每秒只能接受5个计算请求
  - 通过QPS，超过5的请求直接拒绝
- 某台服务器启动时，需要进行大量的请求，直接把调用系统压垮
  - 通过Warm Up的方式，慢慢的请求
- 某服务，只在某时间点业务量剧增，其他时间点大部分处于空闲状态，不想用kafka、rocketmq削峰
  - 通过均速排队的方法，按照一定速率去请求服务
- 某个数据库有多方调用，有时读取数据库太占资源，影响了写的功能
  - 通过关键读与写Entry，设置写的优先级更高，从而降低读对写的影响
- 提供出去给应用商的接口，由于应用商被攻击，导致请求泛滥
  - 通过限制调用方，来降级熔断
- 系统某个服务被多个接口调用(如A、B)，如果我们想限制A接口的调用，B接口则不限制
  - sentinel记录了调用链，调用服务的时候判断是A还是B接口调用，如果是A则限制

### 生产环境使用Sentinel

- [集群流控](https://github.com/alibaba/Sentinel/wiki/%E9%9B%86%E7%BE%A4%E6%B5%81%E6%8E%A7#%E9%85%8D%E7%BD%AE%E6%96%B9%E5%BC%8F)