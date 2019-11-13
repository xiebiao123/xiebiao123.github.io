---
title: jhat
date: 2019-10-25
categories:
    - 学习
tags:
    - java
---

### [jhat](https://www.jianshu.com/p/2c5ba79d8889)
#### 简介
jhat(Java Heap Analysis Tool),是一个用来分析java的堆情况的命令。是java虚拟机自带的一种虚拟机堆转储快照分析工具。

#### 测试代码
```
public class HeapOOM {

    static class OOMObject{
    }

    public static void main(String[] args) {
        List<OOMObject> list = new ArrayList<OOMObject>();

        while (true) {
            list.add(new OOMObject());
        }
    }
}
```

#### jhat分析代码
```
F:\dump>jhat java_pid15860.hprof
Reading from java_pid15860.hprof...
Dump file created Tue Jun 25 17:43:13 CST 2019
Snapshot read, resolving...
Resolving 819118 objects...
Chasing references, expect 163 dots.............................................
................................................................................
......................................
Eliminating duplicate references................................................
................................................................................
...................................
Snapshot resolved.
Started HTTP server on port 7000
Server is ready.
```

#### jhat分析结果
* 显示出堆中所包含的所有的类
* 堆实例的分布表
* [参考](https://www.cnblogs.com/baihuitestsoftware/articles/6406271.html)