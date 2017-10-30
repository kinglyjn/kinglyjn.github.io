---
layout: post
title:  "flume的安装和使用"
desc: "flume的安装和使用"
keywords: "flume的安装和使用,kinglyjn"
date: 2017-10-27
categories: [Java]
tags: [java]
icon: fa-coffee
---



### flume简介

Flume是Cloudera提供的一个高可用的，高可靠的，分布式的海量日志采集、聚合和传输的系统。最早是Cloudera提供的日志收集系统，目前是Apache下的一个孵化项目。支持在日志 系统中定制各类数据发送方，用于收集数据；同时，Flume提供对数据进行简单处理，并写到各种数据接受方。Flume采用了多Master的方式，为了保证配置数据的一致性，Flume引入了ZooKeeper，用于保存配置数据，ZooKeeper本身可保证配置数据的一致性和高可用，另外，在配置数据发生变化时，ZooKeeper可以通知Flume Master节点，Flume Master间使用gossip协议同步数据。

Flume在目前的使用过程中，有两种日志采集结构是比较常用，第一种是多agent链路模式，第二种是多渠道采集模式。具体的结构图如下：

<img src="http://img.blog.csdn.net/20171030113141385?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:60%"/>

<br>

<img src="http://img.blog.csdn.net/20171030113154519?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:60%"/>



Flume日志采集工具从结构图上可以看出，它是一个集群式的部署方式，故在选择主机时，要按照主机所担当的角色来划分主机。在Flume日志采集系统的系统架构中，共有三种主机角色：日志采集源(source)、渠道/缓冲区(channel)和目标主机(sink)。

<br>



### 常见的四种source组件

Source作为Flume中的一个组成部分，其作用是用来作为日志采集的源头而存在，Flume可以通过其提供的多种源来实现日志的采集工作，Flume的源有很多种，下面列出的是比较常见的，适应我们目前的应用场景的源的配置方式，如果下面这四种源不能满足需求，可以到官方网站查看[更多源介绍及配置方式](http://flume.apache.org/FlumeUserGuide.html#flume-sources)。

Avro Source

```default
这个源可以监听一个端口，接收外部Avro客户端发送到这个端口上的数据，可以作为分层拓扑结构中的中转源而存在。
具体的配置方式如下：
a1.sources = r1
a1.channels = c1
a1.sources.r1.type = avro
a1.sources.r1.channels = c1
a1.sources.r1.bind = 0.0.0.0
a1.sources.r1.port = 4141

配置属性如下(必要参数为#标识)：
#channels	当前源中的数据通过哪一频道进行下一步传输
#type		当前源类型的名称，必须填写为avro
#bind		当前源所监听的IP地址，推荐使用127.0.0.1，可以监听任何请求到本机的请求
#port		当前源所监听的端口，要保证这一端口没有被占用
threads		最大工作线程数，需要根据客户端的数量及发送数据的频率进行设置
```

<br>



Exec Source

```default
这个源可以将执行命令所产生的标准输出作为采集内容。这种命令最好可以不断的产生新的数据，如果不能需要设置命令的执行频率以保证数据的连续性。如果是单次执行连续输出的命令，当命令被意外中断，则这种源不会再产生新的数据。
具体配置方式如下：
a1.sources = r1
a1.channels = c1
a1.sources.r1.type = exec
a1.sources.r1.command = tail -F /var/log/secure
a1.sources.r1.channels = c1

配置属性如下(必要参数为#标识)：
#channels		当前源中的数据通过哪一频道进行下一步传输
#type			当前源类型的名称，必须填写为exec
#command		需要执行的命令
shell			如果执行的命令是脚本命令，那么使用什么类型的脚本
restart			当前命令是否需要重复执行(默认值为false)
restartThrottle	命令重复执行频率，单位为毫秒(默认值为10000)
batchTimeout	Source的拉取频率，单位为毫秒(默认为3000)
batchSize		采集多少行数据向频道批量发送一次(默认值为20)
logStdErr		当前命令所产生的标准错误输出是否需要采集(默认值为false)
```

<br>



Spooling Directory Source

```default
此种源会将目标文件夹中新出现的文件作为要采集的资源，为了保证文件的完整采集和不被重复采集，此种源会给采集完成的文件添加上一个特殊的后缀名。
注意：此种源只能够检测并采集整个文件，不能够检测到文件内部的变化，所以不适用于文件被创建后内容依然会不断变化的应用场景。如果需要使用在这种场景下，那么需要利用正则表达式将会不断发生变化的文件进行屏蔽，以保证系统正在进行操作的文件内容不会被同步。
具体的配置方式如下：
agent-1.channels = ch-1
agent-1.sources = src-1
agent-1.sources.src-1.type = spooldir
agent-1.sources.src-1.channels = ch-1
agent-1.sources.src-1.spoolDir = /var/log/apache/flumeSpool

配置属性如下(必要参数为#标识)：
#channels		当前源中的数据通过哪一频道进行下一步传输
#type			当前源类型的名称，必须填写为spooldir
#spoolDir		需要检测的文件目录
fileSuffix		同步完成标识的后缀名(默认值为.COMPLETED)
deletePolicy	是否删除同步完成的文件，只有两个可选值never（从不删除，默认）或immediate（立即删除）
fileHeader		是否把路径加入到Heander(默认值为false)
fileHeaderKey	路径加入到Header的Key是什么(默认值为file)
basenameHeader	是否把文件名加入到Heander
basenameHeaderKey	文件名加入到Header的Key是什么(默认值为basename)
ignorePattern	不同步文件的正则表达式(默认值为 ^$)
trackerDir		被处理文件的元数据的存储目录，如果不是绝对路径，就被会解析到spoolDir目录下(默认值为.flumespool)
batchSize		批量写入Channel端的大小(默认值为100)
inputCharset	输出字符集(默认值为UTF-8)
deserializer	解串器类型，日志文件需要使用LINE(默认)，大二进制文件如：pdf、图片采用BLOB
```

<br>



Syslog TCP/UDP Source

```default
Syslog Source为系统日志采集源，这个源可以作为Syslog服务器来接收syslog客户机所产生的syslog的内容，这种源主要有三个源，分别问TCP系统日志源，多端口TCP系统日志源，和UDP系统日志源。

Syslog TCP Source
用来接收TCP传入的Syslog的源，具体配置方式如下：
a1.sources = r1
a1.channels = c1
a1.sources.r1.type = syslogtcp
a1.sources.r1.port = 5140
a1.sources.r1.host = localhost
a1.sources.r1.channels = c1

配置属性如下(必要参数为#标识)：
#channels		当前源中的数据通过哪一频道进行下一步传输
#type			当前源类型的名称，必须填写为syslogtcp
#host			绑定到哪个IP地址
#port			绑定到哪个端口号
eventSize		每一个事件的最大大小，单位b(默认值为2500)


Multiport Syslog TCP Source
这个是可以监听多个端口的TCP Syslog源,具体的配置方式如下：
a1.sources = r1
a1.channels = c1
a1.sources.r1.type = multiport_syslogtcp
a1.sources.r1.channels = c1
a1.sources.r1.host = 0.0.0.0
a1.sources.r1.ports = 10001 10002 10003
a1.sources.r1.portHeader = port

配置属性如下(必要参数为#标识)：
#channels		当前源中的数据通过哪一频道进行下一步传输
#type			当前源类型的名称，必须填写为multiport_syslogtcp
#host			绑定到哪个IP地址
#ports			绑定到哪些端口号
eventSize		每一个事件的最大大小，单位b(默认值为2500)
portHeader		将事件来源的端口号加入事件头时所使用的key
charset.default	默认输出字符集(UTF-8)
charset.port.<port>	每一个端口的默认输出字符集


Syslog UDP Source
用来接收通过UDP协议所发送过来的日志数据，具体的配置方式如下：
a1.sources = r1
a1.channels = c1
a1.sources.r1.type = syslogudp
a1.sources.r1.port = 5140
a1.sources.r1.host = localhost
a1.sources.r1.channels = c1

配置属性如下(必要参数为#标识)：
#channels		当前源中的数据通过哪一频道进行下一步传输
#type			当前源类型的名称，必须填写为syslogudp
#host			绑定到哪个IP地址
#port			绑定到哪些端口号
```

<br>



### 常见的sink组件

hdfs sink

```default
Sink配置参数说明（必要参数为#标识）：
#channel		传输的channel
#type			值必须为hdfs
#hdfs.path		写入hdfs的路径，如 hdfs://ns1/user/ubuntu/flume/tomcat-logs/%y%m%d
hdfs.filePrefix		写入hdfs的文件名前缀，可以使用日期及%{host}表达式 (默认值为FlumeData)
hdfs.fileSuffix		写入hdfs的文件名后缀，可使用比如：.lzo .log等。
hdfs.inUsePrefix	临时文件的文件名前缀，hdfs sink会先往目标目录中写临时文件，再重命名成最终目标文件
hdfs.inUseSuffix	临时文件的文件名后缀(默认值为 .tmp)
					注意，目标文件的前缀和后缀中间的.timestamp不能去掉！

#参数rollInterval、rollSize、rollCount的含义是逻辑”或“的关系
hdfs.rollInterval	间隔多少秒将临时文件滚动成最终目标文件(默认为30)，0表示不根据时间来滚动文件
hdfs.rollSize	当临时文件达到该大小bytes（默认值为1024）时滚动成目标文件，0表示不根据该值来滚动文件
hdfs.rollCount	当events数据达到该数量时候，将临时文件滚动成目标文件，0表示不根据events数量来滚动文件
				注：source的日志实时写入到sink的临时文件中，滚动roll指的是sink将临时文件重命名成
				目标文件，并新打开一个临时文件来写入数据
hdfs.idleTimeout 当前被打开的临时文件在多少秒内（默认0）无数据写入，则将该临时文件关闭并重命名成目标文件
hdfs.batchSize	每个批次刷新到HDFS上的events数量(默认值为100)

hdfs.codeC		文件压缩格式，包括：gzip, bzip2, lzo, lzop, snappy
hdfs.fileType	文件格式，包括：SequenceFile, DataStream,CompressedStream(默认SequenceFile)
				当使用DataStream时候，文件不会被压缩，不需要设置hdfs.codeC;
				当使用CompressedStream时候，必须设置一个正确的hdfs.codeC值
hdfs.writeFormat	写sequence文件的格式。包含：Text, Writable（默认为Writable）
hdfs.maxOpenFiles	最大允许打开的HDFS文件数（默认5000），当打开文件数达到该值，最早打开的文件将被关闭
hdfs.minBlockReplicas 写入HDFS文件块的最小副本数(默认值为HDFS副本数)
					注意：该参数会影响文件的滚动配置，一般将该参数配置成1，才可以按照配置正确滚动文件。
hdfs.callTimeout	执行HDFS操作的超时时间，单位毫秒(默认值为10000)
hdfs.threadsPoolSize 操作HDFS的线程数(默认值为10)
hdfs.rollTimerPoolSize 滚动文件的线程数(默认值为10)

hdfs.kerberosPrincipal	HDFS安全认证kerberos配置
hdfs.kerberosKeytab		HDFS安全认证kerberos配置
hdfs.proxyUser			代理用户

hdfs.round		是否启用时间上的”舍弃”（类似于”四舍五入”）。如果启用，则会影响除了%t的其他所有时间表达式
hdfs.roundValue	时间上进行“舍弃”的值(默认值为1)
hdfs.roundUnit	时间上进行”舍弃”的单位，包含：second,minute,hour(默认为second)
				注：上述配置配合%y-%m-%d/%H%M/%S形式，可以设置每多长时间单位生成一个文件夹或文件

hdfs.timeZone	时区，例如America/Los_Angeles (默认值为Local Time)
hdfs.useLocalTimeStamp	是否使用当地时间 (默认值为flase)
hdfs.closeTries	是否关闭文件的尝试次数
				0表示当一次关闭失败后，hdfs sink会继续尝试下一次关闭，直到成功(默认)；
				1表示当一次关闭文件失败后，hdfs sink将不会再次尝试关闭文件，
				这个未关闭的文件将会一直留在那，并且是打开状态
hdfs.retryInterval	hdfs sink尝试关闭文件的时间间隔，单位秒(默认值为180)
					如果设置为0，表示不尝试，相当于于将hdfs.closeTries设置成1。
				
serializer	序列化类型（默认TEXT）。其他还有avro_event或者是实现了EventSerializer.Builder的类名。
```

<br>



### flume的安装和配置

```shell
#1.下载(以flume-ng-1.5.0-cdh5.3.6为例)
$ wget http://archive.cloudera.com/cdh5/cdh/5/flume-ng-1.5.0-cdh5.3.6.tar.gz

#2.解压和jar包准备(如果需要将数据sink到hdfs则需要将hadoop相关jar包移到flume的lib文件夹中)
# 注意：如果hadoop配置了环境变量HADDOP_HOME，则可以不用考虑hadoop jar包的准备
$ tar -zxvf flume-ng-1.5.0-cdh5.3.6.tar.gz -C /opt/module
#hadoop的jar包准备
commons-configuration-1.6.jar
hadoop-auth-2.5.0-cdh5.3.6.jar
hadoop-common-2.5.0-chd5.3.6.jar
hadoop-hdfs-2.5.0-chd5.3.6.jar

#3.配置flume-env.sh
$ vim conf/flume-env.sh
	export JAVA_HOME=/opt/tool/jdk1.8.0_144
	
#4.配置一个简单的flume agent
$ cp conf/flume-conf.properties.template flume-conf-01.properties
$ vim conf/flume-conf-01.properties
-------------------------------------
# define agents, one of them named a1
a1.sources = r1
a1.channels = c1
a1.sinks = k1

# define sources, one of them named r1
a1.sources.r1.type = netcat
a1.sources.r1.bind = nimbusz
a1.sources.r1.port = 44444

# define channels, one of them named c1
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

# define sinks, one of them named k1
a1.sinks.k1.type = logger
a1.sinls.k1.maxBytesToLog = 1024

# bind sources and sink to channel
a1.sources.r1.channels = c1
a.sinks.k1.channel = c1
  
#5.测试flume agent是否正常
# 需要netcat、telnet的软件支持
# 分别打开两个linux shell窗口，分别用于netcat在44444端口发送数据 和flume sink数据的显示
$ nc -nimbusz 44444
$ bin/flume-ng agent --name a1 --conf conf --conf-file conf/flume-conf-01.properties -Dflume.root.logger=INFO,console
```

<br>



### 使用exec source将日志文件实时[t+0]抽取到 HDFS-HA中

```shell
#1.配置flume agent
-------------------------------------
# define agents, one of them named a2
a2.sources = r1
a2.channels = c1
a2.sinks = k1


# define sources, one of them named r1
a2.sources.r1.type = exec
a2.sources.r1.command = tail -f /opt/app/apache-tomcat-7.0.82/logs/catalina.out
a2.sources.r1.shell = /bin/bash -c
a2.sources.r1.batchTimeout = 1000
a2.sources.r1.batchSize = 20


# define channels, one of them named c1
# MemoryTransaction会根据事务容量transCapacity创建两个阻塞双端队列（LinkedBlockingDeque）putList和takeList，队列的容量为transCapacity，这两个队列主要就是用于事务处理的。当从Source往 Channel中放事件event 时，会先将event放入 putList 队列（相当于一个临时缓冲队列），然后将putList队列中的event 放入 MemoryChannel的queue中；当从 Channel 中将数据传送给 Sink 时，则会将event先放入 takeList 队列中，然后从takeList队列中将event送入Sink。不论是 put 还是 take 发生异常，都会调用 rollback 方法回滚事务，先给 Channel 加锁防止回滚时有其他线程访问，若takeList 不为空就将写入 takeList中的event再次放入 Channel 中，然后移除 putList 中的所有event（即就是丢弃写入putList临时队列的 event）。
a2.channels.c1.type = memory
a2.channels.c1.capacity = 2000
a2.channels.c1.transactionCapacity = 1000


# define sinks, one of them named k1
a2.sinks.k1.type = hdfs  
a2.sinks.k1.hdfs.path = hdfs://ns1/user/ubuntu/flume/tomcat-logs/%Y%m%d
a2.sinks.k1.hdfs.minBlockReplicas = 1
a2.sinks.k1.hdfs.useLocalTimeStamp = true
a2.sinks.k1.hdfs.round = false
a2.sinks.k1.hdfs.filePrefix = catlina%H
a2.sinks.k1.hdfs.rollInterval = 3600
a2.sinks.k1.hdfs.rollCount = 0
a2.sinks.k1.hdfs.rollSize = 134217728
a2.sinks.k1.hdfs.batchSize = 1000
a2.sinks.k1.hdfs.fileType = DataStream
a2.sinks.k1.hdfs.writeFormat = Text


# bind sources and sink to channel
a2.sources.r1.channels = c1
a2.sinks.k1.channel = c1
-------------------------------------


#2.将hadoop的core-site.xml和hdfs-site.xml文件拷贝到flume的conf文件夹下
cp hadoop-2.5.0/etc/hadoop/core-site.xml apache-flume-1.5.0-cdh5.3.6-bin/conf/
cp hadoop-2.5.0/etc/hadoop/hdfs-site.xml apache-flume-1.5.0-cdh5.3.6-bin/conf/


#3.启动flume agent
$ bin/flume-ng agent --name a2 --conf conf --conf-file conf/flume-conf-02.properties

#4.测试(访问tomcat app将产生日志信息，flume将收集tomcat产生的日志到hdfs的/user/ubuntu/flume/tomcat-logs文件夹)
$ tail -f logs/flume.log
$ hdfs dfs -ls /user/ubuntu/flume/tomcat-logs
```

> 关于flume中涉及到时间戳的错误解决，Expected timestamp in the Flume even：
> 在搭建flume集群收集日志写入hdfs时发生了下面的错误：
> java.lang.NullPointerException: Expected timestamp in the Flume event headers, but it was null
>
> 原因是因为写入到hfds时使用到了时间戳来区分目录结构，flume的消息组件event在接受到之后在header中没有发现时间戳参数，导致该错误发生，有三种方法可以解决这个错误：
>
> ```default
> 1、a2.sources.r1.interceptors = t1
>    a2.sources.r1.interceptors.t1.type = timestamp
>    为source添加拦截，每条event头中加入时间戳（效率会慢一些）
>
> 2、a2.sinks.k1.hdfs.useLocalTimeStamp = true
>    为sink指定该参数为true（如果客户端和flume集群时间不一致数据时间会不准确）
>
> 3、在向source发送event时，将时间戳参数添加到event的header中即可，
>    header是一个map，添加时mapkey为timestamp(推荐使用)
> ```

<br>



### 使用Spooling Directory Source将新增日志文件[t+1]抽取到 HDFS-HA中

Spooling Directory Source可以获取硬盘上“spooling”目录的数据，这个Source将监视指定目录是否有新文件，如果有新文件的话，就解析这个新文件。事件的解析逻辑是可插拔的。在文件的内容所有的都读取到Channel之后，Spooling Directory Source会重名或者是删除该文件以表示文件已经读取完成。
不像Exec Source，这个Source是可靠的，且不会丢失数据。即使Flume重启或者被Kill。但是需要注意如下两点:

1. 如果文件在放入spooling目录之后还在写，那么Flume会打印错误日志，并且停止处理该文件。
2. 如果文件之后重复使用，Flume将打印错误日志，并且停止处理。

为了避免以上问题，我们可以使用唯一的标识符来命令文件，例如：时间戳。
尽管这个Source是可靠的，但是如果下游发生故障，也会导致Event重复，这种情况就需要通过Flume的其他组件提供保障。



配置flume agent：

```shell
# define agents, one of them named a3
a3.sources = r1
a3.channels = c1
a3.sinks = k1


# define sources, one of them named r1
a3.sources.r1.type = spooldir
a3.sources.r1.spoolDir = /opt/app/apache-tomcat-7.0.82/logs
a3.sources.r1.ignorePattern = ^(.)*\\.out$
a3.sources.r1.fileSuffix = .COMPLETED
a3.sources.r1.fileHeader = true
a3.sources.r1.fileHeaderKey = fishfile
a3.sources.r1.basenameHeader = true
a3.sources.r1.basenameHeaderKey = fishbasename


# define channels, one of them named c1
# File Channel是一个持久化的隧道（channel），他持久化所有的事件，并将其存储到磁盘中。因此，即使Java 虚拟机当掉，或者操作系统崩溃或重启，再或者事件没有在管道中成功地传递到下一个代理（agent），这一切都不会造成数据丢失。Memory Channel是一个不稳定的隧道，其原因是由于它在内存中存储所有事件。如果java进程死掉，任何存储在内存的事件将会丢失。另外，内存的空间收到RAM大小的限制,而File Channel这方面是它的优势，只要磁盘空间足够，它就可以将所有事件数据存储到磁盘上。
# 注意：capacity的值一定要大于transactionCapacity，不然会报错！
a3.channels.c1.type = file
a3.channels.c1.checkpointDir = /opt/module/apache-flume-1.5.0-cdh5.3.6-bin/spool/chk
a3.channels.c1.dataDirs = /opt/module/apache-flume-1.5.0-cdh5.3.6-bin/spool/data
a3.channels.c1.capacity = 1000000
a3.channels.c1.transactionCapacity = 10000


# define sinks, one of them named k1
a3.sinks.k1.type = hdfs
a3.sinks.k1.hdfs.path = hdfs://ns1/user/ubuntu/flume/tomcat-logs/%Y%m%d
a3.sinks.k1.hdfs.minBlockReplicas = 1
a3.sinks.k1.hdfs.useLocalTimeStamp = true
a3.sinks.k1.hdfs.round = false
a3.sinks.k1.hdfs.filePrefix = catlina%H
a3.sinks.k1.hdfs.rollInterval = 3600
a3.sinks.k1.hdfs.rollCount = 0
a3.sinks.k1.hdfs.rollSize = 134217728
a3.sinks.k1.hdfs.batchSize = 1000
a3.sinks.k1.hdfs.fileType = DataStream
a3.sinks.k1.hdfs.writeFormat = Text


# bind sources and sink to channel
a3.sources.r1.channels = c1
a3.sinks.k1.channel = c1
```

