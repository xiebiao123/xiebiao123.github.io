---
title: RabbitMQ
date: 2018-07-02
categories:
	- 学习
tags:
    - MQ
---

##### [1.生产RabbitMQ队列阻塞该如何处理](https://www.jianshu.com/p/b18ea3a7c482)
* 现象：
  
由于代码原因，导致消息没有ack，重新入队列头，恶性循环，最终导致消费挤压，无法消费新的消息
* 原因：

由于没有进行ack导致队里阻塞。那么问题来了，这是为什么呢？其实这是RabbitMQ的一种保护机制。防止当消息激增的时候，海量的消息进入consumer而引发consumer宕机。

RabbitMQ提供了一种**QOS(服务质量保证)功能**，即在非自动确认的消息的前提下，限制信道上的消费者所能保持的最大未确认的数量。可以通过设置PrefetchCount实现。

举例说明:可以理解为在consumer前面加了一个缓冲容器，容器能容纳最大的消息数量就是PrefetchCount。如果容器没有满RabbitMQ就会将消息投递到容器内，如果满了就不投递了。当consumer对消息进行ack以后就会将此消息移除，从而放入新的消息。

```yml
listener:
simple:
    # 消费端最小并发数
    concurrency: 1
    # 消费端最大并发数
    max-concurrency: 5
    # 一次处理的消息数量
    prefetch: 2
    # 手动应答
    acknowledge-mode: manual
```
通过上面的配置发现prefetch我只配置了2，并且concurrency配置的只有1，所以当我发送了2条错误消息以后，由于解密失败这2条消息一直没有被ack。将缓冲区沾满了，这个时候RabbitMQ认为这个consumer已经没有消费能力了就不继续给它推送消息了，所以就造成了队列阻塞

> **注意**
> 在自动确认的情况下，RabbitMQ消息监听程序异常时，consumer会向rabbitmq server发送Basic.Reject，表示消息拒绝接受，由于Spring默认requeue-rejected配置为true，消息会重新入队，然后rabbitmq server重新投递。就相当于死循环了，所以控制台在疯狂刷错误日志造成磁盘利用率飙升的原因

* 解决办法
1. 解决代码问题
2. 将default-requeue-rejected: false即可
   
* 总结
  * 个人建议，生产环境不建议使用自动ack，这样会QOS*无法生效。
  * 在使用手动ack的时候，需要非常注意消息签收。
  * 其实在将有问题的MQ重置时，是将错误的消息给清除才没有问题了，相当于是消息丢失了

##### 2.配置详解

    spring.rabbitmq.addresses =＃客户端应连接的地址的逗号分隔列表。
    spring.rabbitmq.cache.channel.checkout-timeout =＃如果已经达到缓存大小，等待获得频道的毫秒数。
    spring.rabbitmq.cache.channel.size =＃要在缓存中保留的通道数。
    spring.rabbitmq.cache.connection.mode = CHANNEL＃连接工厂缓存模式。
    spring.rabbitmq.cache.connection.size =＃缓存的连接数。
 <!-- more -->
    spring.rabbitmq.connection-timeout =＃连接超时，以毫秒为单位;零无限。
    spring.rabbitmq.dynamic = true＃创建一个AmqpAdmin bean。
    spring.rabbitmq.host = localhost＃RabbitMQ主机。
    spring.rabbitmq.listener.acknowledge-mode =＃容器的确认模式。
    spring.rabbitmq.listener.auto-startup = true＃启动时自动启动容器。
    spring.rabbitmq.listener.concurrency =＃最小消费者数量。
    spring.rabbitmq.listener.default-requeue-rejected =＃是否要拒绝递送失败;默认`true`。
    spring.rabbitmq.listener.max-concurrency =＃消费者的最大数量。
    spring.rabbitmq.listener.prefetch =＃在单个请求中要处理的消息数。它应该大于或等于事务大小（如果使用）。
    spring.rabbitmq.listener.retry.enabled = false＃是否启用发布重试。
    spring.rabbitmq.listener.retry.initial-interval = 1000＃第一次和第二次尝试传递消息的时间间隔。
    spring.rabbitmq.listener.retry.max-attempts = 3＃传递消息的最大尝试次数。
    spring.rabbitmq.listener.retry.max-interval = 10000＃尝试之间的最大间隔。
    spring.rabbitmq.listener.retry.multiplier = 1.0＃应用于先前传递重试时间间隔的乘数。
    spring.rabbitmq.listener.retry.stateless = true＃是否重试是无状态或有状态的。
    spring.rabbitmq.listener.transaction-size =＃事务中要处理的消息数。为获得最佳结果，它应小于或等于预取计数。
    spring.rabbitmq.password =＃登录对经纪人进行身份验证。
    spring.rabbitmq.port = 5672＃RabbitMQ端口。
    spring.rabbitmq.publisher-confirms = false＃启用发布商确认。
    spring.rabbitmq.publisher-returns = false＃启用发布商退货。
    spring.rabbitmq.requested-heartbeat =＃请求的心跳超时，以秒为单位;零没有。
    spring.rabbitmq.ssl.enabled = false＃启用SSL支持。
    spring.rabbitmq.ssl.key-store =＃保存SSL证书的密钥存储区的路径。
    spring.rabbitmq.ssl.key-store-password =＃用于访问密钥存储区的密码。
    spring.rabbitmq.ssl.trust-store =＃持有SSL证书的信任库。
    spring.rabbitmq.ssl.trust-store-password =＃用于访问信任存储的密码。
    spring.rabbitmq.ssl.algorithm = #SSL算法使用。默认情况下由rabbit客户端库配置。
    spring.rabbitmq.template.mandatory = false＃启用强制消息。
    spring.rabbitmq.template.receive-timeout = 0＃receive（）方法超时。
    spring.rabbitmq.template.reply-timeout = 5000＃sendAndReceive（）方法超时。
    spring.rabbitmq.template.retry.enabled = false＃设置为true以在“RabbitTemplate”中启用重试。
    spring.rabbitmq.template.retry.initial-interval = 1000＃发布消息的第一次和第二次尝试之间的时间间隔。
    spring.rabbitmq.template.retry.max-attempts = 3＃发布消息的最大尝试次数。
    spring.rabbitmq.template.retry.max-interval = 10000＃尝试发布消息的最大次数。
    spring.rabbitmq.template.retry.multiplier = 1.0＃应用于先前发布重试时间间隔的乘数。
    spring.rabbitmq.username =＃登录用户向代理进行身份验证。
    spring.rabbitmq.virtual-host =＃连接到代理时使用的虚拟主机
