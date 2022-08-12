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

### mysql区分大小写

* 系统参数

```sql
show global variables like '%lower_case%';
+------------------------+-------+
| Variable_name          | Value |
+------------------------+-------+
| lower_case_file_system | ON    |
| lower_case_table_names | 0     |
+------------------------+-------+
```
参数说明：
1.lower_case_file_system，代表当前系统文件是否大小写敏感，只读参数，无法修改。ON 大小写不敏感，OFF 大小写敏感。
> 此变量描述数据所在的操作系统的文件目录是否区分大小写。 OFF表示文件名区分大小写，ON表示它们不区分大小写。此变量是只读的，因为它反映了文件系统的属性，设置它对文件系统没有影响。

2.lower_case_table_names，代表表名是否大小写敏感，可以修改，参数有0、1、2三种。
* 0 大小写敏感。（Unix，Linux默认） 创建的库表将原样保存在磁盘上。如create database TeSt;将会创建一个TeSt的目录，create table AbCCC …将会原样生成AbCCC.frm文件，SQL语句也会原样解析。
* 1 大小写不敏感。（Windows默认） 创建的库表时，MySQL将所有的库表名转换成小写存储在磁盘上。 SQL语句同样会将库表名转换成小写。 如需要查询以前创建的Testtable（生成Testtable.frm文件），即便执行select * from Testtable，也会被转换成select * from testtable，致使报错表不存在。
* 2 大小写不敏感（OS X默认） 创建的库表将原样保存在磁盘上， 但SQL语句将库表名转换成小写。
> 在Linux系统中修改my.cnf文件，在Windows下修改my.ini文件，新增或修改 **lower_case_table_names = 0 或 lower_case_table_names = 1** 然后重启MySQL服务才可以生效。

* 字符校验

如果你在mysql有唯一约束的列上插入两行值'A'和'a',Mysql会认为它是相同的，而在oracle中就不会。就是mysql默认的字段值不区分大小写？这点是比较令人头痛的事。直接使用客户端用sql查询数据库。 发现的确是大小不敏感 。
* 可以设置**collate（校对）**来区分大小
  * *_bin: 表示的是binary case sensitive collation，也就是说是区分大小写的
  * *_cs: case sensitive collation，区分大小写
  * *_ci: case insensitive collation，不区分大小写
  
> 关于字符集与校验规则，mysql能：
> 1、使用字符集来存储字符串，支持多种字符集；
> 2、使用校验规则来比较字符串，同种字符集还能使用多种校验规则来比较；
> 3、在同一台服务器、同一个数据库或者甚至在同一个表中使用不同字符集或校对规则来混合组合字符串；
> 4、可以在任何级别（服务器、数据库、表、字段、字符串），定义不同的字符集和校验规则。

* binary关键字
* sql语句
```sql
select * from usertable where binary id='AAMkADExM2M5NjQ2LWUzYzctNDFkMC1h'; 
```
* 建表语句
```sql
create table `usertable`( 
 `id` varchar(32) binary, 
 PRIMARY KEY (`id`) 
) ENGINE=InnoDB DEFAULT CHARSET=utf8; 

或

CREATE TABLE `usertable` ( 
 `id` varchar(32) CHARACTER SET utf8 COLLATE utf8_bin NOT NULL DEFAULT '', 
 PRIMARY KEY (`id`) 
) ENGINE=InnoDB DEFAULT CHARSET=utf8; 
```