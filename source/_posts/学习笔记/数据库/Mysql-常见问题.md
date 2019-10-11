---
title: mysql常见问题
date: 2019-09-10
categories:
    - 学习
tags:
    - mysql
---

### 常见问题
* datetime 精度丢失的问题
```text
    mysql如果创建表示字段类型为datetime或者timestamp时，默认精度是到秒的。当你传入的数据包含毫秒时会自动的四舍五入也就是我们
说的精度丢失问题。如果想保存毫秒的精度需要指定精度。如datetime(1),datetime(2),datetime(3)

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


