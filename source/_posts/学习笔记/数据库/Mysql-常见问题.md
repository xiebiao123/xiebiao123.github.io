---
title: mysql常见问题
date: 2019-09-10
categories:
    - 学习
tags:
    - mysql
---

### 常见问题

* datetime mysql精度丢失的问题
  
```sql
    mysql如果创建表示字段类型为datetime或者timestamp时，默认精度是到秒的。当你传入的数据包含毫秒时会自动的四舍五入也就是我们说的精度丢失问题。如果想保存毫秒的精度需要指定精度。如datetime(1),datetime(2),datetime(3)

create table test_date (
    id bigint(20) ,
    datetime0  datetime,
    datetime1 datetime(1),
    datetime2 datetime(2),
    datetime3 datetime(3)
)ENGINE=INNODB DEFAULT CHARSET=utf8;

insert into test_date values (1,'2019-02-22 13:02:58.789','2019-02-22 13:02:58.789','2019-02-22 13:02:58.789','2019-02-22 13:02:58.789');
执行结果为：
id  datetime0               datetime1                   datetime2                   datetime3
1   2019-02-22 13:02:59     2019-02-22 13:02:58.8       2019-02-22 13:02:58.79      2019-02-22 13:02:58.789
```

<!-- more -->

* FIND_IN_SET mysql查询匹配包含字符串
  
``` sql
1.正确的方式：
判断字段field_A中是否包含23:

select * from table_test where FIND_IN_SET("23", field_A) ;
2.错误的方式：
select * form table_test where field_A like "%23%"
```

* GROUP_CONCAT mysql连接字符串
  
``` sql
group_concat([DISTINCT] 要连接的字段 [Order BY ASC/DESC 排序字段] [Separator '分隔符'])
```

* [数据库锁表](https://www.jianshu.com/p/aa99df051c8f)
  
``` sql
-- 查询是否锁表

show OPEN TABLES ;

-- 查询进程

show processlist ;

-- 查询到相对应的进程，然后杀死进程

kill id; -- 一般到这一步就解锁了

-- 查看正在锁的事务

SELECT * FROM INFORMATION_SCHEMA.INNODB_LOCKS;

-- 查看等待锁的事务

SELECT * FROM INFORMATION_SCHEMA.INNODB_LOCK_WAITS;

-- 解锁表

UNLOCK TABLES;
```

* [**mysql** EXPLAIN用法和结果分析](https://blog.csdn.net/why15732625998/article/details/80388236)
  