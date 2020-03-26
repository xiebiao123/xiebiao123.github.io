---
title: Window
date: 2020-01-14
categories:
    - 学习
tags:
    - Window
---

### [Windows10关闭占用某一端口号的进程](https://blog.csdn.net/eagleuniversityeye/article/details/79985027)
* 查看端口的使用情况
```
netstat -ano | findstr 8080(端口号)

  TCP    0.0.0.0:8080           0.0.0.0:0              LISTENING       3732
  TCP    10.204.254.11:49581    10.204.243.46:8080     ESTABLISHED     11624
  TCP    10.204.254.11:55550    183.3.235.67:8080      ESTABLISHED     21528
  TCP    10.204.254.11:63736    10.204.58.224:8080     ESTABLISHED     3732
  TCP    10.204.254.11:65290    10.204.58.223:8080     ESTABLISHED     3732
  TCP    [::]:8080              [::]:0                 LISTENING       3732
```

* 强制关闭指定进程号的进程
```
taskkill -PID 3732(进程号) -F

成功: 已终止 PID 为 3732 的进程。
```