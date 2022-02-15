---
title: Docker Desktop
date: 2022-02-09
categories:
    - 学习
tags:
    - docker
---

### Docker Desktop 安装

### Docker Desktop安装软件

#### [挂载mysql](https://blog.csdn.net/yaoyuncn/article/details/103914588) 

```shell
# 1. 安装镜像
docker pull mysql

# 2. 创建目录

# 3. 获取默认的mysql配置
docker run --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -d mysql:latest --default-authentication-plugin=mysql_native_password

# 4. 拷贝文件到本地目录
docker cp mysql:/etc/mysql/my.cnf D:/docker/mysql/conf/

#5.修改配置文件

# 表名大小写是否敏感
lower_case_table_names=1
# 是否开启慢查询日志
slow_query_log=ON
# 慢查询的阈值，单位秒
long_query_time=3
# 是否记录未使用索引的查询语句，记录在慢查询日志
log_queries_not_using_indexes=OFF
# 错误日志
log_error=/var/lib/mysql/error.log
# 慢查询日志
slow_query_log_file=var/lib/mysql/slowquery.log
# 允许导入导出
secure-file-priv=''

# 6. 重新创建容器
docker stop mysql
docker rm mysql

docker run --name mysql -p 3306:3306 -v /d/docker/mysql/data:/var/lib/mysql/ -v /d/docker/mysql/conf/my.cnf:/etc/mysql/my.cnf -e MYSQL_ROOT_PASSWORD=123456 -d mysql:latest --default-authentication-plugin=mysql_native_password
```



