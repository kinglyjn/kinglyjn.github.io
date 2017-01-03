---
layout: post
title:  "redis基础简介（三）- 事务"
desc: "redis基础简介（三）- 事务"
keywords: "redis基础简介（三）,事务,redis,kinglyjn"
date: 2012-10-18
categories: [DB]
tags: [redis]
icon: fa-database
---

和众多其它数据库一样，Redis作为NoSQL数据库也同样提供了事务机制。在Redis中，multi/exec/discard/watch 这四个命令是我们实现事务的基石。相信对有关系型数据库开发经验的开发者而言这一概念并不陌生，即便如此，我们还是会简要的列出Redis中事务的实现特征：

1. 在事务中的所有命令都将会被串行化的顺序执行，事务执行期间，Redis不会再为其它客户端的请求提供任何服务，从而保证了事物中的所有命令被原子的执行。
2. 和关系型数据库中的事务相比，在Redis事务中如果有某一条命令执行失败，其后的命令仍然会被继续执行。
3. 我们可以通过multi命令开启一个事务，有关系型数据库开发经验的人可以将其理解为"begin transaction"语句。在该语句之后执行的命令都将被视为事务之内的操作，最后我们可以通过执行exec/discard命令来提交/回滚该事务内的所有操作。这两个Redis命令可被视为等同于关系型数据库中的commit/rollback语句。
4. 在事务开启之前，如果客户端与服务器之间出现通讯故障并导致网络断开，其后所有待执行的语句都将不会被服务器执行。然而如果网络中断事件是发生在客户端执行EXEC命令之后，那么该事务中的所有命令都会被服务器执行。
5. 当使用Append-Only模式时，Redis会通过调用系统函数write将该事务内的所有写操作在本次调用中全部写入磁盘。然而如果在写入的过程中出现系统崩溃，如电源故障导致的宕机，那么此时也许只有部分数据被写入到磁盘，而另外一部分数据却已经丢失。Redis服务器会在重新启动时执行一系列必要的一致性检测，一旦发现类似问题，就会立即退出并给出相应的错误提示。此时，我们就要充分利用Redis工具包中提供的redis-check-aof工具，该工具可以帮助我们定位到数据不一致的错误，并将已经写入的部分数据进行回滚。修复之后我们就可以再次重新启动Redis服务器了。

<b>事务正常执行</b><br>

```shell
#在当前连接上启动一个新的事务。
redis 127.0.0.1:6379> multi
OK

#执行事务中的第一条命令，从该命令的返回结果可以看出，该命令并没有立即执行，而是存于事务的命令队列。
redis 127.0.0.1:6379> incr t1
QUEUED

#又执行一个新的命令，从结果可以看出，该命令也被存于事务的命令队列。
redis 127.0.0.1:6379> incr t2
QUEUED

#执行事务命令队列中的所有命令，从结果可以看出，队列中命令的结果得到返回。
redis 127.0.0.1:6379> exec
1) (integer) 1
2) (integer) 1
```
<br>

<b>事务中存在失败的命令</b><br>

```shell
#开启一个新的事务。
redis 127.0.0.1:6379> multi
OK

#设置键a的值为string类型的3。
redis 127.0.0.1:6379> set a 3
QUEUED

#从键a所关联的值的头部弹出元素，由于该值是字符串类型，而lpop命令仅能用于List类型，因此在执行exec命令时，该命令将会失败。
redis 127.0.0.1:6379> lpop a
QUEUED

#再次设置键a的值为字符串4。
redis 127.0.0.1:6379> set a 4
QUEUED

#获取键a的值，以便确认该值是否被事务中的第二个set命令设置成功。
redis 127.0.0.1:6379> get a
QUEUED

#从结果中可以看出，事务中的第二条命令lpop执行失败，而其后的set和get命令均执行成功，这一点是Redis的事务与关系型数据库中的事务之间最为重要的差别。
redis 127.0.0.1:6379> exec
1) OK
2) (error) ERR Operation against a key holding the wrong kind of value
3) OK
4) "4"
```
<br>

<b>回滚事务</b><br>

```shell
#为键t2设置一个事务执行前的值。
redis 127.0.0.1:6379> set t2 tt
OK
#开启一个事务。
redis 127.0.0.1:6379> multi
OK
#在事务内为该键设置一个新值。
redis 127.0.0.1:6379> set t2 ttnew
QUEUED
#放弃事务。
redis 127.0.0.1:6379> discard
OK
#查看键t2的值，从结果中可以看出该键的值仍为事务开始之前的值。
redis 127.0.0.1:6379> get t2
"tt" 
```
<br>


<b>watch/unwatch命令和基于cas的乐观锁</b><br>

在Redis的事务中，WATCH命令可用于提供CAS(check-and-set)功能。假设我们通过WATCH命令在事务执行之前监控了多个Keys，倘若在WATCH之后有任何Key的值发生了变化，EXEC命令执行的事务都将被放弃，同时返回Null multi-bulk应答以通知调用者事务执行失败。例如，我们再次假设Redis中并未提供incr命令来完成键值的原子性递增，如果要实现该功能，我们只能自行编写相应的代码。其伪码如下：<br>

```default
val = GET mykey
val = val + 1
SET mykey $val
```

以上代码只有在单连接的情况下才可以保证执行结果是正确的，因为如果在同一时刻有多个客户端在同时执行该段代码，那么就会出现多线程程序中经常出现的一种错误场景--竞态争用(race condition)。比如，客户端A和B都在同一时刻读取了mykey的原有值，假设该值为10，此后两个客户端又均将该值加一后set回Redis服务器，这样就会导致mykey的结果为11，而不是我们认为的12。为了解决类似的问题，我们需要借助WATCH命令的帮助，见如下代码：<br>

```default
WATCH mykey
val = GET mykey
val = val + 1
MULTI
SET mykey $val
EXEC
```

和此前代码不同的是，新代码在获取mykey的值之前先通过WATCH命令监控了该键，此后又将set命令包围在事务中，这样就可以有效的保证每个连接在执行EXEC之前，如果当前连接获取的mykey的值被其它连接的客户端修改，那么当前连接的EXEC命令将执行失败。这样调用者在判断返回值后就可以获悉val是否被重新设置成功。<br>

