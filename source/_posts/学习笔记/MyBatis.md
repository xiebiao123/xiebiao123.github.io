---
title: MyBatis
date: 2018-07-19
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
* @MapKey
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