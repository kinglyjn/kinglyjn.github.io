---
layout: post
title:  "redis基础简介（四）- 消息的发布订阅"
desc: "redis基础简介（四）- 消息的发布订阅"
keywords: "redis基础简介（四）,消息的大部订阅,redis,kinglyjn"
date: 2012-10-18
categories: [DB]
tags: [redis]
icon: fa-database
---

### 订阅发布通信模式

* Redis 发布订阅(pub/sub)是一种消息通信模式：发送者(pub)发送消息，订阅者(sub)接收消息。
* Redis 客户端可以订阅任意数量的频道。

<br>
下图展示了频道 channel1，以及订阅这个频道的三个客户端 —— client2 、 client5 和 client1 之间的关系：<br>

<img src="http://img.blog.csdn.net/20170103141242087?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:30%"/><br>

当有新消息通过 PUBLISH 命令发送给频道 channel1 时， 这个消息就会被发送给订阅它的三个客户端：<br>

<img src="http://img.blog.csdn.net/20170103141334267?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:30%"/><br>

以下实例演示了发布订阅是如何工作的。在我们实例中我们创建了订阅频道名为 redisChat:<br>

```shell
redis 127.0.0.1:6379> subscribe redisChat
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "redisChat"
3) (integer) 1
```

现在，我们先重新开启个 redis 客户端，然后在同一个频道 redisChat 发布两次消息，订阅者就能接收到消息。<br>

```shell
redis 127.0.0.1:6379> publish redisChat "Redis is a great caching technique"
(integer) 1

redis 127.0.0.1:6379> publish redisChat "Learn redis by runoob.com"
(integer) 1

# 订阅者的客户端会显示如下消息
1) "message"
2) "redisChat"
3) "Redis is a great caching technique"
1) "message"
2) "redisChat"
3) "Learn redis by runoob.com"
```
<br>


### 常用Redis发布订阅命令

下表列出了 redis 发布订阅常用命令：

```shell
# 将信息发送到指定的频道。
# 时间复杂度：O(N+M)，其中 N 是频道 channel 的订阅者数量，而 M 则是使用模式订阅(subscribed patterns)的客户端的数量。
# 返回值：接收到信息 message 的订阅者数量。
publish channel message 


# 订阅给定的一个或多个频道的信息。
# 时间复杂度：O(N)，其中 N 是订阅的频道的数量。
# 返回值：接收到的信息
subscribe channel [channel ...] 


# 指退订给定的频道。
# 时间复杂度：O(N)，其中 N 是订阅的频道的数量。
# 返回值：这个命令在不同的客户端中有不同的表现。
unsubscibe [channel [channel ...]] 


# 订阅一个或多个符合给定模式的频道。
# 每个模式以 * 作为匹配符，比如 it* 匹配所有以 it 开头的频道( it.news 、 it.blog 、 it.tweets 等等)， 
# news.* 匹配所有以 news. 开头的频道( news.it 、 news.global.today 等等)，诸如此类。
# 时间复杂度：O(N)， N 是订阅的模式的数量。
# 返回值：接收到的信息
psubscribe pattern [pattern ...] 


# 退订所有给定模式的频道。
# 如果没有模式被指定，也即是，一个无参数的 PUNSUBSCRIBE调用被执行，
# 那么客户端使用 PSUBSCRIBE 命令订阅的所有模式都会被退订。
# 在这种情况下，命令会返回一个信息，告知客户端所有被退订的模式。
# 这个命令在不同的客户端中有不同的表现。
punsubscribe [pattern [pattern ...]] 



# 查看订阅与发布系统状态。 
# 可用版本： >= 2.8.0
pubsub channels news.i*   #打印所有符合模式的活跃频道
1) "news.internet"
2) "news.it"

pubsub numsub news.it news.internet news.sport news.music #打印各个频道的订阅者数量
1) "news.it"    # 频道
2) "2"          # 订阅该频道的客户端数量
3) "news.internet"
4) "1"
5) "news.sport"
6) "1"
7) "news.music" # 没有任何订阅者
8) "0"

pubsub numpat #计算被订阅模式的数量，注意当有多个客户端订阅相同的模式时，相同的订阅也被计算在PUBSUB NUMPAT之内
(integer) 4
```





