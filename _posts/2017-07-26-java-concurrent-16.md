---
layout: post
title:  "并发编程常用的调试工具"
desc: "并发编程常用的调试工具"
keywords: "并发编程常用的调试工具,java,kinglyjn,张庆力"
date: 2017-07-28
categories: [Java]
tags: [Java]
icon: fa-coffee
---



### top命令

1) 使用top命令查看每个进程的情况，显示如下：

```shell
top - 14:05:54 up 40 min,  1 user,  load average: 0.02, 0.03, 0.05
Tasks: 176 total,   1 running, 173 sleeping,   2 stopped,   0 zombie
%Cpu(s):  0.0 us,  0.2 sy,  0.0 ni, 99.8 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem:   2031244 total,  1010452 used,  1020792 free,    42992 buffers
KiB Swap:        0 total,        0 used,        0 free.   724172 cached Mem
  PID  USER      PR   NI VIRT     RES    SHR  S   %CPU %MEM  TIME+   COMMAND
  9828 ubuntu    20   0 2037636  57820  16332 S   0.3  120%   0:01.55 java  
  9889 ubuntu    20   0   24956   3000   2504 R   0.3  0.1   0:00.04 top  
     1 root      20   0   33636   4196   2672 S   0.0  0.2   0:01.85 init  
     2 root      20   0       0      0      0 S   0.0  0.0   0:00.01 kthreadd
```

当我们的程序是java程序的售后，我们只需要关心COMMAND是java的性能数据，我们看到java进程一行里，cpu的使用率为120%，不用担心，这是当前机器所有的核加在一起的CPU使用率。

2) 使用top命令的交互命令数字1，查看每个cpu的性能数据：

```shell
top - 14:14:48 up 49 min,  1 user,  load average: 0.00, 0.01, 0.05
Tasks: 173 total,   1 running, 170 sleeping,   2 stopped,   0 zombie
%Cpu0  :  58.2% us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu1  :  63.4% us,  0.7 sy,  0.0 ni, 99.3 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem:   2031244 total,  1020068 used,  1011176 free,    43200 buffers
KiB Swap:        0 total,        0 used,        0 free.   724168 cached Mem
```

命令行显示了CPU1，这说明这是一个2核的CPU主机，平均每个cpu的利用率在60%左右，如果这里显示cpu利用率在100%，则很可能程序中写了一个死循环。这些参数的意义如下：

```default
us: 表示用户空间占用cpu百分比
1.0% sy: 内核空间占用cpu百分比
0.0% ni: 用户进程空间内改变过优先级的进程所占用的cpu百分比
98.7% id: 空闲cpu百分比
0.0% wa: 等待输入/输出cpu时间百分比
```

3) 使用top命令交互命令参数H查看线程的新能信息：

```shell
top - 14:24:50 up 59 min,  1 user,  load average: 0.03, 0.04, 0.05
Threads: 228 total,   1 running, 211 sleeping,  16 stopped,   0 zombie
%Cpu0  :  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu1  :  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem:   2031244 total,  1020816 used,  1010428 free,    43468 buffers
KiB Swap:        0 total,        0 used,        0 free.   724184 cached Mem
   PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND  
  9207 root      20   0  151672  10324   7896 S  0.3  0.5   0:03.57 vmtoolsd 
  9711 ubuntu    20   0  124056   4408   3252 S  0.3  0.2   0:01.50 sshd  
  9843 ubuntu    20   0 2037636  59876  16332 S  0.3  2.9   0:01.31 java 
  9889 ubuntu    20   0   24956   3000   2504 R  0.3  0.1   0:01.66 top    
     1 root      20   0   33636   4196   2672 S  0.0  0.2   0:01.85 init  
     2 root      20   0       0      0      0 S  0.0  0.0   0:00.01 kthreadd 
```

这里的进程使用情况可能会出现三种情况：

* 某个进程的cpu使用率一直是100%，则说明这个进程中可能有死循环，那么请记住这个PID（进程号）。
* 某个进程一直在top10的位置，说明这个进程有性能问题
* cpu高的几个线程在不停发生变化，说明某一个线程导致的cpu偏高

<br>



### 使用jps 或者 ps命令查看java程序的进程

```shell
ubuntu@nimbusz:~/test/shells$ jps
12487 Jps
9828 QuorumPeerMain  #zk进程

ps -aux |grep 9828
#或者
ps -ef |grep 9828
```

<br>



### java的 jstat命令 

一个极强的监视VM内存工具，可以用来监视VM内存内的各种堆和非堆的大小及其内存使用量。如何判断JVM垃圾回收是否正常？一般的top指令基本上满足不了这样的需求，因为它主要监控的是总体的系统资源，很难定位到java应用程序。

```shell
[root@localhost bin]# jstat -gcutil 25444
  S0     S1     E      O      P     YGC     YGCT    FGC    FGCT     GCT
 11.63   0.00   56.46  66.92  98.49 162    0.248    6      0.331    0.579

[root@localhost bin]# jstat -gcutil 25444 1000 5
  S0     S1     E      O      P     YGC     YGCT    FGC    FGCT     GCT
 73.54   0.00  99.04  67.52  98.49    166    0.252     6    0.331    0.583
 73.54   0.00  99.04  67.52  98.49    166    0.252     6    0.331    0.583
 73.54   0.00  99.04  67.52  98.49    166    0.252     6    0.331    0.583
 73.54   0.00  99.04  67.52  98.49    166    0.252     6    0.331    0.583
 73.54   0.00  99.04  67.52  98.49    166    0.252     6    0.331    0.583
 
 #参数解释
S0  — Heap上的 Survivor space 0 区已使用空间的百分比
S1  — Heap上的 Survivor space 1 区已使用空间的百分比
E   — Heap上的 Eden space 区已使用空间的百分比
O   — Heap上的 Old space 区已使用空间的百分比
P   — Perm space 区已使用空间的百分比
YGC — 从应用程序启动到采样时发生 Young GC 的次数
YGCT– 从应用程序启动到采样时 Young GC 所用的时间(单位秒)
FGC — 从应用程序启动到采样时发生 Full GC 的次数
FGCT– 从应用程序启动到采样时 Full GC 所用的时间(单位秒)
GCT — 从应用程序启动到采样时用于垃圾回收的总时间(单位秒)
```

<br>



### java的jstack命令

```shell
> jstack 9828
# 类似下面的内容
# 这里的tid是线程号
"QuorumPeer[myid=1]/0:0:0:0:0:0:0:0:2181" prio=10 tid=0x00007fa7a427a800 nid=0x2679 runnable [0x00007fa799c3c000]
   java.lang.Thread.State: RUNNABLE
        at java.net.SocketInputStream.socketRead0(Native Method)
        at java.net.SocketInputStream.read(SocketInputStream.java:152)
        at java.net.SocketInputStream.read(SocketInputStream.java:122)
        at java.io.BufferedInputStream.fill(BufferedInputStream.java:235)
        at java.io.BufferedInputStream.read(BufferedInputStream.java:254)
        - locked <0x00000000f241f878> (a java.io.BufferedInputStream)
        at java.io.DataInputStream.readInt(DataInputStream.java:387)
        at org.apache.jute.BinaryInputArchive.readInt(BinaryInputArchive.java:63)
        at org.apache.zookeeper.server.quorum.QuorumPacket.deserialize(QuorumPacket.java:83)
        at org.apache.jute.BinaryInputArchive.readRecord(BinaryInputArchive.java:103)
        at org.apache.zookeeper.server.quorum.Learner.readPacket(Learner.java:153)
        - locked <0x00000000f241f8f0> (a org.apache.jute.BinaryInputArchive)
        at org.apache.zookeeper.server.quorum.Follower.followLeader(Follower.java:85)
        at org.apache.zookeeper.server.quorum.QuorumPeer.run(QuorumPeer.java:786)
        
#使用printf进行进制转换：
printf "%x\n" 9828
2664
```

<br>



### jconsole监控

从Java 5开始 引入了 JConsole。JConsole 是一个内置 Java 性能分析器，可以从命令行或在 GUI shell 中运行。您可以轻松地使用 JConsole（或者，它更高端的 “近亲” VisualVM ）来监控 Java 应用程序性能和跟踪 Java 中的代码。启动JConsole：

1. 如果是从命令行启动，使 JDK 在 PATH 上，运行 jconsole 即可。
2. 如果从 GUI shell 启动，找到 JDK 安装路径，打开 bin 文件夹，双击 **jconsole** 。

​    当分析工具弹出时（取决于正在运行的 Java 版本以及正在运行的 Java 程序数量），可能会出现一个对话框，要求输入一个进程的 URL 来连接，也可能列出许多不同的本地 Java 进程（有时包含 JConsole 进程本身）来连接。

<br>

**如何设置JAVA程序运行时可以被JConsolse连接分析**

1. 本地程序（相对于开启JConsole的计算机），无需设置任何参数就可以被本地开启的JConsole连接（Java SE 6开始无需设置，之前还是需要设置运行时参数 -Dcom.sun.management.jmxremote ）

2. 无认证连接 (下面的设置表示：连接的端口为8999、无需认证就可以被连接)

   ```default
   -Dcom.sun.management.jmxremote.port=8999 \  
   -Dcom.sun.management.jmxremote.authenticate=false \  
   -Dcom.sun.management.jmxremote.ssl=false  
   ```

   如果考虑到安全因素，需要认证，需要安全连接，也是可以搞定的。可以参考后面的链接： [JDK文档链接](http://docs.oracle.com/javase/7/docs/technotes/guides/management/agent.html#gdenv) 。

3. 如果是jconsole进行远程连接，请参照我写的另一篇文章：[zookeeper运维](http://www.page.keyllo.com/linux/2017/07/26/zookeeper-base03.html) 。



###  JConsole工具的升级版 jvisualvm

```shell
> jvisualvm
  
#安装插件 
#只要安装JDK即可，运行jvisualvm.exe ，选择【工具】——【插件】——【可用插件】
#常用的插件VisualVM-JConsole、Visual-GC、Tracer-Collections Probes、Tracer-Monitor Probes
```

<br>



### 查询有多少台机器连接到本机的某个端口上

```java
netstat -lntp |grep 22 -c
2
```

<br>



### 文件内容搜索工具awk

awk，官网为 https://www.gnu.org/software/gawk/manual/gawk.html，是linux环境下的一个命令行工具，但是由于awk强大的能力，我们可以为awk工具传递一个字符串，该字符串的内容类似一种编程语言的语法，我们可以称其为Awk语言，而awk工具本身则可以看作是Awk语言的解析器。就好比python解析器与Python语言的关系。我们一般使用awk来做什么，awk又适合做什么工作呢。由于awk天生提供对文件中文本分列进行处理，所以如果一个文件中的每行都被特定的分隔符(常见的是空格)隔开，我们可以将这个文件看成是由很多列的文本组成，这样的文件最适合用awk进行处理，其实awk在工作中很多时候被用来处理log文件，进行一些统计工作等。

参照我之前写的文章：[参考](http://www.page.keyllo.com/linux/2016/11/11/linux-centos7-init-cmd.html)

<br>



### proc方式系统平均负载、cpu利用率、内存使用情况、查看网路流量

```shell
ubuntu@nimbusz:~$ cat /proc/loadavg 
0.00 0.01 0.05 1/217 12995


ubuntu@nimbusz:~$ cat /proc/stat 
cpu  5442 334 4277 3916162 132 0 197 0 0 0
cpu0 2708 18 1985 1957558 59 0 109 0 0 0
cpu1 2734 315 2292 1958604 72 0 88 0 0 0
intr xxx
ctxt 2130458
btime 1501738009
processes 13121
procs_running 1
procs_blocked 0
softirq 759696 1 275566 3934 108303 25238 0 2829 270673 0 73152


ubuntu@nimbusz:~$ cat /proc/meminfo 
MemTotal:        2031244 kB
MemFree:          942968 kB
MemAvailable:    1819272 kB
Buffers:           51628 kB
Cached:           804180 kB
SwapCached:            0 kB
Active:           474988 kB
Inactive:         482956 kB
...


ubuntu@nimbusz:~$ cat /proc/net/dev
Inter-|   Receive                                                |  Transmit
 face |bytes    packets errs drop fifo frame compressed multicast|bytes    packets errs drop fifo colls carrier compressed
  eth0: 5250143   67419    0    0    0     0          0         0 10269872   51441    0    0    0     0       0          0
    lo:   21725     199    0    0    0     0          0         0    21725     199    0    0    0     0       0          0
```

