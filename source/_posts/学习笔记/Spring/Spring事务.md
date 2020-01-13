---
title: Spring事务
date: 2020-01-13
categories:
    - 学习
tags:
    - Spring
---

### 事务的隔离级别
* READ UNCOMMITTED (读未提交)
* READ COMMITTED (读已提交)
* REPEATABLE READ (可重复读)
* SERIALIZABLE (串行化)

### 事务的隔离级别与脏读、不可重复度、幻读
|事务隔离级别| 脏读 | 不可重复度 | 幻读 | 默认 |
|:----------:|:----:|:----------:|:----:|:----:|
| 读未提交   |  √  |      √    |  √  |      |
| 读已提交   |  ×  |      √    |  √  |Oracle|
| 可重复读   |  ×  |      ×    |  √  |Mysql |
| 串行化     |  ×  |      ×    |  ×  |      |

#### Spring 5种事务隔离级别
* **ISOLATION_DEFAULT** Spring将使用数据库中默认的事务隔离级别
* **ISOLATION_READ_UNCOMMITTED**
* **ISOLATION_READ_COMMITTED**
* **ISOLATION_REPEATABLE_READ**
* **ISOLATION_SERIALIZABLE**

#### Spring 7种事务传播机制
* REQUIRED
    * 如果当前方法有事务则加入事务，没有则创建一个事务
* NOT_SUPPORTED
    * 不支持事务，如果当前有事务则挂起事务运行
* REQUIREDS_NEW
    * 新建一个事务并在这个事务中运行，如果当前存在事务就把当前事务挂起。新建的事务的提交与回滚与挂起事务没有联系，不会影响挂起事务的操作
* MANDATORY
    * 强制当前方法使用事务运行，如果当前没有事务则抛出异常
* NEVER
    * 当前方法不能存在事务，即非事务状态运行，如果存在事务则抛出异常
* SUPPORTS
    * 支持当前事务，如果当前没事务也支持非事务状态运行
* NESTED
    * 如果当前存在事务，则在嵌套事务内执行。嵌套事务的提交与回滚与父事务没有任何关系，反之，当父事务提交嵌套事务也一起提交，父事务回滚会也回滚嵌套事务的。如果当前没有事务，则新建一个事务运行，这时候则与PROPAGATION_REQUIRED场景一致

### Spring的@Transactional 注解控制事务失效场景
* 数据库引擎不支持事务
    *  MyISAM 引擎是不支持事务操作的，InnoDB 才是支持事务的引擎（MySQL 5.5.5 开始的默认存储引擎是：InnoDB）
* 没有被Spring管理
```
// @Service
public class OrderServiceImpl implements OrderService {

    @Transactional
    public void updateOrder(Order order) {
        // update order
    }

}
```
* 方法不是public的
    * @Transactional 只能用于 public 的方法上，否则事务不会失效，如果要用在非 public 方法上，可以开启 AspectJ 代理模式
* 自身调用问题  
```
// 示例一
@Service
public class OrderServiceImpl implements OrderService {

    public void update(Order order) {
        updateOrder(order);
    }

    @Transactional
    public void updateOrder(Order order) {
        // update order
    }

}

// 示例二
@Service
public class OrderServiceImpl implements OrderService {

    @Transactional
    public void update(Order order) {
        updateOrder(order);
    }

    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void updateOrder(Order order) {
        // update order
    }

}

// 这两个示例事务均失效，默认只有外部调用的事务才会生效，因为需要经过Spring的代理类
```
* 数据源没有配置事务管理器了
```
@Bean
public PlatformTransactionManager transactionManager(DataSource dataSource) {
    return new DataSourceTransactionManager(dataSource);
}
```
* 不支持事务
    * Propagation.NOT_SUPPORTED： 表示不以事务运行，当前若存在事务则挂起
```
@Service
public class OrderServiceImpl implements OrderService {

    @Transactional
    public void update(Order order) {
        updateOrder(order);
    }

    @Transactional(propagation = Propagation.NOT_SUPPORTED)
    public void updateOrder(Order order) {
        // update order
    }

}
```
* 异常被吃掉
```
@Service
public class OrderServiceImpl implements OrderService {

    @Transactional
    public void updateOrder(Order order) {
        try {
            // update order
        } catch {

        }
    }

}
```
* 异常的类型错误
```
@Service
public class OrderServiceImpl implements OrderService {

    @Transactional
    public void updateOrder(Order order) {
        try {
            // update order
        } catch {
            throw new Exception("更新错误");
        }
    }

}

// 这样事务也是不生效的，因为默认回滚的是：RuntimeException，如果你想触发其他异常的回滚，需要在注解上配置一下，如: @Transactional(rollbackFor = Exception.class) 这个配置仅限于 Throwable 异常类及其子类
```
