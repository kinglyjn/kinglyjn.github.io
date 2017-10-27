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



### 使用tail -f 命令的形式将tomcat的日志文件实时抽取到 HDFS-HA中

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
#agent source拉取频率，单位为ms
a2.sources.r1.batchTimeout = 3000


# define channels, one of them named c1
a2.channels.c1.type = memory
a2.channels.c1.capacity = 100
a2.channels.c1.transactionCapacity = 100


# define sinks, one of them named k1
a2.sinks.k1.type = hdfs  
a2.sinks.k1.hdfs.path = hdfs://ns1/user/ubuntu/flume/tomcat-logs/%y-%m-%d
a2.sinks.k1.hdfs.useLocalTimeStamp = true
a2.sinks.k1.hdfs.round = true
a1.sinks.k1.hdfs.filePrefix = catlina-
a1.sinks.k1.hdfs.fileSuffix = .log
a2.sinks.k1.hdfs.fileType = DataStream
a2.sinks.k1.hdfs.writeFormat = Text
# 每个批次刷新到HDFS上的events数量
a2.sinks.k1.hdfs.batchSize = 100
# 以下参数含义是逻辑”或“的关系
# 当event个数达到该数量时候，将临时文件滚动成目标文件(0: 不根据event数量来滚动文件)
a2.sinks.k1.hdfs.rollCount = 0
# 间隔多长将临时文件滚动成最终目标文件，单位s (0: 不根据时间间隔来滚动文件)
a2.sinks.k1.hdfs.rollInterval = 3600
# 当临时文件达到该大小时将临时文件滚动成目标文件，单位byte (0: 不根据临时文件大小来滚动文件)
a2.sinks.k1.hdfs.rollSize = 128000000


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
> 1、a2.sources.r1.interceptors = t1
> ​      a2.sources.r1.interceptors.t1.type = timestamp
> ​     为source添加拦截，每条event头中加入时间戳（效率会慢一些）
> 2、a2.sinks.k1.hdfs.useLocalTimeStamp = true 
> ​     为sink指定该参数为true（如果客户端和flume集群时间不一致数据时间会不准确）
> 3、在向source发送event时，将时间戳参数添加到event的header中即可，
> ​      header是一个map，添加时mapkey为timestamp(推荐使用)