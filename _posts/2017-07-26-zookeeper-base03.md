---
layout: post
title:  "zookeeper运维"
desc: "zookeeper运维"
keywords: "zookeeper运维,linux,kinglyjn,张庆力"
date: 2017-07-26
categories: [Linux]
tags: [Linux]
icon: fa-bookmark-o
---



### 运维配置详解

```properties
ubuntu@supervisor01z:~$ vi zookeeper-3.4.6/conf/zoo.cfg 

####################
基本配置
####################
#表示zk的一个基本时间单元，可用它的倍数来表示系统内部的时间间隔设置
tickTime=2000

#用于配置存储快照文件的目录，如果没有配置dataLogDir，那么事务日志也会存储在此目录
dataDir=/home/ubuntu/zookeeper-3.4.6/zkdata
#日志的存放路径
dataLogDir=/home/ubuntu/zookeeper-3.4.6/logs

#zk的运行端口，默认是2181
clientPort=2181
#server.A=host:port1:port2 
#其中A是一个数字，表示这个是第几号服务器
#host表示这个服务器的地址
#port1是follower服务器和leader服务器的通信端口
#port2是leader服务器选举过程中的投票通信端口
server.1=nimbusz:2888:3888
server.2=supervisor01z:2888:3888
server.3=supervisor02z:2888:3888


####################
高级配置
####################
#配置zk最大请求堆积数(限制服务端同时处理客户端发送请求的数量，默认值为1000)
globalOutstandingLimit=1000

#配置zk事务日志文件与分配的磁盘空间的大小，默认为64M
preAllocSize=64M

#配置相邻两次数据快照之间的事务操作的次数，zk会在snapCount次事务操作之后进行一次数据快照，默认为100000
snapCount=100000

#配置从socket层面限制【单个客户端与单台服务器】之间的并发连接数，它是以ip地址的粒度进行连接数的限制，
#如果将该参数设置为0，那么表示对连接数不做任何限制。默认值为60
maxClientCnxns=60

#针对多网卡的机器这个参数允许为每个ip地址指定不同的监听端口
#clientPortAddress

#配置服务端对客户端会话的超时时间限制，如果客户端设置的超时时间不在这个范围之内，
#那么会被服务端强制设置为最大或者是最小的超时时间。单位为心跳数，默认值分别为2，20
minSessionTimeout=2
maxSessionTimeout=20

#配置zk进行事务日志同步操作时消耗时间的报警时间阈值，一旦进行一个同步日志消耗的时间大于这个参数指定的时间，
#那么就会在日志中打印出报警日志，默认值为1000，单位为ms
fsync.warningthresholdms=1000

#自动清理snapshot和事务日志(单位为小时,默认为0表示不开启)
autopurge.purgeInterval=1

#配置zk在日志自动清理的时候，需要保留的快照数据文件数目和对应的事务日志文件，
#需要注意的是并不是磁盘上的所有快照数据文件和对应的事务日志文件都可以被清理掉，
#那样的话将无法恢复数据，因此这个配置默认值和最小值都为3，如果配置的值比3小，那么会自动调整为3
autopurge.snapRetainCount=3

#配置zk选举策略，从3.4.0开始，zk只保留了fastleader(3)一种选举方式，所以现在看起来这个参数没有什么用了
#electionAlg=3
#表示follower连接到leader，初始化连接时最长能忍受多少个心跳的时间间隔，这里的10表示，当已经超过了10个
#tickTime心跳的时间，zk服务器还没有收到客户端的返回信息，表明这个客户端连接失败。默认值为10
initLimit=10	

#配置leader服务器和follower之间进行心跳检测的最大延时时间，在zk集群运行过程中，leader服务器会与所有的follower服务器进行心跳检测，来确定这个服务器出了问题，如果leader服务器在这个参数规定的时间内无法获取到follower心跳检测响应，那么leader就会认为这个follower已经脱离了和自己的同步，通常情况下使用默认值即可。默认值为5。如果网络状况不好的话，可以适当调大这个参数。
syncLimit=5

#用于配置leader服务器是否接受客户端的连接，也就是是否允许leader向客户端直接提供服务，在默认情况下leader能够接受并处理客户端所有请求，在zk的设计中，leader服务器主要用来对事务更新请求的协调以及集群本身运行时的协调，因此可以让leader服务器不接受客户端的请求，以使得它专注于进行分布式的协调。默认值为 yes
leaderServes=yes

#配置leader选举过程中各个服务器进行TCP连接的创建的超时时间，默认值为5000
cnxTimeout=5000

#配置zk服务器是否在事务提交的时候，将日志写入操作强制刷入磁盘(yes|no，默认yes)
forceSync=yes

#配置单个数据节点上可以存储的最大数据量，通常使用默认(1M)
#jute.maxbuffer

#配置zk服务器是否跳过acl权限检查(yes|no，默认情况为no)
skipACL=no
```

<br>



### 四字命令

```shell
#输出zk运行时的状态信息
> echo stat | nc supervisor01z 2181
Zookeeper version: 3.4.6-1569965, built on 02/20/2014 09:09 GMT
Clients:
 /172.16.127.130:46468[0](queued=0,recved=1,sent=0)
Latency min/avg/max: 0/0/803
Received: 5463
Sent: 5463
Connections: 1
Outstanding: 0
Zxid: 0x41400000488
Mode: leader
Node count: 24

#输出zk运行时的更加详细的状态信息
> echo mntr | nc supervisor01z 2181
zk_version      3.4.6-1569965, built on 02/20/2014 09:09 GMT
zk_avg_latency  0
zk_max_latency  803
zk_min_latency  0
zk_packets_received     5485
zk_packets_sent 5485
zk_num_alive_connections        1
zk_outstanding_requests 0
zk_server_state leader
zk_znode_count  25
zk_watch_count  0
zk_ephemerals_count     0
zk_approximate_data_size        552
zk_open_file_descriptor_count   35
zk_max_file_descriptor_count    4096
zk_followers    2
zk_synced_followers     2
zk_pending_syncs        0

#查看服务器配置信息(注意不包含所有配置项)
> echo conf | nc supervisor01z 2181
clientPort=2181
dataDir=/home/ubuntu/zookeeper-3.4.6/zkdata/version-2
dataLogDir=/home/ubuntu/zookeeper-3.4.6/logs/version-2
tickTime=3000
maxClientCnxns=60
minSessionTimeout=6000
maxSessionTimeout=60000
serverId=2
initLimit=10
syncLimit=5
electionAlg=3
electionPort=3888
quorumPort=2888
peerType=0

#查看连接到当前服务器的所有连接信息
> echo cons | nc supervisor01z 2181
/172.16.127.129:34272[0](queued=0,recved=1,sent=0)

#重置所有客户端连接信息
> echo crst | nc supervisor01z 2181   
Connection stats reset.

#输出当前集群的所有会话信息宝库哦这些会话的会话id、以及每个会话创建的临时节点信息
> echo dump | nc supervisor01z 2181
SessionTracker dump:
Session Sets (4):
0 expire at Thu Jul 27 15:06:48 CST 2017:
1 expire at Thu Jul 27 15:07:00 CST 2017:
        0x15d7dd05c0e00bd
0 expire at Thu Jul 27 15:07:03 CST 2017:
1 expire at Thu Jul 27 15:07:06 CST 2017:
        0x15d7dd05c0e00be
ephemeral nodes dump:
Sessions with Ephemerals (0):

#输出zk所在服务器运行时的环境信息
> echo envi | nc supervisor01z 2181
Environment:
zookeeper.version=3.4.6-1569965, built on 02/20/2014 09:09 GMT
host.name=supervisor01z
java.version=1.7.0_111
java.vendor=Oracle Corporation
java.home=/usr/lib/jvm/java-7-openjdk-amd64/jre
java.class.path=/home/ubuntu/zookeeper-3.4.6/bin/../build/classes:/home/ubuntu/zookeeper-3.4.6/bin/../build/lib/*.jar:/home/ubuntu/zookeeper-3.4.6/bin/../lib/slf4j-log4j12-1.6.1.jar:/home/ubuntu/zookeeper-3.4.6/bin/../lib/slf4j-api-1.6.1.jar:/home/ubuntu/zookeeper-3.4.6/bin/../lib/netty-3.7.0.Final.jar:/home/ubuntu/zookeeper-3.4.6/bin/../lib/log4j-1.2.16.jar:/home/ubuntu/zookeeper-3.4.6/bin/../lib/jline-0.9.94.jar:/home/ubuntu/zookeeper-3.4.6/bin/../zookeeper-3.4.6.jar:/home/ubuntu/zookeeper-3.4.6/bin/../src/java/lib/*.jar:/home/ubuntu/zookeeper-3.4.6/bin/../conf:
java.library.path=/usr/java/packages/lib/amd64:/usr/lib/x86_64-linux-gnu/jni:/lib/x86_64-linux-gnu:/usr/lib/x86_64-linux-gnu:/usr/lib/jni:/lib:/usr/lib
java.io.tmpdir=/tmp
java.compiler=<NA>
os.name=Linux
os.arch=amd64
os.version=4.2.0-27-generic
user.name=ubuntu
user.home=/home/ubuntu
user.dir=/home/ubuntu/zookeeper-3.4.6

#输出当前这台服务器是否正在运行
> echo ruok | nc supervisor01z 2181
imok

#输出当前服务器上管理的watcher的概要信息
> echo wcsh | nc supervisor01z 2181

#输出当前服务器上管理的watcher的详细信息
> echo wchc | nc supervisor01z 2181
```

<br>



### 使用JMX运维zk集群

要在运维中使用jmx，有两个大的步骤：

* 修改zk启动脚本，重启zk服务器
* 使用jcontrol（jdk1.7以后为 jconsole）

1）修改zookeeper的启动脚本vim zkServer.sh。找到启动参数ZOOMAIN，修改为下面值。
其中local.only=false，设为false才能在远程建立连接。

```shell
ZOOMAIN="-Dcom.sun.management.jmxremote  -Dcom.sun.management.jmxremote.local.only=false  -Djava.rmi.server.hostname=nimbusz  -Dcom.sun.
management.jmxremote.port=9991  -Dcom.sun.management.jmxremote.ssl=false  -Dcom.sun.management.jmxremote.authenticate=false  org.apache.zook
eeper.server.quorum.QuorumPeerMain"  
 
#说明：
#Djava.rmi.server.hostname：表示使用jconsole连接到zk时的服务器的名称
#Dcom.sun.management.jmxremote.port：表示使用jconsole连接到zk时的服务器的端口号
#Dcom.sun.management.jmxremote.ssl=true：表示是否启用ssl
#Dcom.sun.management.jmxremote.authenticate：表示是否启用权限验证
#Dcom.sun.management.jmxremote.access.file：权限jmxremote.access文件(文件权限需设置为600)
#Dcom.sun.management.jmxremote.password.file：权限jmxremote.password文件(文件权限需设置为600)

#关于权限设置可参考JDK里的文件 xxx\jre\lib\management\jmxremote.password.template
#jmxremote.access文件内容
monitorRole   readonly  
controlRole   readwrite \  
              create javax.management.monitor.*,javax.management.timer.* \  
              unregister

#jmxremote.password文件内容
monitorRole  123456
controlRole  123456
```

<br>



2）使用jcontrol（jdk1.7以后为 jconsole）

```shell
#运行jconsole
> $JAVA_HOME/bin/jconsole

#选择远程进程进行连接
> nimbusz:9991  #然后点击连接
```

<br>



### 监控平台的搭建和使用

* netflix公司的exhibitor：zk集群的监控及设置
* zabbix：业界比较出色的服务器监控软件

