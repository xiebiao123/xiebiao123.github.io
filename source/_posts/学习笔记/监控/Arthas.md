---
title: Arthas
date: 2022-09-01
categories:
    - 学习
tags:
    - Arthas
    - 监控
---

### 什么是Arthas

Arthas 是Alibaba开源的Java诊断工具，深受开发者喜爱。当你遇到以下类似问题而束手无策时，Arthas可以帮助你解决：

1. 这个类从哪个 jar 包加载的？为什么会报各种类相关的 Exception？
2. 我改的代码为什么没有执行到？难道是我没 commit？分支搞错了？（jad 反编译代码）
3. 遇到问题无法在线上 debug，难道只能通过加日志再重新发布吗？（动态修改日志的级别）
4. 线上遇到某个用户的数据处理有问题，但线上同样无法 debug，线下无法重现！
5. 是否有一个全局视角来查看系统的运行状况？(trace 查看方法的调用链和执行时间)
6. 有什么办法可以监控到JVM的实时运行状态？
7. 怎么快速定位应用的热点，生成火焰图？
8. 怎样直接从JVM内查找某个类的实例？

### 快速安装

* 使用arthas-boot（推荐）

```shell
# 下载arthas-boot.jar，然后用java -jar的方式启动：
curl -O https://arthas.aliyun.com/arthas-boot.jar
java -jar arthas-boot.jar

# 打印帮助信息：
java -jar arthas-boot.jar -h
```

* 使用as.sh
  
```shell
curl -L https://arthas.aliyun.com/install.sh | sh

```

### 后台异步任务

#### 1. 使用&在后台执行任务
```shell
$ trace Test t &
```

#### 2. 通过 jobs 查看任务
```shell
$ jobs
[10]*
       Stopped           watch com.taobao.container.Test test "params[0].{? #this.name == null }" -x 2
       execution count : 19
       start time      : Fri Sep 22 09:59:55 CST 2017
       timeout date    : Sat Sep 23 09:59:55 CST 2017
       session         : 3648e874-5e69-473f-9eed-7f89660b079b (current)
```

* job id 是 10, * 表示此 job 是当前 session 创建
* 状态是 Stopped
* execution count 是执行次数，从启动开始已经执行了 19 次
* timeout date 是超时的时间，到这个时间，任务将会自动超时退出

#### 3.任务暂停和取消
当任务正在前台执行，比如直接调用命令trace Test t或者调用后台执行命令trace Test t &后又通过fg命令将任务转到前台。这时 console 中无法继续执行命令，但是可以接收并处理以下事件：

* ‘ctrl + z’：将任务暂停。通过jbos查看任务状态将会变为 Stopped，通过bg <job-id>或者fg <job-id>可让任务重新开始执行
* ‘ctrl + c’：停止任务
* ‘ctrl + d’：按照 linux 语义应当是退出终端，目前 arthas 中是空实现，不处理

#### 4. fg、bg 命令，将命令转到前台、后台继续执行
* 任务在后台执行或者暂停状态（ctrl + z暂停任务）时，执行fg <job-id>将可以把对应的任务转到前台继续执行。在前台执行时，无法在 console 中执行其他命令
* 当任务处于暂停状态时（ctrl + z暂停任务），执行bg <job-id>将可以把对应的任务在后台继续执行
* 非当前 session 创建的 job，只能由当前 session fg 到前台执行

#### 5. 任务输出重定向
可通过>或者>>将任务输出结果输出到指定的文件中，可以和&一起使用，实现 arthas 命令的后台异步任务。

```shell
$ trace Test t >> test.out &
```

这时 trace 命令会在后台执行，并且把结果输出到~/logs/arthas-cache/test.out。可继续执行其他命令。并可查看文件中的命令执行结果。

当连接到远程的 arthas server 时，可能无法查看远程机器的文件，arthas 同时支持了自动重定向到本地缓存路径。使用方法如下：

``` shell
$ trace Test t >>  &
job id  : 2
cache location  : /Users/gehui/logs/arthas-cache/28198/2
```

可以看到并没有指定重定向文件位置，arthas 自动重定向到缓存中了，执行命令后会输出 job id 和 cache location。cache location 就是重定向文件的路径，在系统 logs 目录下，路径包括 pid 和 job id，避免和其他任务冲突。命令输出结果到/Users/gehui/logs/arthas-cache/28198/2中，job id 为 2。

#### 6. 停止命令

异步执行的命令，如果希望停止，可执行kill <job-id>

#### 7. 其他

* 最多同时支持 8 个命令使用重定向将结果写日志
* 请勿同时开启过多的后台异步命令，以免对目标 JVM 性能造成影响
* 如果不想停止 arthas，继续执行后台任务，可以执行 quit 退出 arthas 控制台（stop 会停止 arthas 服务）

### 批处理功能
#### 1. 创建批处理脚本
这里我们新建了一个test.as脚本，为了规范，我们采用了.as 后缀名，但事实上任意的文本文件都 ok。
> * 目前需要每个命令占一行
> * dashboard 务必指定执行次数(-n)，否则会导致批处理脚本无法终止
> * watch/tt/trace/monitor/stack 等命令务必指定执行次数(-n)，否则会导致批处理脚本无法终止
> * 可以使用异步后台任务，如 watch c.t.X test returnObj > &，让命令一直在后台运行，通过日志获取结果

#### 2. 运行你的批处理脚本
```shell
# 通过-f执行脚本文件， 批处理脚本默认会输出到标准输出中，可以将结果重定向到文件中。
$ ./as.sh -f /var/tmp/test.as <pid> > test.out

# 也可以通过 -c 来指定指行的命令，比如
$ ./as.sh -c 'sysprop; thread' <pid> > test.out 
```
> pid 可以通过 jps 命令查看

### [常用命令](https://arthas.aliyun.com/doc/commands.html)

#### jvm相关
* [dashboard](https://arthas.aliyun.com/doc/dashboard.html) 当前实例的数据面板
* [heapdump](https://arthas.aliyun.com/doc/heapdump.html) 类似于jmap命令的 heap dump功能

```shell
# dump 到指定文件
$ heapdump /tmp/dump.hprof

# 只 dump live 对象
$ heapdump --live /tmp/dump.hprof
```

* [jvm](https://arthas.gitee.io/doc/jvm.html) 查看当前JVM信息

```shell
$ jvm

···
--------------------------------------------------------------------------------------------------------------
 THREAD
--------------------------------------------------------------------------------------------------------------
 COUNT                          30      #  JVM 当前活跃的线程数
 DAEMON-COUNT                   24      #  JVM 当前活跃的守护线程数
 PEAK-COUNT                     31      #  从 JVM 启动开始曾经活着的最大线程数
 STARTED-COUNT                  36      #  从 JVM 启动开始总共启动过的线程次数
 DEADLOCK-COUNT                 0       #  JVM 当前死锁的线程数

--------------------------------------------------------------------------------------------------------------
 FILE-DESCRIPTOR
--------------------------------------------------------------------------------------------------------------
 MAX-FILE-DESCRIPTOR-COUNT      1048576     # JVM 进程最大可以打开的文件描述符数
 OPEN-FILE-DESCRIPTOR-COUNT     100         # JVM 当前打开的文件描述符数
Affect(row-cnt:0) cost in 88 ms.

```

* [logger](https://arthas.gitee.io/doc/logger.html) 查看logger信息，更新logger level
* [memory](https://arthas.gitee.io/doc/memory.html) 查看jvm内存信息
* [thread](https://arthas.gitee.io/doc/thread.html) 查看当前线程的信息，查看线程的堆栈

```shell
# thread -n x, 当前最忙的前 X 个线程并打印堆栈
$ thread -n 3

"C1 CompilerThread0" [Internal] cpuUsage=1.63% deltaTime=3ms time=1170ms


"arthas-command-execute" Id=23 cpuUsage=0.11% deltaTime=0ms time=401ms RUNNABLE
    at java.management@11.0.7/sun.management.ThreadImpl.dumpThreads0(Native Method)
    at java.management@11.0.7/sun.management.ThreadImpl.getThreadInfo(ThreadImpl.java:466)
    at com.taobao.arthas.core.command.monitor200.ThreadCommand.processTopBusyThreads(ThreadCommand.java:199)
    at com.taobao.arthas.core.command.monitor200.ThreadCommand.process(ThreadCommand.java:122)
    at com.taobao.arthas.core.shell.command.impl.AnnotatedCommandImpl.process(AnnotatedCommandImpl.java:82)
    at com.taobao.arthas.core.shell.command.impl.AnnotatedCommandImpl.access$100(AnnotatedCommandImpl.java:18)
    at com.taobao.arthas.core.shell.command.impl.AnnotatedCommandImpl$ProcessHandler.handle(AnnotatedCommandImpl.java:111)
    at com.taobao.arthas.core.shell.command.impl.AnnotatedCommandImpl$ProcessHandler.handle(AnnotatedCommandImpl.java:108)
    at com.taobao.arthas.core.shell.system.impl.ProcessImpl$CommandProcessTask.run(ProcessImpl.java:385)
    at java.base@11.0.7/java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:515)
    at java.base@11.0.7/java.util.concurrent.FutureTask.run(FutureTask.java:264)
    at java.base@11.0.7/java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecutor.java:304)
    at java.base@11.0.7/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1128)
    at java.base@11.0.7/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:628)
    at java.base@11.0.7/java.lang.Thread.run(Thread.java:834)


"VM Periodic Task Thread" [Internal] cpuUsage=0.07% deltaTime=0ms time=584ms
```

```shell
# thread id, 显示指定线程的运行堆栈
$ thread 1
"main" Id=1 WAITING on java.util.concurrent.CountDownLatch$Sync@29fafb28
    at sun.misc.Unsafe.park(Native Method)
    -  waiting on java.util.concurrent.CountDownLatch$Sync@29fafb28
    at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
    at java.util.concurrent.locks.AbstractQueuedSynchronizer.parkAndCheckInterrupt(AbstractQueuedSynchronizer.java:836)
    at java.util.concurrent.locks.AbstractQueuedSynchronizer.doAcquireSharedInterruptibly(AbstractQueuedSynchronizer.java:997)
    at java.util.concurrent.locks.AbstractQueuedSynchronizer.acquireSharedInterruptibly(AbstractQueuedSynchronizer.java:1304)
    at java.util.concurrent.CountDownLatch.await(CountDownLatch.java:231)
```

```shell
# thread -b, 找出当前阻塞其他线程的线程
$ thread -b
"http-bio-8080-exec-4" Id=27 TIMED_WAITING
    at java.lang.Thread.sleep(Native Method)
    at test.arthas.TestThreadBlocking.doGet(TestThreadBlocking.java:22)
    -  locked java.lang.Object@725be470 <---- but blocks 4 other threads!
    at javax.servlet.http.HttpServlet.service(HttpServlet.java:624)
    at javax.servlet.http.HttpServlet.service(HttpServlet.java:731)
    at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:303)
    at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:208)
    at org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:52)
    at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:241)
    at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:208)
    at test.filter.TestDurexFilter.doFilter(TestDurexFilter.java:46)
    at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:241)
    at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:208)
    at org.apache.catalina.core.StandardWrapperValve.invoke(StandardWrapperValve.java:220)
    at org.apache.catalina.core.StandardContextValve.invoke(StandardContextValve.java:122)
    at org.apache.catalina.authenticator.AuthenticatorBase.invoke(AuthenticatorBase.java:505)
    at com.taobao.tomcat.valves.ContextLoadFilterValve$FilterChainAdapter.doFilter(ContextLoadFilterValve.java:191)
    at com.taobao.eagleeye.EagleEyeFilter.doFilter(EagleEyeFilter.java:81)
    at com.taobao.tomcat.valves.ContextLoadFilterValve.invoke(ContextLoadFilterValve.java:150)
    at org.apache.catalina.core.StandardHostValve.invoke(StandardHostValve.java:170)
    at org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:103)
    at org.apache.catalina.core.StandardEngineValve.invoke(StandardEngineValve.java:116)
    at org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:429)
    at org.apache.coyote.http11.AbstractHttp11Processor.process(AbstractHttp11Processor.java:1085)
    at org.apache.coyote.AbstractProtocol$AbstractConnectionHandler.process(AbstractProtocol.java:625)
    at org.apache.tomcat.util.net.JIoEndpoint$SocketProcessor.run(JIoEndpoint.java:318)
    -  locked org.apache.tomcat.util.net.SocketWrapper@7127ee12
    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
    at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
    at java.lang.Thread.run(Thread.java:745)

    Number of locked synchronizers = 1
    - java.util.concurrent.ThreadPoolExecutor$Worker@31a6493e
```
> 注意， 目前只支持找出 synchronized 关键字阻塞住的线程， 如果是java.util.concurrent.Lock， 目前还不支持。

```shell
# thread --state ，查看指定状态的线程
$ thread --state WAITING
Threads Total: 16, NEW: 0, RUNNABLE: 9, BLOCKED: 0, WAITING: 3, TIMED_WAITING: 4, TERMINATED: 0
ID   NAME                           GROUP           PRIORITY   STATE     %CPU      DELTA_TIME TIME      INTERRUPTE DAEMON
3    Finalizer                      system          8          WAITING   0.0       0.000      0:0.000   false      true
20   arthas-UserStat                system          9          WAITING   0.0       0.000      0:0.001   false      true
14   arthas-timer                   system          9          WAITING   0.0       0.000      0:0.000   false      true
```

#### class/classloader相关
* [jad](https://arthas.gitee.io/doc/jad.html) 反编译已加载类的源码

```shell
# 反编译时只显示源代码,默认情况下，反编译结果里会带有ClassLoader信息，通过--source-only选项，可以只打印源代码。方便和mc/retransform命令结合使用。

$ jad --source-only demo.MathGame

/*
 * Decompiled with CFR 0_132.
 */
package demo;

import java.io.PrintStream;
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;
import java.util.Random;
import java.util.concurrent.TimeUnit;

public class MathGame {
    private static Random random = new Random();
    public int illegalArgumentCount = 0;
    ...
}
```

```shell
# 反编译指定的函数
$ jad demo.MathGame main

ClassLoader:
+-sun.misc.Launcher$AppClassLoader@232204a1
  +-sun.misc.Launcher$ExtClassLoader@7f31245a

Location:
/private/tmp/math-game.jar

       public static void main(String[] args) throws InterruptedException {
           MathGame game = new MathGame();
           while (true) {
/*16*/         game.run();
/*17*/         TimeUnit.SECONDS.sleep(1L);
           }
       }
```

* [mc](https://arthas.gitee.io/doc/mc.html) 内存编译器，编译.java文件生成.class文件
```shell
# Memory Compiler/内存编译器，编译.java文件生成.class。
$ mc /tmp/Test.java

# 可以通过-c参数指定 classloader：
$ mc -c 327a647b /tmp/Test.java

# 也可以通过--classLoaderClass参数指定 ClassLoader，可以通过-d命令指定输出目录：
$ mc --classLoaderClass org.springframework.boot.loader.LaunchedURLClassLoader /tmp/UserController.java -d /tmp
Memory compiler output:
/tmp/com/example/demo/arthas/user/UserController.class
```

* [retransform](https://arthas.gitee.io/doc/retransform.html) 加载外部的.class文件，retransform jvm 已加载的类。
```shell
# 加载指定的class
$ retransform /tmp/Test.class
# 列出所有加载的class
$ retransform -l
# 删除指定加载的class
$ retransform -d 1                    # delete retransform entry
# 删除所有加载的class
$ retransform --deleteAll             # delete all retransform entries
# 通配符加载class
$ retransform --classPattern demo.*
# 通过指定的编译器加载class
$ retransform --classLoaderClass 'sun.misc.Launcher$AppClassLoader' /tmp/Test.class
```
> 1. 对于同一个类，当存在多个 retransform entry 时，如果显式触发 retransform ，则最后添加的 entry 生效(id 最大的)。
> 2. 如果不清除掉所有的 retransform entry，并重新触发 retransform ，则 arthas stop 时，retransform 过的类仍然生效。

```shell
# 结合 jad/mc 命令使用

# jad 命令反编译，然后可以用其它编译器，比如 vim 来修改源码
jad --source-only com.example.demo.arthas.user.UserController > /tmp/UserController.java
# mc 命令来内存编译修改过的代码
mc /tmp/UserController.java -d /tmp
# 用 retransform 命令加载新的字节码
retransform /tmp/com/example/demo/arthas/user/UserController.class
```
> 1. 可以上传 .class 文件到服务器的技巧
> 2. retransform 的限制:
>   1. 不允许新增加 field/method 
>   2. 正在跑的函数，没有退出不能生效，比如下面新增加的System.out.println，只有run()函数里的会生效

* [sc](https://arthas.gitee.io/doc/sc.html) 查看 JVM 已加载的类信息
* [sm](https://arthas.gitee.io/doc/sm.html) 查看 JVM 已加载类的方法信息
  
#### monitor/watch/trace 相关
* [monitor](https://arthas.gitee.io/doc/monitor.html) 方法执行监控 
```shell
$ monitor [-b] [-c 5] [-E] class-pattern method-pattern [condition-express]
```
参数说明：

| 参数名称       | 参数说明           |
| -------------     |:-------------:|
| class-pattern     | 类名表达式匹配        |
| method-pattern    | 方法名表达式匹配      |   
| condition-express | 条件表达式            |  
| [E]               | 开启正则表达式匹配，默认为通配符匹配      |  
| [c]              | 统计周期，默认值为 120 秒     |  
| [b]               | 在方法调用之前计算    |  

```shell
# 计算条件表达式过滤统计结果(方法执行完毕之前)
monitor -b -c 5 com.test.testes.MathGame primeFactors "params[0] <= 2"
Press Q or Ctrl+C to abort.
Affect(class count: 1 , method count: 1) cost in 21 ms, listenerId: 4
 timestamp            class          method         total  success  fail  avg-rt(ms)  fail-rate
----------------------------------------------------------------------------------------------
 2020-09-02 09:41:57  demo.MathGame  primeFactors    1       0        1      0.10      100.00%

 timestamp            class          method         total  success  fail  avg-rt(ms)  fail-rate
----------------------------------------------------------------------------------------------
 2020-09-02 09:42:02  demo.MathGame  primeFactors    3       0        3      0.06      100.00%

 timestamp            class          method         total  success  fail  avg-rt(ms)  fail-rate
----------------------------------------------------------------------------------------------
 2020-09-02 09:42:07  demo.MathGame  primeFactors    2       0        2      0.06      100.00%

 timestamp            class          method         total  success  fail  avg-rt(ms)  fail-rate
----------------------------------------------------------------------------------------------
 2020-09-02 09:42:12  demo.MathGame  primeFactors    1       0        1      0.05      100.00%

 timestamp            class          method         total  success  fail  avg-rt(ms)  fail-rate
----------------------------------------------------------------------------------------------
 2020-09-02 09:42:17  demo.MathGame  primeFactors    2       0        2      0.10      100.00%
```
监控项说明：

| 监控项	     | 说明           |
| ------------- |:-------------: |
| timestamp     | 时间戳         |
| class         | Java 类        |   
| method        | 方法（构造方法、普通方法） |  
| total         | 调用次数 |  
| success       | 成功次数 |  
| fail          | 失败次数 |  
| rt            | 平均 RT  |  
| fail-rate	    | 失败率 |  

* [stack](https://arthas.gitee.io/doc/stack.html) 输出当前方法被调用的调用路径 (内 -> 外)
```shell
$ stack demo.MathGame primeFactors
Press Ctrl+C to abort.
Affect(class-cnt:1 , method-cnt:1) cost in 36 ms.
ts=2018-12-04 01:32:19;thread_name=main;id=1;is_daemon=false;priority=5;TCCL=sun.misc.Launcher$AppClassLoader@3d4eac69
    @demo.MathGame.run()
        at demo.MathGame.main(MathGame.java:16)

# 据条件表达式来过滤，[n]执行次数限制
$ stack demo.MathGame primeFactors 'params[0]<0' -n 2

# 据执行时间来过滤 （大于5毫秒）
$ stack demo.MathGame primeFactors '#cost>5'
```

* [trace](https://arthas.gitee.io/doc/trace.html) 方法内部调用路径，并输出方法路径上的每个节点上耗时(外 -> 内)
> 能方便的帮助你定位和发现因 RT 高而导致的性能问题缺陷，但其每次只能跟踪一级方法的调用链路。
```shell
# trace 次数限制 -n
$ trace demo.MathGame run -n 1
Press Q or Ctrl+C to abort.
Affect(class-cnt:1 , method-cnt:1) cost in 20 ms.
`---ts=2019-12-04 00:45:53;thread_name=main;id=1;is_daemon=false;priority=5;TCCL=sun.misc.Launcher$AppClassLoader@3d4eac69
    `---[0.549379ms] demo.MathGame:run()
        +---[0.059839ms] demo.MathGame:primeFactors() #24
        `---[0.232887ms] demo.MathGame:print() #25

Command execution times exceed limit: 1, so command will exit. You can set it with -n option.

# 包含 jdk 的函数，默认情况下，trace 不会包含 jdk 里的函数调用，如果希望 trace jdk 里的函数，需要显式设置--skipJDKMethod false
$ trace --skipJDKMethod false demo.MathGame run

# 据调用耗时过滤
$ trace demo.MathGame run '#cost > 10'
```

* [tt](https://arthas.gitee.io/doc/tt.html) 方法执行数据的时空隧道，记录下指定方法每次调用的入参和返回信息，并能对这些不同的时间下调用进行观测
```shell
$ tt -t -n 5 demo.MathGame primeFactors
Press Ctrl+C to abort.
Affect(class-cnt:1 , method-cnt:1) cost in 66 ms.
 INDEX   TIMESTAMP            COST(ms)  IS-RET  IS-EXP   OBJECT         CLASS                          METHOD
-------------------------------------------------------------------------------------------------------------------------------------
 1000    2018-12-04 11:15:38  1.096236  false   true     0x4b67cf4d     MathGame                       primeFactors
 1001    2018-12-04 11:15:39  0.191848  false   true     0x4b67cf4d     MathGame                       primeFactors
 1002    2018-12-04 11:15:40  0.069523  false   true     0x4b67cf4d     MathGame                       primeFactors
 1003    2018-12-04 11:15:41  0.186073  false   true     0x4b67cf4d     MathGame                       primeFactors
 1004    2018-12-04 11:15:42  17.76437  true    false    0x4b67cf4d     MathGame
```

> 解决方法重载，通过制定参数个数的形式解决不同的方法签名；如果参数个数一样, 可以判断参数的类型
> * tt -t *Test print params.length==1
> * tt -t *Test print 'params[1] instanceof Integer'

```sehll
# 检索调用记录
$ tt -l
 INDEX   TIMESTAMP            COST(ms)  IS-RET  IS-EXP   OBJECT         CLASS                          METHOD
-------------------------------------------------------------------------------------------------------------------------------------
 1000    2018-12-04 11:15:38  1.096236  false   true     0x4b67cf4d     MathGame                       primeFactors
 1001    2018-12-04 11:15:39  0.191848  false   true     0x4b67cf4d     MathGame                       primeFactors
 1002    2018-12-04 11:15:40  0.069523  false   true     0x4b67cf4d     MathGame                       primeFactors
 1003    2018-12-04 11:15:41  0.186073  false   true     0x4b67cf4d     MathGame                       primeFactors
 1004    2018-12-04 11:15:42  7.764379  true    false    0x4b67cf4d     MathGame                       primeFactors
 1005    2018-12-04 11:15:43  0.4776    false   true     0x4b67cf4d     MathGame                       primeFactors
Affect(row-cnt:6) cost in 4 ms.

# 筛选出 primeFactors 方法的调用信息
$ tt -s 'method.name=="primeFactors"'

# 查看调用信息
$ tt -i 1003
 INDEX            1003
 GMT-CREATE       2018-12-04 11:15:41
 COST(ms)         0.186073
 OBJECT           0x4b67cf4d
 CLASS            demo.MathGame
 METHOD           primeFactors
 IS-RETURN        false
 IS-EXCEPTION     true
 PARAMETERS[0]    @Integer[-564322413]
 THROW-EXCEPTION  java.lang.IllegalArgumentException: number is: -564322413, need >= 2
                      at demo.MathGame.primeFactors(MathGame.java:46)
                      at demo.MathGame.run(MathGame.java:24)
                      at demo.MathGame.main(MathGame.java:16)

# 重新调用一次，tt 命令由于保存了当时调用的所有现场信息，所以我们可以自己主动对一个 INDEX 编号的时间片自主发起一次调用，从而解放你的沟通成本。此时你需要 -p 参数。通过 --replay-times 指定 调用次数，通过 --replay-interval 指定多次调用间隔(单位 ms, 默认 1000ms)
$ tt -i 1004 -p
 RE-INDEX       1004
 GMT-REPLAY     2018-12-04 11:26:00
 OBJECT         0x4b67cf4d
 CLASS          demo.MathGame
 METHOD         primeFactors
 PARAMETERS[0]  @Integer[946738738]
 IS-RETURN      true
 IS-EXCEPTION   false
 COST(ms)         0.186073
 RETURN-OBJ     @ArrayList[
                    @Integer[2],
                    @Integer[11],
                    @Integer[17],
                    @Integer[2531387],
                ]
Time fragment[1004] successfully replayed.
Affect(row-cnt:1) cost in 14 ms.
```

* [watch](https://arthas.gitee.io/doc/watch.html)

```shell
$ watch class-pattern method-pattern [express] [condition-express]
```
参数说明：

| 参数名称       | 参数说明           |
| -------------     |:-------------:|
| class-pattern     | 类名表达式匹配        |
| method-pattern    | 方法名表达式匹配      |  
| express           | 观察表达式，默认值：{params, target, returnObj} |  
| condition-express | 条件表达式            |  
| [b]               | 在函数调用之前观察      |  
| [e]               | 在函数异常之后观察     |  
| [s]               | 在函数返回之后观察    |
| [f]               | 在函数结束之后(正常返回和异常返回)观察      |  
| [E]               | 开启正则表达式匹配，默认为通配符匹配     |  
| [x]               | 指定输出结果的属性遍历深度，默认为 1，最大值是 4    |  

> 1. 观察表达式的构成主要由 ognl 表达式组成，所以你可以这样写"{params,returnObj}"，只要是一个合法的 ognl 表达式，都能被正常支持。
> 2. 4 个观察事件点 -b、-e、-s 默认关闭，-f 默认打开，当指定观察点被打开后，在相应事件点会对观察表达式进行求值并输出
> 3. 当使用 -b 时，由于观察事件点是在函数调用前，此时返回值或异常均不存在
> 4. 观察表达式，默认值是{params, target, returnObj}
> 5. 在 watch 命令的结果里，会打印出location信息。location有三种可能值：AtEnter(对应函数入口)，AtExit(正常 return)，AtExceptionExit(函数，函数抛出异常)。

```shell
# 同时观察函数调用前和函数返回后，结果的输出顺序和事件发生的先后顺序一致，和命令中 -s -b 的顺序无关
$ watch demo.MathGame primeFactors "{params,target,returnObj}" -x 2 -b -s -n 2
Press Ctrl+C to abort.
Affect(class-cnt:1 , method-cnt:1) cost in 46 ms.
ts=2018-12-03 19:29:54; [cost=0.01696ms] result=@ArrayList[
    @Object[][
        @Integer[1],
    ],
    @MathGame[
        random=@Random[java.util.Random@522b408a],
        illegalArgumentCount=@Integer[13038],
    ],
    null,
]
ts=2018-12-03 19:29:54; [cost=4.277392ms] result=@ArrayList[
    @Object[][
        @Integer[1],
    ],
    @MathGame[
        random=@Random[java.util.Random@522b408a],
        illegalArgumentCount=@Integer[13038],
    ],
    @ArrayList[
        @Integer[2],
        @Integer[2],
        @Integer[2],
        @Integer[5],
        @Integer[5],
        @Integer[73],
        @Integer[241],
        @Integer[439],
    ],
]

# 条件表达式
$ watch demo.MathGame primeFactors "{params[0],target}" "params[0]<0"

# 观察异常信息的例子
$ watch demo.MathGame primeFactors "{params[0],throwExp}" -e -x 2

# 按照耗时进行过滤
$ watch demo.MathGame primeFactors '{params, returnObj}' '#cost>200' -x 2
```


> watch/stack/trace 这个三个命令都支持#cost

#### profiler/火焰图
* [profiler](https://arthas.gitee.io/doc/profiler.html)


### ognl表达式

#### [表达式核心变量](https://arthas.gitee.io/doc/advice-class.html)
```java
public class Advice {

    private final ClassLoader loader;           // 本次调用类所在的 ClassLoader
    private final Class<?> clazz;               // 本次调用类的 Class 引用
    private final ArthasMethod method;          // 本次调用方法反射引用
    private final Object target;                // 本次调用类的实例
    private final Object[] params;              // 本次调用参数列表，这是一个数组，如果方法是无参方法则为空数组
    private final Object returnObj;             // 本次调用返回的对象。当且仅当 isReturn==true 成立时候有效，表明方法调用是以正常返回的方式结束。如果当前方法无返回值 void，则值为 null
    private final Throwable throwExp;           // 本次调用抛出的异常。当且仅当 isThrow==true 成立时有效，表明方法调用是以抛出异常的方式结束。
    private final boolean isBefore;             // 辅助判断标记，当前的通知节点有可能是在方法一开始就通知，此时 isBefore==true 成立，同时 isThrow==false 和 isReturn==false，因为在方法刚开始时，还无法确定方法调用将会如何结束。
    private final boolean isThrow;              // 辅助判断标记，当前的方法调用以抛异常的形式结束。
    private final boolean isReturn;             // 辅助判断标记，当前的方法调用以正常返回的形式结束。

    // getter/setter
}
```
> 所有变量都可以在表达式中直接使用，如果在表达式中编写了不符合 OGNL 脚本语法或者引入了不在表格中的变量，则退出命令的执行；用户可以根据当前的异常信息修正条件表达式或观察表达式

### 应用场景

* 查找Top N线程   
  * thread -n N
* 排查死锁 (找出当前阻塞其他线程的线程)
  * thread -b
* 动态更新应用Logger Level
  * logger --name ROOT --level debug               # 更新全局日志等级
  * logger -c 2a139a55 --name ROOT --level debug   # 更新指定类日志等级
* 热更新代码
  * jad (反编译java) ->vim (更新代码)  -> mc (编译为class) -> retransform (重新加载class)
* 请求重放
  * tt -i 1004 -p
* 生成