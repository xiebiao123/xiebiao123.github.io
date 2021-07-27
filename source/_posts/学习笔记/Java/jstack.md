---
title: jstack
date: 2019-10-25
categories:
    - 学习
tags:
    - java
---

### [jstack](https://jingyan.baidu.com/article/4f34706e3ec075e387b56df2.html)

#### 简介

``` shell
jstack用于打印出给定的java进程ID或core file或远程调试服务的Java堆栈信息，如果是在64位机器上，需要指定选项"-J-d64"，
Windows的jstack使用方式只支持以下的这种方式
    jstack [-l] pid

如果java程序崩溃生成core文件，jstack工具可以用来获得core文件的java stack和native stack的信息，从而可以轻松地知道java程
序是如何崩溃和在程序何处发生问题。另外，jstack工具还可以附属到正在运行的java程序中，看到当时运行的java程序的java stack和
native stack的信息, 如果现在运行的java程序呈现hung的状态，jstack是非常有用的

Usage
    jstack [-l] <pid>
        (to connect to running process) 连接活动线程
    jstack -F [-m] [-l] <pid>
        (to connect to a hung process) 连接阻塞线程
    jstack [-m] [-l] <executable> <core>
        (to connect to a core file) 连接dump的文件
    jstack [-m] [-l] [server_id@]<remote server IP or hostname>
        (to connect to a remote debug server) 连接远程服务器

Options
    -F  to force a thread dump. Use when jstack <pid> does not respond (process is hung)
    -m  to print both java and native frames (mixed mode)
    -l  long listing. Prints additional information about locks
    -h or -help to print this help message
```

#### jstack查看输出

``` shell
/opt/java8/bin/jstack -l 28367

2019-06-25 15:04:46
Full thread dump Java HotSpot(TM) 64-Bit Server VM (25.77-b03 mixed mode):

"Attach Listener" #453 daemon prio=9 os_prio=0 tid=0x00007f9f94001000 nid=0xf30 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
        - None

"grpc-default-executor-263" #452 daemon prio=5 os_prio=0 tid=0x00007f9f4c01f800 nid=0x9aa waiting on condition [0x00007f9f398bd000]
   java.lang.Thread.State: TIMED_WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x00000007400243f0> (a java.util.concurrent.SynchronousQueue$TransferStack)
        at java.util.concurrent.locks.LockSupport.parkNanos(LockSupport.java:215)
        at java.util.concurrent.SynchronousQueue$TransferStack.awaitFulfill(SynchronousQueue.java:460)
        at java.util.concurrent.SynchronousQueue$TransferStack.transfer(SynchronousQueue.java:362)
        at java.util.concurrent.SynchronousQueue.poll(SynchronousQueue.java:941)
        at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1066)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1127)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
        at java.lang.Thread.run(Thread.java:745)

   Locked ownable synchronizers:
        - None

"http-bio-8080-exec-10" #235 daemon prio=5 os_prio=0 tid=0x0000000001bcc800 nid=0x3c13 waiting on condition [0x00007f9f384a9000]
   java.lang.Thread.State: WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x0000000743d26638> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
        at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2039)
        at java.util.concurrent.LinkedBlockingQueue.take(LinkedBlockingQueue.java:442)
        at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:104)
        at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:32)
        at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1067)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1127)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
        at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
        at java.lang.Thread.run(Thread.java:745)

   Locked ownable synchronizers:
        - None
```

#### jstack统计线程数

``` shell
/opt/java8/bin/jstack -l 28367 | grep 'java.lang.Thread.State' | wc -l
```

#### jstack检测死锁

##### 死锁代码

``` java
public class DeathLock {

    private static Lock lock1 = new ReentrantLock();
    private static Lock lock2 = new ReentrantLock();

    public static void deathLock() {
        Thread t1 = new Thread() {
            @Override
            public void run() {
                try {
                    lock1.lock();
                    TimeUnit.SECONDS.sleep(1);
                    lock2.lock();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        };
        Thread t2 = new Thread() {
            @Override
            public void run() {
                try {
                    lock2.lock();
                    TimeUnit.SECONDS.sleep(1);
                    lock1.lock();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        };

        t1.setName("mythread1");
        t2.setName("mythread2");
        t1.start();
        t2.start();
    }

    public static void main(String[] args) {
        deathLock();
    }
}
```

##### 死锁日志

``` sehll
"mythread2" #12 prio=5 os_prio=0 tid=0x0000000058ef7800 nid=0x1ab4 waiting on condition [0x0000000059f8f000]
   java.lang.Thread.State: WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x00000000d602d610> (a java.util.concurrent.lock
s.ReentrantLock$NonfairSync)
        at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer.parkAndCheckInt
errupt(AbstractQueuedSynchronizer.java:836)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer.acquireQueued(A
bstractQueuedSynchronizer.java:870)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer.acquire(Abstrac
tQueuedSynchronizer.java:1199)
        at java.util.concurrent.locks.ReentrantLock$NonfairSync.lock(ReentrantLo
ck.java:209)
        at java.util.concurrent.locks.ReentrantLock.lock(ReentrantLock.java:285)

        at DeathLock$2.run(DeathLock.java:34)

   Locked ownable synchronizers:
        - <0x00000000d602d640> (a java.util.concurrent.locks.ReentrantLock$Nonfa
irSync)

"mythread1" #11 prio=5 os_prio=0 tid=0x0000000058ef7000 nid=0x3e68 waiting on condition [0x000000005947f000]
   java.lang.Thread.State: WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x00000000d602d640> (a java.util.concurrent.lock
s.ReentrantLock$NonfairSync)
        at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer.parkAndCheckInt
errupt(AbstractQueuedSynchronizer.java:836)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer.acquireQueued(A
bstractQueuedSynchronizer.java:870)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer.acquire(Abstrac
tQueuedSynchronizer.java:1199)
        at java.util.concurrent.locks.ReentrantLock$NonfairSync.lock(ReentrantLo
ck.java:209)
        at java.util.concurrent.locks.ReentrantLock.lock(ReentrantLock.java:285)

        at DeathLock$1.run(DeathLock.java:22)

   Locked ownable synchronizers:
        - <0x00000000d602d610> (a java.util.concurrent.locks.ReentrantLock$Nonfa
irSync)


Found one Java-level deadlock:
=============================
"mythread2":
  waiting for ownable synchronizer 0x00000000d602d610, (a java.util.concurrent.l
ocks.ReentrantLock$NonfairSync),
  which is held by "mythread1"
"mythread1":
  waiting for ownable synchronizer 0x00000000d602d640, (a java.util.concurrent.l
ocks.ReentrantLock$NonfairSync),
  which is held by "mythread2"

Java stack information for the threads listed above:
===================================================
"mythread2":
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x00000000d602d610> (a java.util.concurrent.lock
s.ReentrantLock$NonfairSync)
        at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer.parkAndCheckInt
errupt(AbstractQueuedSynchronizer.java:836)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer.acquireQueued(A
bstractQueuedSynchronizer.java:870)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer.acquire(Abstrac
tQueuedSynchronizer.java:1199)
        at java.util.concurrent.locks.ReentrantLock$NonfairSync.lock(ReentrantLo
ck.java:209)
        at java.util.concurrent.locks.ReentrantLock.lock(ReentrantLock.java:285)

        at DeathLock$2.run(DeathLock.java:34)
"mythread1":
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x00000000d602d640> (a java.util.concurrent.lock
s.ReentrantLock$NonfairSync)
        at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer.parkAndCheckInt
errupt(AbstractQueuedSynchronizer.java:836)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer.acquireQueued(A
bstractQueuedSynchronizer.java:870)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer.acquire(Abstrac
tQueuedSynchronizer.java:1199)
        at java.util.concurrent.locks.ReentrantLock$NonfairSync.lock(ReentrantLo
ck.java:209)
        at java.util.concurrent.locks.ReentrantLock.lock(ReentrantLock.java:285)

        at DeathLock$1.run(DeathLock.java:22)

Found 1 deadlock.
```

#### jstack检测cpu高

##### 步骤一：查看cpu占用高进程

``` sehll
top

Mem:  16333644k total,  9472968k used,  6860676k free,   165616k buffers
Swap:        0k total,        0k used,        0k free,  6665292k cached

  PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND     
17850 root      20   0 7588m 112m  11m S 100.7  0.7  47:53.80 java       
 1552 root      20   0  121m  13m 8524 S  0.7  0.1  14:37.75 AliYunDun   
 3581 root      20   0 9750m 2.0g  13m S  0.7 12.9 298:30.20 java        
    1 root      20   0 19360 1612 1308 S  0.0  0.0   0:00.81 init        
    2 root      20   0     0    0    0 S  0.0  0.0   0:00.00 kthreadd    
    3 root      RT   0     0    0    0 S  0.0  0.0   0:00.14 migration/0 
```

##### 步骤二：查看cpu占用高线程

``` sehll
top -H -p 17850

top - 17:43:15 up 5 days,  7:31,  1 user,  load average: 0.99, 0.97, 0.91
Tasks:  32 total,   1 running,  31 sleeping,   0 stopped,   0 zombie
Cpu(s):  3.7%us,  8.9%sy,  0.0%ni, 87.4%id,  0.0%wa,  0.0%hi,  0.0%si,  0.0%st
Mem:  16333644k total,  9592504k used,  6741140k free,   165700k buffers
Swap:        0k total,        0k used,        0k free,  6781620k cached

  PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND
17880 root      20   0 7588m 112m  11m R 99.9  0.7  50:47.43 java
17856 root      20   0 7588m 112m  11m S  0.3  0.7   0:02.08 java
17850 root      20   0 7588m 112m  11m S  0.0  0.7   0:00.00 java
17851 root      20   0 7588m 112m  11m S  0.0  0.7   0:00.23 java
17852 root      20   0 7588m 112m  11m S  0.0  0.7   0:02.09 java
17853 root      20   0 7588m 112m  11m S  0.0  0.7   0:02.12 java
17854 root      20   0 7588m 112m  11m S  0.0  0.7   0:02.07 java
```

##### 步骤三：转换线程ID

``` shell
printf "%x\n" 17880          
45d8
```

##### 步骤四：定位cpu占用线程

``` shell
jstack 17850|grep 45d8 -A 30
"pool-1-thread-11" #20 prio=5 os_prio=0 tid=0x00007fc860352800 nid=0x45d8 runnable [0x00007fc8417d2000]
   java.lang.Thread.State: RUNNABLE
        at java.io.FileOutputStream.writeBytes(Native Method)
        at java.io.FileOutputStream.write(FileOutputStream.java:326)
        at java.io.BufferedOutputStream.flushBuffer(BufferedOutputStream.java:82)
        at java.io.BufferedOutputStream.flush(BufferedOutputStream.java:140)
        - locked <0x00000006c6c2e708> (a java.io.BufferedOutputStream)
        at java.io.PrintStream.write(PrintStream.java:482)
        - locked <0x00000006c6c10178> (a java.io.PrintStream)
        at sun.nio.cs.StreamEncoder.writeBytes(StreamEncoder.java:221)
        at sun.nio.cs.StreamEncoder.implFlushBuffer(StreamEncoder.java:291)
        at sun.nio.cs.StreamEncoder.flushBuffer(StreamEncoder.java:104)
        - locked <0x00000006c6c26620> (a java.io.OutputStreamWriter)
        at java.io.OutputStreamWriter.flushBuffer(OutputStreamWriter.java:185)
        at java.io.PrintStream.write(PrintStream.java:527)
        - eliminated <0x00000006c6c10178> (a java.io.PrintStream)
        at java.io.PrintStream.print(PrintStream.java:597)
        at java.io.PrintStream.println(PrintStream.java:736)
        - locked <0x00000006c6c10178> (a java.io.PrintStream)
        at com.demo.guava.HardTask.call(HardTask.java:18)
        at com.demo.guava.HardTask.call(HardTask.java:9)
        at java.util.concurrent.FutureTask.run(FutureTask.java:266)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
        at java.lang.Thread.run(Thread.java:745)

"pool-1-thread-10" #19 prio=5 os_prio=0 tid=0x00007fc860345000 nid=0x45d7 waiting on condition [0x00007fc8418d3000]
   java.lang.Thread.State: WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x00000006c6c14178> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
        at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
```
