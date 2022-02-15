---
title: Nacos
date: 2019-07-19
categories:
    - 学习
tags:
    - nacos
---

### Nacos Server

#### 启动nacos服务

* [下载源码或者安装包](https://github.com/alibaba/nacos/releases)
* 解压后进入nacos/bin目录
* 输入命令启动服务
  * Linux： sh startup.sh -m standalone
  * Windows：cmd startup.cmd
* nacos默认使用8848端口，可通过 **http://127.0.0.1:8848/nacos/index.html** 进入自带的控制台界面，默认用户名/密码是nacos/nacos

#### 配置集群

* 在nacos的解压目录conf下，有配置文件cluster.conf(若无则手动创建)，每行配置成ip:port。（配置3个或3个以上节点）
  
``` yml
#cluster.conf
192.168.0.1:8848
192.168.0.2:8848
192.168.0.3:8848
```

<!-- more -->

* 配置后在各个节点服务器输入命令启动所有服务：sh startup.sh

#### 配置Mysql

* 初始化nacos相关表：运行conf/nacos-mysql.sql文件
* 修改conf/application.properties文件，增加支持mysql数据源配置，添加mysql数据源的url、用户名和密码
  
``` yml
spring.datasource.platform=mysql
db.num=1
db.url.0=jdbc:mysql://127.0.0.1:3306/nacos?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true
db.user=user
db.password=password
```

* 配置后输入命令启动服务

### 整合nacos

* [Spring整合nacos](https://nacos.io/zh-cn/docs/quick-start-spring.html)
* [Spring Boot整合nacos](https://nacos.io/zh-cn/docs/quick-start-spring-boot.html)
* [Spring cloud整合nacos](https://nacos.io/zh-cn/docs/quick-start-spring-cloud.html)
