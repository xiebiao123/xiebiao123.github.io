---
title: Docker
date: 2018-08-04
categories:
    - 学习
tags:
    - 笔记
    - Docker
---
# 概念
* **镜像(Images)** 镜像是用于创建 Docker 容器的模板
* **容器(Container)** 容器是独立运行的一个或一组应用
* **客户端(Client)** Docker 客户端通过命令行或者其他工具使用 [Docker API](https://docs.docker.com/reference/api/docker_remote_api) 与 Docker 的守护进程通信
* **主机(Host)** 一个物理或者虚拟的机器用于执行 Docker 守护进程和容器
* **仓库(Registry)** Docker 仓库用来保存镜像，可以理解为代码控制中的代码仓库,[Docker Hub](https://hub.docker.com) 提供了庞大的镜像集合供使用
* **Machine** Docker Machine是一个简化Docker安装的命令行工具，通过一个简单的命令行即可在相应的平台上安装Docker，比如VirtualBox、 Digital Ocean、Microsoft Azure

# 命令

## 常用
    service docker start 启动 Docker 进程
    docker command \-\-help Docker命令使用方法

## 容器

### docker run
    语法
        docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
    OPTIONS说明
        -d: 后台运行容器，并返回容器ID
        -i: 以交互模式运行容器，通常与 -t 同时使用
        -t: 为容器重新分配一个伪输入终端，通常与 -i 同时使用
        -p: 端口映射，格式为：主机(宿主)端口:容器端口
        --name="nginx-lb": 为容器指定一个名称
    示例
        以后台模式启动一个容器,并将容器命名为mynginx
        docker run --name mynginx -d nginx:latest
        以后台模式启动一个容器,将容器的 80 端口映射到主机的 80 端口,主机的目录 /data 映射到容器的 /data。
        docker run -p 80:80 -v /data:/data -d nginx:latest
        使用镜像nginx:latest以交互模式启动一个容器,在容器内执行/bin/bash命令
        docker run -it nginx:latest /bin/bash

### docker start/stop/restart
    语法
        启动一个或多个已经被停止的容器
        docker start [OPTIONS] CONTAINER [CONTAINER...]
        停止一个或多个已经被停止的容器
        docker stop [OPTIONS] CONTAINER [CONTAINER...]
        重启一个或多个已经被停止的容器
        docker restart [OPTIONS] CONTAINER [CONTAINER...]
    示例
        docker start myrunoob
        docker stop myrunoob
        docker restart myrunoob

### docker rm
    语法
        docker rm [OPTIONS] CONTAINER [CONTAINER...]
    OPTIONS说明
        -f :通过SIGKILL信号强制删除一个运行中的容器
        -l :移除容器间的网络连接，而非容器本身
        -v :-v 删除与容器关联的卷
    示例
        强制删除容器db01、db02
        docker rm -f db01 db02
        移除容器nginx01对容器db01的连接，连接名db
        docker rm -l db
        删除容器nginx01,并删除容器挂载的数据卷
        docker rm -v nginx01

### docker exec
    语法
        docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
    OPTIONS说明
        -d :分离模式: 在后台运行
        -i :即使没有附加也保持STDIN 打开
        -t :分配一个伪终端
    示例
        容器mynginx中开启一个交互模式的终端
        docker exec -it  mynginx /bin/bash


### docker logs
    语法
        docker logs [OPTIONS] CONTAINER
    OPTIONS说明
        -f : 跟踪日志输出
        --since :显示某个开始时间的所有日志
        -t : 显示时间戳
        --tail :仅列出最新N条容器日志
    示例
        跟踪查看容器mynginx的日志输出
        docker logs -f mynginx
        查看容器mynginx从2018年7月1日后的最新10条日志
        docker logs --since="2018-07-01" --tail=10 mynginx

### docker ps/port/cp/diff/export
    docker ps 列出容器
    docker port 列出指定的容器的端口映射
    docker cp 用于容器与主机之间的数据拷贝
    docker diff 检查容器里文件结构的更改
    docker export 将文件系统作为一个tar归档文件导出到STDOUT



## 镜像
### docker build
    语法
        docker build [OPTIONS] PATH | URL | -
    OPTIONS说明
    --build-arg=[] :设置镜像创建时的变量
    -f :指定要使用的Dockerfile路径
    -m :设置内存最大值
    --tag, -t: 镜像的名字及标签，通常 name:tag 或者 name 格式；可以在一次构建中为一个镜像设置多个标签
    示例
        使用当前目录的 Dockerfile 创建镜像，标签为 runoob/ubuntu:v1
        ocker build -t runoob/ubuntu:v1 .
        使用URL github.com/creack/docker-firefox 的 Dockerfile 创建镜像
        docker build github.com/creack/docker-firefox
        使用-f 指定Dockerfile 文件的位置 创建镜像
        docker build -f /path/to/a/Dockerfile .

### docker images/pull/rmi/tag/search
    docker images 列出本地镜像
    docker pull 拉取镜像
    docker rmi 删除本地一个或多少镜像
    docker tag 标记本地镜像
    docker search 查找镜像

# Dockerfile详解

1.[Dockerfile](https://blog.csdn.net/u010884123/article/details/55213279)
2.[Dockerfile命令详解](https://www.cnblogs.com/dazhoushuoceshi/p/7066041.html)

