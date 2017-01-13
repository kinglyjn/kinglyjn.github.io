---
layout: post
title:  "redis基础简介（五）- 数据备份与恢复、数据安全、性能测试、客户端连接、分区"
desc: "redis基础简介（五）- 数据备份与恢复、数据安全、性能测试、客户端连接、分区"
keywords: "redis基础简介,数据备份与恢复,数据安全,性能测试,客户端连接,分区,redis,kinglyjn"
date: 2012-10-18
categories: [DB]
tags: [redis]
icon: fa-database
---

### 数据备份与恢复

`数据备份` <br>
redis save 命令用于创建当前数据库的备份。

```shell
redis 127.0.0.1:6379> SAVE 
OK
```
该命令将在 redis 安装目录中创建dump.rdb文件。<br>

创建 redis 备份文件也可以使用命令 BGSAVE，该命令在后台执行。<br>

```shell
127.0.0.1:6379> bgsave
Background saving started
```

`数据恢复` <br>
如果需要恢复数据，只需将备份文件 (dump.rdb) 移动到 redis 安装目录并启动服务即可。<br>
获取 redis 目录可以使用 CONFIG 命令，如下所示：<br>

```shell
127.0.0.1:6379> config get dir
1) "dir"
2) "/var/lib/redis"
```
以上命令 config get dir 输出的 redis 安装目录为 /var/lib/redis。<br>
注意：如果redis最后save操作（同步当前数据到dump.rdb）后与数据恢复文件dump.rdb不一致，则会导致redis重启失败，所以要想redis能够正常重启，必须保持redis的数据一致性。<br>
<br>


### redis安全

我们可以通过 redis 的配置文件设置密码参数，这样客户端连接到 redis 服务就需要密码验证，这样可以让你的 redis 服务更安全。
<br>

我们可以通过以下命令查看是否设置了密码验证：<br>

```shell
127.0.0.1:6379> CONFIG get requirepass
1) "requirepass"
2) ""
```

默认情况下 requirepass 参数是空的，这就意味着你无需通过密码验证就可以连接到 redis 服务。
你可以通过以下命令来修改该参数：<br>

```shell
127.0.0.1:6379> config set requirepass "mypass"
OK
127.0.0.1:6379> config get requirepass
1) "requirepass"
2) "mypass"
```

或者直接修改配置文件redis.conf，这样密码设置就会永久生效：<br>

```shell
# 登录redis客户端可以指定密码（-a 即auth）
$ redis-cli -a xxxpassxxx

# 或者先开启客户端，再授权
$ redis-cli
127.0.0.1:6379> auth "mypass"
```


设置密码后，客户端连接 redis 服务就需要密码验证，否则无法执行命令。格式如下: <br>

```shell
127.0.0.1:6379> auth "mypass"
```
<br>


### Redis性能测试

Redis 性能测试是通过同时执行多个命令实现的。<br>

语法：redis-benchmark [option] [option value] <br>

以下实例同时执行 10000 个请求来检测性能：<br>

```shell
redis-benchmark -n 10000

PING_INLINE: 141043.72 requests per second
PING_BULK: 142857.14 requests per second
SET: 141442.72 requests per second
GET: 145348.83 requests per second
INCR: 137362.64 requests per second
LPUSH: 145348.83 requests per second
LPOP: 146198.83 requests per second
SADD: 146198.83 requests per second
SPOP: 149253.73 requests per second
LPUSH (needed to benchmark LRANGE): 148588.42 requests per second
LRANGE_100 (first 100 elements): 58411.21 requests per second
LRANGE_300 (first 300 elements): 21195.42 requests per second
LRANGE_500 (first 450 elements): 14539.11 requests per second
LRANGE_600 (first 600 elements): 10504.20 requests per second
MSET (10 keys): 93283.58 requests per second
```

redis 性能测试工具可选参数如下所示：<br>

```shell
-h	    指定服务器主机名，默认值 127.0.0.1
-p	    指定服务器端口，默认值 6379
-s	    指定服务器socket	
-c	    指定并发连接数，默认值 50
-n	    指定请求数，默认值 10000
-d	    以字节的形式指定SET/GET值的数据大小，默认值 2
-k	    1=keep alive 0=reconnect，默认值 1
-r	    SET/GET/INCR 使用随机 key, SADD 使用随机值	
-P	    通过管道传输 <numreq> 请求	，默认值 1
-q	    强制退出 redis。仅显示query/sec值	
--csv       以 CSV 格式输出
-l	    生成循环，永久执行测试	
-t	    仅运行以逗号分隔的测试命令列表。	
-I	    Idle 模式。仅打开 N 个 idle 连接并等待。	
```

以下实例我们使用了多个参数来测试 redis 性能：<br>

```shell
redis-benchmark -h 127.0.0.1 -p 6379 -t set,lpush -n 10000 -q

SET: 146198.83 requests per second
LPUSH: 145560.41 requests per second
```

以上实例中主机为 127.0.0.1，端口号为 6379，执行的命令为 set,lpush，请求数为 10000，通过 -q 参数让结果只显示每秒执行的请求数。<br>

<br>

### redis客户端连接

Redis 通过监听一个 TCP 端口或者 Unix socket 的方式来接收来自客户端的连接，当一个连接建立后，Redis 内部会进行以下一些操作：<br>

* 首先，客户端socket会被设置为非阻塞模式，因为Redis在网络事件处理上采用的是非阻塞多路复用模型。
* 然后为这个socket设置TCP_NODELAY属性，禁用Nagle算法
* 然后创建一个可读的文件事件用于监听这个客户端socket的数据发送


`最大连接数` <br>

在 Redis2.4 中，最大连接数是被直接硬编码在代码里面的，而在2.6版本中这个值变成可配置的。
maxclients 的默认值是 10000，你也可以在 redis.conf 中对这个值进行修改。<br>

```shell
config get maxclients
1) "maxclients"
2) "10000"
```
注意：如果当前计算机的能力不能够同时被10000个客户端连接，那么，config get maxclients得到的就不是redis.conf中默认的10000，而是实际的3984（可能值）。<br>

`客户端连接命令` <br>

```shell
client list        返回连接到 redis 服务的客户端列表
client setname     设置当前连接的名称
client getname     获取通过 CLIENT SETNAME 命令设置的服务名称
client pause       挂起客户端连接，指定挂起的时间以毫秒计
client kill        关闭客户端连接
```
<br>


### Redis分区

分区是分割数据到多个Redis实例的处理过程，因此每个实例只保存key的一个子集。<br>

分区的优势：<br>

* 通过利用多台计算机内存的和值，允许我们构造更大的数据库。
* 通过多核和多台计算机，允许我们扩展计算能力；通过多台计算机和网络适配器，允许我们扩展网络带宽。

分区的不足：<br>

* 涉及多个key的操作通常是不被支持的。举例来说，当两个set映射到不同的redis实例上时，你就不能对这两个set执行交集操作。
* 涉及多个key的redis事务不能使用。
* 当使用分区时，数据处理较为复杂，比如你需要处理多个rdb/aof文件，并且从多个实例和主机备份持久化文件。
* 增加或删除容量也比较复杂。redis集群大多数支持在运行时增加、删除节点的透明数据平衡的能力，但是类似于客户端分区、代理等其他系统则不支持这项特性。然而，一种叫做presharding的技术对此是有帮助的。

分区类型：<br>

Redis 有两种类型分区。 假设有4个Redis实例 R0，R1，R2，R3，和类似user:1，user:2这样的表示用户的多个key，对既定的key有多种不同方式来选择这个key存放在哪个实例中。也就是说，有不同的系统来映射某个key到某个Redis服务。<br>

* 范围分区：最简单的分区方式是按范围分区，就是映射一定范围的对象到特定的Redis实例。比如，ID从0到10000的用户会保存到实例R0，ID从10001到 20000的用户会保存到R1，以此类推。这种方式是可行的，并且在实际中使用，不足就是要有一个区间范围到实例的映射表。这个表要被管理，同时还需要各 种对象的映射表，通常对Redis来说并非是好的方法。
* 哈希分区：用一个hash函数将key转换为一个数字，比如使用crc32 hash函数。对key foobar执行crc32(foobar)会输出类似93024922的整数。对这个整数取模，将其转化为0-3之间的数字，就可以将这个整数映射到4个Redis实例中的一个了。93024922 % 4 = 2，就是说key foobar应该被存到R2实例中。注意：取模操作是取除的余数，通常在多种编程语言中用%操作符实现。





