---
title: RabbitMQ

categories:
	- 笔记

tags:
    - MQ
---

##### 1.配置详解
    spring.rabbitmq.addresses =＃客户端应连接的地址的逗号分隔列表。
    spring.rabbitmq.cache.channel.checkout-timeout =＃如果已经达到缓存大小，等待获得频道的毫秒数。
    spring.rabbitmq.cache.channel.size =＃要在缓存中保留的通道数。
    spring.rabbitmq.cache.connection.mode = CHANNEL＃连接工厂缓存模式。
    spring.rabbitmq.cache.connection.size =＃缓存的连接数。
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