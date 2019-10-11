---
title: Dubbo
date: 2018-08-28
categories:
    - 学习
tags:
    - dubbo
---
### [注册中心Zookeeper](http://dubbo.apache.org/zh-cn/docs/user/references/registry/zookeeper.html)
Dubbo支持多种注册中心，官方推荐使用Zookeeper，Zookeeper是Apache Hadoop的一个子项目，是一个树形的目录服务，支持变更推送

![zookeeper](/images/dubbo/zookeeper.jpg)

* 流程说明
    * 服务提供者启动时：向 /dubbo/com.foo.BarService/providers 目录下写入自己的 URL 地址
    * 服务消费者启动时：订阅 /dubbo/com.foo.BarService/providers 目录下的提供者 URL 地址。并向 /dubbo/com.foo.BarService/consumers 目录下写入自己的 URL 地址
    * 监控中心启动时：订阅 /dubbo/com.foo.BarService 目录下的所有提供者和消费者 URL 地址
* 支持以下功能：
    * 当提供者出现断电等异常停机时，注册中心能自动删除提供者信息
    * 当注册中心重启时，能自动恢复注册数据，以及订阅请求
    * 当会话过期时，能自动恢复注册数据，以及订阅请求
    * 当设置<dubbo:registry check="false" /> 时，记录失败注册和订阅请求，后台定时重试
    * 可通过<dubbo:registry username="admin" password="1234" /> 设置 zookeeper 登录信息
    * 可通过<dubbo:registry group="dubbo" /> 设置 zookeeper 的根节点，不设置将使用无根树
    * 支持 * 号通配符 <dubbo:reference group="*" version="*" />，可订阅服务的所有分组和所有版本的提供者
 
> 注意:在2.7.x的版本中已经移除了zkclient的实现,使用Consumer来实现（2.3版本添加）

### 配置的覆盖策略
下图展示了配置覆盖关系的优先级，从上到下优先级依次降低：

![configuration](/images/dubbo/dubbo-configuration.jpg)

> * Java虚拟机 > 外部配置（Spring application.properties | Apollo | Nacos）> Spring API(Java Bean) > 本地配置（dubbo.properties）
> * 方法级 >  接口级 > 全局配置
> * 如果级别一样，则消费者 > 提供者

#### 常用配置
* [启动时检查](http://dubbo.apache.org/zh-cn/docs/user/demos/preflight-check.html)
* [集群容错](http://dubbo.apache.org/zh-cn/docs/user/demos/fault-tolerent-strategy.html)
* [负责均衡](http://dubbo.apache.org/zh-cn/docs/user/demos/loadbalance.html)
* [服务降级](http://dubbo.apache.org/zh-cn/docs/user/demos/service-downgrade.html)
* [多版本](http://dubbo.apache.org/zh-cn/docs/user/demos/multi-versions.html)
* [本地存根](http://dubbo.apache.org/zh-cn/docs/user/demos/local-stub.html)
* [参数验证](http://dubbo.apache.org/zh-cn/docs/user/demos/parameter-validation.html)
* [结果缓存](http://dubbo.apache.org/zh-cn/docs/user/demos/result-cache.html)
* [Consumer异步调用](http://dubbo.apache.org/zh-cn/docs/user/demos/async-call.html)
* [Provider异步调用](http://dubbo.apache.org/zh-cn/docs/user/demos/async-execute-on-provider.html)
* [参数回调](http://dubbo.apache.org/zh-cn/docs/user/demos/callback-parameter.html)
* [事件通知:远程方法调用之前、调用之后、执行异常](http://dubbo.apache.org/zh-cn/docs/user/demos/events-notify.html)