---
layout: post
title:  "企业打他数据平台"
desc: "企业打他数据平台"
keywords: "企业打他数据平台,kinglyjn"
date: 2017-08-26
categories: [Java]
tags: [java]
icon: fa-coffee
---



### 集群硬件准备：

测试集群

```default
常见组件的最低内存配置：
zookeeper		2G
namenode		8G-12G（大概每100万个文件需要1G的内存）
datanode & MR	4G-6G
resourcemanager	2G-4G
nodemanager		2G-4G

机器数量：5-10台
机器配置：
	内存：>=32G
	硬盘：>=4TB
	CPU：>=6core（一个物理cpu可以看成两个虚拟cpu，一个i7cpu看成两个i5cpu）
	网卡和网线：>=10万兆	
```

生产集群：

```default
小型机群：
	集群数量：<=20台
	内存：>=128G
	硬盘：>=20TB
	CPU：>=16core
	网卡和网线：>=10万兆
	zookeeper：3-5个节点
	
中型集群：
	集群数量：<=50台
	内存：>=128G
	硬盘：>=20TB
	CPU：>=16core
	网卡和网线：>=10万兆
	zookeeper：5-7个节点
	
大型集群：
	集群数量：>=50台
	内存：>=128G
	硬盘：>=20TB
	CPU：>=16core
	网卡和网线：>=10万兆
	zookeeper：几十上百个节点
```

<br>



### 性能测试

1. 数据读/写效率
2. MR批量任务测试

<br>



 ### hadoop发行版本

目前而言，不收费的Hadoop版本主要有三个（均是国外厂商），分别是：Apache（最原始的版本，所有发行版均基于这个版本进行改进）、Cloudera版本（Cloudera’s Distribution Including Apache Hadoop，简称CDH）、Hortonworks版本(Hortonworks Data Platform，简称“HDP”）。对于国内而言，绝大多数选择CDH版本。

* apache

* cdh

* hdp


<br>

为什么使用CHD而不是apache hadoop?

1. CDH对Hadoop版本的划分非常清晰相比而言，Apache版本则混乱得多；比Apache hadoop在兼容性，安全性，稳定性上有增强。CDH3版本是基于Apache  hadoop  0.20.2改进的，并融入了最新的patch，CDH4版本是基于Apache hadoop 2.X改进的，CDH总是并应用了最新Bug修复或者Feature的Patch，并比Apache hadoop同功能版本提早发布，更新速度比Apache官方快。
2. CDH支持Kerberos安全认证，apache hadoop则使用简陋的用户名匹配认证。
3. CDH文档清晰，很多采用Apache版本的用户都会阅读CDH提供的文档，包括安装文档、升级文档等。
4. CDH支持Yum/Apt包，Tar包，RPM包，Cloudera Manager四种方式安装,Apache hadoop只支持Tar包安装。CDH使用推荐的Yum/Apt包安装时，有以下几个好处：
   1. 联网安装、升级，非常方便
   2. 自动下载依赖软件包
   3. Hadoop生态系统包自动自动创建相关目录并软链到合适的地方（如conf和logs等目录）；自动创建hdfs, mapred用户，hdfs用户是HDFS的最高权限用户，mapred用户则负责mapreduce执行过程中相关目录的权限。
   4. 不需要你寻找与当前Hadoop匹配的Hbase，Flume，Hive等软件，Yum/Apt会根据当前安装Hadoop版本自动寻找匹配版本的软件包，并保证兼容性。

<br>















