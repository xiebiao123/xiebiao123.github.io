---
title: Linux常见命令
date: 2019-10-25
categories:
    - 学习
tags:
    - Linux
---

### whereis、which、find、locate区别和用法

* find
  
``` shell
* find <指定名录> <指定条件> <指定动作>
* <指定名录>：所要搜索的目录及其所有子目录。默认为当前目录
* <指定条件>：所要搜索的文件的特征
* <指定动作>：对搜索结果进行特定的处理

$ find . -name "my*"
搜索当前目录（含子目录，以下同）中，所有文件名以my开头的文件

$ find . -name "my*" -ls
搜索当前目录中，所有文件名以my开头的文件，并显示它们的详细信息

$ find . -type f -mmin -10
搜索当前目录中，所有过去10分钟中更新过的普通文件。如果不加-type f参数，则搜索普通文件+特殊文件+目录
```

* locate
  
``` shell
locate命令其实是“find -name”的另一种写法，但是要比后者快得多，原因在于它不搜索具体目录，而是搜索一个数据库（/var/lib/
locatedb），这个数据库中含有本地所有文件信息。Linux系统自动创建这个数据库，并且每天自动更新一次，所以使用locate命令查不
到最新变动过的文件。为了避免这种情况，可以在使用locate之前，先使用 updatedb 命令，手动更新数据库

$ locate /etc/sh
搜索etc目录下所有以sh开头的文件

$ locate ~/m
搜索用户主目录下，所有以m开头的文件

$ locate -i ~/m
搜索用户主目录下，所有以m开头的文件，并且忽略大小写
```

* whereis
  
``` shell
whereis命令只能用于程序名的搜索，而且只搜索二进制文件（参数-b）、man说明文件（参数-m）和源代码文件（参数-s）。如果省略参数
，则返回所有信息。

$ whereis grep
$ whereis java
```

* which
  
``` shell
which命令的作用是，在PATH变量指定的路径中，搜索某个系统命令的位置，并且返回第一个搜索结果。也就是说，使用which命令，就可
以看到某个系统命令是否存在，以及执行的到底是哪一个位置的命令

which java
```

### 去重&排序

* uniq
  
``` shell
Linux uniq 命令用于检查及删除文本文件中重复出现的行列，一般与 sort 命令结合使用，uniq 可检查文本文件中重复出现的行列
```

* sort
  
``` shell
Linux sort命令用于将文本文件内容加以排序，sort可针对文本文件的内容，以行为单位来排序。
```
