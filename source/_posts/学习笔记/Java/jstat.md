---
title: jstat
date: 2019-10-25
categories:
    - 学习
tags:
    - java
---

### [jhat](https://www.jianshu.com/p/9f41d4a42c33)
#### 简介
Jstat是JDK自带的一个轻量级小工具。全称“Java Virtual Machine statistics monitoring tool”，它位于java的bin目录下，主要利
用JVM内建的指令对Java应用程序的资源和性能进行实时的命令行的监控，包括了对Heap size和垃圾回收状况的监控

#### 命令详解
```
/opt/java8/bin/jstat -h

-h requires an integer argument
Usage: jstat -help|-options
       jstat -<option> [-t] [-h<lines>] <vmid> [<interval> [<count>]]

Definitions:
  <option>      An option reported by the -options option
  <vmid>        Virtual Machine Identifier. A vmid takes the following form:
                     <lvmid>[@<hostname>[:<port>]]
                Where <lvmid> is the local vm identifier for the target
                Java virtual machine, typically a process id; <hostname> is
                the name of the host running the target Java virtual machine;
                and <port> is the port number for the rmiregistry on the
                target host. See the jvmstat documentation for a more complete
                description of the Virtual Machine Identifier.
  <lines>       Number of samples between header lines.
  <interval>    Sampling interval. The following forms are allowed:
                    <n>["ms"|"s"]
                Where <n> is an integer and the suffix specifies the units as 
                milliseconds("ms") or seconds("s"). The default units are "ms".
  <count>       Number of samples to take before terminating.
  -J<flag>      Pass <flag> directly to the runtime system.
```

* option 参数选项
* -t 可以在打印的列加上Timestamp列，用于显示系统运行的时间
* -h 可以在周期性数据数据的时候，可以在指定输出多少行以后输出一次表头
* vmid Virtual Machine ID（ 进程的 pid）
* interval 执行每次的间隔时间，单位为毫秒
* count 用于指定输出多少次记录，缺省则会一直打印

#### jstat options参数
```
/opt/java8/bin/jstat -options
-class
-compiler
-gc
-gccapacity
-gccause
-gcmetacapacity
-gcnew
-gcnewcapacity
-gcold
-gcoldcapacity
-gcutil


```

* -class 显示ClassLoad的相关信息
* -compiler 显示JIT编译的相关信息
* -gc 显示和gc相关的堆信息
* -gccapacity 显示各个代的容量以及使用情况
* -gcmetacapacity 显示metaspace的大小
* -gcnew 显示新生代信息
* -gcnewcapacity 显示新生代大小和使用情况
* -gcold 显示老年代和永久代的信息
* -gcoldcapacity 显示老年代的大小
* -gcutil 显示垃圾收集信息
* -gccause 显示垃圾回收的相关信息（通-gcutil）,同时显示最后一次或当前正在发生的垃圾回收的诱因
* -printcompilation 输出JIT编译的方法信息

#### 举例
```
/opt/java8/bin/jstat -class 28367
Loaded  Bytes  Unloaded  Bytes     Time   
 10046 18032.6        0     0.0       7.34
 
* Loaded : 已经装载的类的数量
* Bytes : 装载类所占用的字节数
* Unloaded：已经卸载类的数量
* Bytes：卸载类的字节数
* Time：装载和卸载类所花费的时间

/opt/java8/bin/jstat -compiler 28367
Compiled Failed Invalid   Time   FailedType FailedMethod
   12036      1       0    52.31          1 org/apache/skywalking/apm/dependencies/net/bytebuddy/jar/asm/ClassReader readMethod
   
Compiled：编译任务执行数量
Failed：编译任务执行失败数量
Invalid ：编译任务执行失效数量
Time ：编译任务消耗时间
FailedType：最后一个编译失败任务的类型
FailedMethod：最后一个编译失败任务所在的类及方法
```