---
title: jmap
date: 2019-10-25
categories:
    - 学习
tags:
    - java
---

### [jmap](https://www.jianshu.com/p/2499c0cbf7a3)
#### 简介
jmap是一个JDK自带的多功能的命令。它可以生成 java 程序的 dump 文件， 也可以查看堆内对象示例的统计信息、查看 ClassLoader 的
信息以及 finalizer 队列

#### jmap命令
```
Usage:
    jmap [option] <pid>
        (to connect to running process) 连接活动线程
    jmap [option] <executable <core>
        (to connect to a core file)  连接dump的文件
    jmap [option] [server_id@]<remote server IP or hostname>
        (to connect to remote debug server) 连接远程服务器

where <option> is one of:
    <none>               to print same info as Solaris pmap
    -heap                to print java heap summary
    -histo[:live]        to print histogram of java object heap; if the "live"
                         suboption is specified, only count live objects
    -clstats             to print class loader statistics
    -finalizerinfo       to print information on objects awaiting finalization
    -dump:<dump-options> to dump java heap in hprof binary format
                         dump-options:
                           live         dump only live objects; if not specified,
                                        all objects in the heap are dumped.
                           format=b     binary format
                           file=<file>  dump heap to <file>
                         Example: jmap -dump:live,format=b,file=heap.bin <pid>
    -F                   force. Use with -dump:<dump-options> <pid> or -histo
                         to force a heap dump or histogram when <pid> does not
                         respond. The "live" suboption is not supported
                         in this mode.
    -h | -help           to print this help message
    -J<flag>             to pass <flag> directly to the runtime system
```

#### jmap -heap例子
```
/opt/java8/bin/jmap -heap 28367
Attaching to process ID 28367, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.77-b03

using parallel threads in the new generation.
using thread-local object allocation.
Concurrent Mark-Sweep GC

Heap Configuration:
   MinHeapFreeRatio         = 40
   MaxHeapFreeRatio         = 70
   MaxHeapSize              = 3221225472 (3072.0MB)
   NewSize                  = 1073741824 (1024.0MB)
   MaxNewSize               = 1073741824 (1024.0MB)
   OldSize                  = 2147483648 (2048.0MB)
   NewRatio                 = 2
   SurvivorRatio            = 8
   MetaspaceSize            = 21807104 (20.796875MB)
   CompressedClassSpaceSize = 1073741824 (1024.0MB)
   MaxMetaspaceSize         = 17592186044415 MB
   G1HeapRegionSize         = 0 (0.0MB)

Heap Usage:
New Generation (Eden + 1 Survivor Space):
   capacity = 966393856 (921.625MB)
   used     = 498418872 (475.3292770385742MB)
   free     = 467974984 (446.2957229614258MB)
   51.575128391544744% used
Eden Space:
   capacity = 859045888 (819.25MB)
   used     = 455409032 (434.31189727783203MB)
   free     = 403636856 (384.93810272216797MB)
   53.013353344868115% used
From Space:
   capacity = 107347968 (102.375MB)
   used     = 43009840 (41.01737976074219MB)
   free     = 64338128 (61.35762023925781MB)
   40.065816616109586% used
To Space:
   capacity = 107347968 (102.375MB)
   used     = 0 (0.0MB)
   free     = 107347968 (102.375MB)
   0.0% used
concurrent mark-sweep generation:
   capacity = 2147483648 (2048.0MB)
   used     = 62204400 (59.32273864746094MB)
   free     = 2085279248 (1988.677261352539MB)
   2.8966180980205536% used

22841 interned Strings occupying 2735144 bytes.
```
* MaxHeapSize = 3221225472 (3072.0MB) 最大堆大小
* NewSize = 1073741824 (1024.0MB) 新生成堆大小
* MaxNewSize = 1073741824 (1024.0MB) 最大新生代堆大小
* OldSize = 2147483648 (2048.0MB) 老年代堆大小
* NewRatio = 2 老年代/新生代 比列，例子中新生代占1/3，老年代占2/3
* SurvivorRatio = 8 新生代中Eden/S0的比例，例子中Eden占8/10，from和to各占1/10

#### jmap histo[:live]例子
```
 num     #instances         #bytes  class name
----------------------------------------------
   1:        372501      512486688  [B
   2:        784169       50697752  [C
   3:          9341       19841992  [I
   4:        584062       14017488  java.lang.String
   5:        271464       11324272  [Ljava.lang.Object;
   6:         67905        7605360  java.net.SocksSocketImpl
   7:        357504        5720064  java.lang.Object
   8:        169855        5435360  org.apache.commons.pool2.impl.LinkedBlockingDeque$DescendingItr
   9:        122874        3931968  java.util.LinkedList
  10:        148881        3573144  java.util.ArrayList
  11:         67900        3259200  java.net.SocketInputStream
  12:         67900        3259200  java.net.SocketOutputStream
  13:        179979        2879664  java.lang.Integer
  14:         68799        2751960  java.lang.ref.Finalizer
  15:         35629        2280256  java.nio.DirectByteBuffer
  16:         28278        2262240  java.net.URI
  17:         70065        2242080  java.io.FileDescriptor
  18:         67952        2174464  java.net.InetAddress$InetAddressHolder
  19:         67902        2172864  java.net.Socket
  20:         89583        2149992  java.lang.StringBuilder
  21:         29078        2093616  org.apache.commons.pool2.impl.DefaultPooledObject
  22:         58096        1859072  java.util.HashMap$Node
  23:         67946        1630704  java.net.Inet4Address
  24:         29078        1628368  redis.clients.jedis.Client
  25:         36784        1471360  org.apache.skywalking.apm.dependencies.io.netty.buffer.UnpooledSlicedByteBuf
  26:         44612        1427584  java.util.concurrent.ConcurrentHashMap$Node
  27:         31039        1241560  java.util.HashMap$KeyIterator
  28:         10920        1211016  java.lang.Class
  29:         31849        1201864  [[B
  30:          3695        1101344  [Ljava.util.HashMap$Node;
  31:         29726        1070032  [Lorg.apache.skywalking.apm.dependencies.io.netty.util.AsciiString;
  32:         11387        1002056  java.lang.reflect.Method
  33:         17692         990752  org.apache.skywalking.apm.dependencies.io.grpc.CallOptions
```
* B byte
* C char
* D double
* F float
* S short
* I int
* L long
* Z boolean
* [ 数组，如[I 表示int[]
* [L + 类名 其他对象

#### 生成堆转储快照dump文件
```
/opt/java8/bin/jmap -dump:format=b,file=/home/buser/heapdump.phrof  28367 
Dumping heap to /home/buser/heapdump.phrof ...
Heap dump file created


-rw------- 1 root root 911642981 Jun 25 17:28 heapdump.phrof
```
* 命令：jmap -dump:format=b,file=heapdump.phrof pid
* 以hprof二进制格式转储Java堆到指定filename的文件中。live子选项是可选的。如果指定了live子选项，堆中只有活动的对象会被转储。
想要浏览heap dump，你可以使用jhat(Java堆分析工具)读取生成的文件