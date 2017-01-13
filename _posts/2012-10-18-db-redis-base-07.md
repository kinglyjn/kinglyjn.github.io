---
layout: post
title:  "redis基础简介（七）- 主从复制（master & slave）"
desc: "redis基础简介（七）- 主从复制（master & slave）"
keywords: "redis基础简介,主从复制,master,slave,redis,kinglyjn"
date: 2012-10-18
categories: [DB]
tags: [redis]
icon: fa-database
---

### 简介

redis主从复制配置和使用都非常的简单。通过主从复制可以允许多个 slave 拥有和 master 相同的数据库副本。<br>

redis主从复制的特点：<br>

1. master可以拥有多个slave
2. 多个 slave 除了可以连接同一个master之外，还可以连接其他slave，当master宕机之后，可以用该slave再次充当master
3. 主从复制不会阻塞master，在同步数据时，master可以继续处理client请求
4. redis的主从复制能够提高系统的伸缩性

下面是redis主从复制的一般架构设计：<br>

<img src="http://img.blog.csdn.net/20170113142502041?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:70%"/><br>

<br>

### 主从复制的过程：<br>

1. slave 与 master 建立连接，发送sync同步命令
2. master 会启动一个后台进程，将数据库快照保存到文件中，同时 master 主进程会开始收集新的写命令并缓存
3. 后台完成保存后，将此文件发送给 slave
4. slave 将此文件保存到硬盘上

<br>

### 主从复制的配置

配置 slave 服务器，只需在 slave 的配置文件中加入以下配置：<br>

```shell
slaveof 192.168.1.1 6379 #指定要同步的master的主机和端口
masterauth xxxxxx #如果主机加了密码，则指定master服务主机的密码
```
<br>

### 验证是否设置成功

我们在主数据库上设置一对键值对：<br>

```shell
redis 127.0.0.1:6379> set name zhangsan
OK
redis 127.0.0.1:6379> 
```

在从服务器上获取这个键：<br>

```shell
redis 127.0.0.1:6379> get name
"zhangsan"
redis 127.0.0.1:6379>
```
<br>

那么如果事先我们对redis主从集群的情况不甚了解，我们是如何区分哪个是主，哪个是从的呢？<br>
别着急，我们只需要使用 info 命令就可以得到主从的信息，比如我们在从库上执行info：<br>

<img src="http://img.blog.csdn.net/20170113142353811?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:60%"/><br>