---
layout: post
title:  "redis基础简介（八）- redis持久化配置和使用"
desc: "redis基础简介（八）- redis持久化配置和使用"
keywords: "redis基础简介,redis持久化配置和使用,持久化,redis,kinglyjn"
date: 2012-10-18
categories: [DB]
tags: [redis]
icon: fa-database
---

### 简介

Redis是一个可以持久化的内存数据库，也就是说redis需要经常将内存中的数据同步到硬盘来保证持久化。<br>
redis支持两种持久化方式：<br>

1. `snapshotting`（快照），也是默认的方式
2. append-only file （缩写`aof`）的方式

<br>

### snapshotting方式持久化数据

快照方式是redis默认的持久化方式。这种方式是将内存中的数据以快照的形式写入二进制文件中，默认的文件名为dump.rdb。后台保存快照到磁盘会占用大量内存，调用save保存内存中的数据到磁盘，将阻塞客户端请求，直到保存完毕。可以通过配置文件设置自动做快照的持久化方式。调用shutdown命令，Redis服务器会先调用save，所有数据持久化到磁盘之后才会真正退出。修改redis.conf：<br>

```shell
save 900 1 #900s内如果超过1个key被修改，则发起快照保存
save 300 10 #300s内如果超过10个key被修改，则发起快照保存
save 600 10000 #600s内如果超过10000个key被修改，则发起快照保存
```
<br>


### aof方式持久化数据

由于快照方式是在一定的时间间隔内做快照的，所以如果redis意外宕机的话，就会丢失最后一次快照后的所有修改。<br>

aof比快照方式具有更好的持久化性，是由于在使用aof时，redis会将每一个收到的写命令都通过write函数追加到文件中当redis重启时会通过重新执行文件中保存的写命令来在内存中重建整个数据库的内容。aof最大的问题就是随着时间append file会变的很大，所以我们需要 bgrewriteaof 命令重新整理文件，只保留最新的kv数据。<br>

当然由于os会在内核中缓存write做的修改，所以可能不是立即写到磁盘上，这样aof方式的持久化还是有可能丢失部分的修改。<br>

可以通过配置文件（redis.conf）告诉redis，我们想要通过fsync函数强制os将修改写入到磁盘的时机：<br>

```shell
appendonly yes #默认是关闭的，启用aof持久化方式

#appendfsync always  #收到写命令就立即写入磁盘，最慢，但是能够保证完全的持久化
appendfsync everysec #每秒钟写入磁盘一次，在性能和持久化方面做了很好的折中
#appendfsync no      #完全依赖os，性能最好，但持久化没有保证
```
<br>


按照以上方式对redis设置进行修改之后，我们向数据库写入一些数据：<br>

```shell
ubuntu@dbserver:/var/lib/redis$ redis-cli -a xxxxxx
127.0.0.1:6379> keys *
(empty list or set)
127.0.0.1:6379> set name zhangsan
OK
127.0.0.1:6379> hmset user:001 name xiaojuan age 23
OK
127.0.0.1:6379> 
```

然后退出redis客户端，我们将会看到刚才我们的写操作全部记录到appendonly.aof文件中：<br>

```shell
127.0.0.1:6379> exit
ubuntu@dbserver:/var/lib/redis$ ls
appendonly.aof  dump.rdb
ubuntu@dbserver:/var/lib/redis$ sudo cat appendonly.aof 
*2
$6
SELECT
$1
0
*3
$3
set
$4
name
$8
zhangsan
*6
$5
hmset
$8
user:001
$4
name
$8
xiaojuan
$3
age
$2
23
ubuntu@dbserver:/var/lib/redis$ 
```
<br>


### 自动的bgrewriteaof

为了避免aof文件过大，我们会周期性的做bgrewriteaof来重整aof文件。以前我们会额外的配置crontab在业务低峰期执行这个命令，这额外的增加一个workaroud的脚本任务在大集群里是很糟糕的，不易检查，出错无法即时发现。<br>

于是这个自动bgrewriteaof功能被直接加到redis的内部。首先对于aof文件，server对象添加一个字段来记录aof文件的大小server.appendonly_current_size，每次aof发生变化都会维护这个字段。<br>

bgrewriteaof完毕或者实例启动载入aof数据后也会调用aofUpdateCurrentSize这个函数维护这个字段，同时会记录下此时的aof文件的大小server.auto_aofrewrite_base_size作为基准值，用于接下来判断aof增长率。<br>

有了当前值和基准值我们就可以判断aof文件的增长情况。另外还需要配置两个参数来判断是否需要自动触发bgrewriteaof。<br>

auto_aofrewrite_perc: aof文件的大小超过基准百分之多少后触发bgrewriteaof。默认这个值设置为100，意味着当前aof是基准大小的两倍的时候触发bgrewriteaof。把它设置为0可以禁用自动触发的功能。<br>
auto_aofrewrite_min_size: 当前aof文件大于多少字节后才触发。避免在aof较小的时候无谓行为。默认大小为64mb。
两个参数都是可以在conf里静态配置，或者通过config set来动态修改的。<br>

```shell
redis 127.0.0.1:6379> config get auto-aof-rewrite-percentage  
1) "auto-aof-rewrite-percentage"  
2) "100"  
redis 127.0.0.1:6379> config get auto-aof-rewrite-min-size  
1) "auto-aof-rewrite-min-size"  
2) "1048576"  
redis 127.0.0.1:6379> config get auto-aof-rewrite-min-size  
1) "auto-aof-rewrite-min-size"  
2) "1048576"  
redis 127.0.0.1:6379> config set auto-aof-rewrite-percentage 200  
OK  
redis 127.0.0.1:6379> config set auto-aof-rewrite-min-size 10485760  
OK 
```

然后就是触发检查的主逻辑，serverCron时间事件中每次都会检查现有状态和参数来判断是否需要启动bgrewriteaof。<br>
如果aof文件增长百分率growth大于auto_aofrewrite_perc，则自动的触发后一个bgrewriteaof。<br>

> 注：从 Redis 2.4 开始， AOF 重写由 Redis 自行触发， bgrewriteaof 仅仅用于手动触发重写操作。<br>


