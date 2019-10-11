---
title: MyBatis
date: 2018-08-22
categories:
    - 学习
tags:
    - MyBatis
---
### 简介
* 是一个半自动化持久层框架
* 编写sql -> 预编译 -> 设置参数 -> 执行sql -> 封装结果

#### 组件
* Executor(update,query,flushStatements,commit,rollback,getTransaction,close,isClosed)
* ParameterHandler(getParameterObject,s  etParameters)
* ResultSetHandler(handleResultSets,handleOutputParameters)
* StatementHandler(prepare,parameter,batch,update,query)
<!-- more -->

#### #{} 和 ${}
 
* #{}: 是以预编译的形式，将参数设置到sql语句中；preparedStatement；防止sql注入
* ${}：取出的值直接拼装在sql语句中国；会有安全问题，（常用语分表或者动态排序）

#### 其他
* @MapKey  返回map
* 级联属性
* association （1-1）
    * 关联属性
    * 分步查询
    * 延迟加载（按需加载） 使用的时候再加载
* collection  （1-n）
    * 定义集合（嵌套结果集）
    * 分步查询
    * 延迟加载（按需加载） 使用的时候再加载
* 鉴别器 discriminator  根据某列的值改变封装的行为

```
# 分步查询多值传递
column = "{key1=value1,key2=value2}"
# 局部懒加载控制开关
fetchType = lazy/eager  懒加载/立即加载
```

* 两个内置参数
    * _parameter: 代表整个参数
        * 单个参数_parameter就是这个参数
        * 多个参数会被封装成一个map,_parameter就代表这个map
    * _databaseId: 如果配置了databaseIdProvider标签，_databaseId就代表当前数据库的别名
 
* bind 绑定OGNL表达式的值到一个变量中

### mybatis两级缓存
* 一级缓存
    * 与数据库同一次会话期间查询到的数据会放在本地缓存中，如果以后获取相同的数据，直接从缓存中获取
    * sqlSession级别的缓存（其实是一个map），一级缓存是一直开启的
    * 缓存失效条件
        * 不同的sqlSession
        * 相同的sqlSession，不同的查询条件
        * 相同的SqlSession，两次查询之间有增删改操作
        * 相同的SqlSession，手动清除一级缓存
* 二级缓存 
    * 全局缓存，基于namespace级别的缓存，一个namespace对应一个二级缓存
    * 工作机制
        * 一个会话查询一条数据，这个数据会被放在一级缓存中
        * 如果**会话关闭**，一级缓存中的数据才会被保存到二级缓存中，新的会话可以参照二级缓存
        * 不同的namespace查询的数据会放在自己对应的缓存map中
     * 使用
        * 开启全局二级缓存配置<setting name="cacheEnabled" value="true">
        * 去mapper中配置使用<cache eviction="FIFO" flushInterval="60000" readOnly="false" siez= "1024" type=""></cache>
            * eviction 缓存的回收策略
                * LRU 最近最少使用的（默认）
                * FIFO 先进先出
                * SOFT 软引用
                * WEAK 弱引用
            * flushInterval 缓存的多长时间清空一次，默认不清空，可以设置一个毫秒值
            * size 缓存存放多少元素
            * type 指定自定义缓存的全类名，实现Cache接口即可
        * POJO需要实现序列化接口

 > 注意： 和缓存有关的设置/熟悉
 > 1. cacheEnable=true  (false:关闭二级缓存，一级缓依旧可用)
 > 2. 每个select标签都有useCache=true  (false:关闭二级缓存，一级缓依旧可用)
 > 3. 每个增删改查标签都有flushCache=true (一二级缓存都会清空)
 > 4. sqlSession.clearCache()  一级缓存被清空，二级缓存依旧可用
 > 5. localCacheScope 本地缓存作用域 （session一级缓存，statement禁用一级缓存）

### 原理解析
* 获取sqlSessionFactory对象
    * 解析文件的每一个信息保存在configuration中，返回包含Configuration的DefaultSqlSessionFactory
    * 注意【mappedStatement】代表一个增删改查的详细信息
* 获取SqlSession对象
    * 返回一个DefaultSqlSession对象，包含Executor和Configuration
    * 这一步会创建Executor对象
* 获取接口的代理对象（MapperProxy）
    * getMapper,使用MapperProxyFactory创建一个MapperProxy的代理对象，代理对象里面包含了defaultSqlSession(包含Executor)
* 执行增删改查方法

### 源码分析
```
mapper接口代理对象 -> defaultSqlSession -> Executor -> statementHandler 
->ParameterHandler -> TypeHandler
->ResultSetHandler -> TypeHandler

* statementHandler 处理sql预编译，设置参数等相关工作
* parameterHandler 设置编译参数
* resultSetHandler 处理结果集
* typeHandler 在整个过程中，进行数据库类型和javaBean类型的映射
* 底层原生 JDBC: statement ; PreparedStatement 
```
1. 根据配置文件（全局配置，sql映射）初始化出configuration对象
2. 创建一个DefaultSqlSession对象，它里面包含configuration和executor（根据全局配置文件中的defaultExecutorType创建出对应的Executor）
3. DefaultSqlSession.getMapper()，拿到mapper接口对应的MapperProxy（动态代理对象）
4. MapperProxy里面有DefaultSqlSession
5. 执行增删改查方法
    1. 调用DefaultSqlSession的增删改查（即Executor的增删改查）
    2. 会创建一个StatementHandler对象，同时也会创建ParameterHandler和ResultSetHandler
    3. 调用StatementHandler预编译SQL,ParameterHandlers设置参数
    4. 调用StatementHandler的增删改查方法（底层调用原生JDBC）
    5. ResultSetHandler封装结果

### mybatis 插件
* 实现Interceptor接口
* 指定拦截的方法@Intercepts({@Signature})
* 将写好的插件注册到全局配置文件中

> 注意：多个插件创建动态代理的时候，是按照插件配置顺序创建层层代理，执行目标方法之后，按照逆向顺序执行

### mysql 批量执行器
* ExecutorType.batch , 指定sqlSession类型
* 与Spring整合， 指定sqlSessionTemplate bean

### mybatis 使用存储过程
1. 在select 标签中设置  statementType = callable 表示调用存储过程
2. {call 存储过程}

### mybatis 自定义类型处理器
1. 实现TypeHandler接口
2. 全局配置自定义的typeHandler /  也可以处理某个字段的时候，指定类型处理器