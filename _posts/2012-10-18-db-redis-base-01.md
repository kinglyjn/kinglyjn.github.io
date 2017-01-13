---
layout: post
title:  "redis基础简介（一）- string（字符串）、list（列表）"
desc: "redis基础简介（一）- string（字符串）、list（列表）"
keywords: "redis基础简介,字符串,string,列表,list,redis,kinglyjn"
date: 2012-10-18
categories: [DB]
tags: [redis]
icon: fa-database
---

### Redis简介

Remote Dictionary Server(Redis) 是一个由Salvatore Sanfilippo写的key-value存储系统。
Redis是一个开源的使用ANSI C语言编写、遵守BSD协议、支持网络、可基于内存亦可持久化的日志型、Key-Value数据库，并提供多种语言的API。它通常被称为数据结构服务器，因为值（value）可以是 字符串(String), 哈希(Map), 列表(list), 集合(sets) 和 有序集合(sorted sets)等类型。<br>

Redis 与其他 key - value 缓存产品有以下三个特点：
* Redis支持数据的持久化，可以将内存中的数据保存在磁盘中，重启的时候可以再次加载进行使用。
* Redis不仅仅支持简单的key-value类型的数据，同时还提供list，set，zset，hash等数据结构的存储。
* Redis支持数据的备份，即master-slave模式的数据备份。


<b>Redis的优势：</b>

* 性能极高 – Redis能读的速度是110000次/s,写的速度是81000次/s 
* 丰富的数据类型 – Redis支持二进制案例的 Strings, Lists, Hashes, Sets 及 Ordered Sets 数据类型操作。
* 原子 – Redis的所有操作都是原子性的，同时Redis还支持对几个操作全并后的原子性执行。
* 丰富的特性 – Redis还支持 publish/subscribe, 通知, key 过期等等特性。
* Redis有着更为复杂的数据结构并且提供对他们的原子性操作，这是一个不同于其他数据库的进化路径。Redis的数据类型都是基于基本数据结构的同时对程序员透明，无需进行额外的抽象。
* Redis运行在内存中但是可以持久化到磁盘，所以在对不同数据集进行高速读写时需要权衡内存，因为数据量不能大于硬件内存。在内存数据库方面的另一个优点是，相比在磁盘上相同的复杂的数据结构，在内存中操作起来非常简单，这样Redis可以做很多内部复杂性很强的事情。同时，在磁盘格式方面他们是紧凑的以追加的方式产生的，因为他们并不需要进行随机访问。


<b>Redis的持久化：</b> <br>

缺省情况下，Redis会参照当前数据库中数据被修改的数量，在达到一定的阈值后会将数据库的快照存储到磁盘上，这一点我们可以通过配置文件来设定该阈值。通常情况下，我们也可以将Redis设定为定时保存。如当有1000个以上的键数据被修改时，Redis将每隔60秒进行一次数据持久化操作。缺省设置为，如果有9个或9个以下数据修改是，Redis将每15分钟持久化一次。<br>

从上面提到的方案中可以看出，如果采用该方式，Redis的运行时效率将会是非常高效的，既每当有新的数据修改发生时，仅仅是内存中的缓存数据发生改变，而这样的改变并不会被立即持久化到磁盘上，从而在绝大多数的修改操作中避免了磁盘IO的发生。然而事情往往是存在其两面性的，在该方法中我们确实得到了效率上的提升，但是却失去了数据可靠性。如果在内存快照被持久化到磁盘之前，Redis所在的服务器出现宕机，那么这些未写入到磁盘的已修改数据都将丢失。为了保证数据的高可靠性，redis还提供了另外一种数据持久化机制，即append模式。如果Redis服务器被配置为该方式，那么每当有数据修改发生时，都会被立即持久化到磁盘。<br>
<br>

<b>常见nosql产品之间的比较</b><br>

1. BerkeleyDB：支持事务及嵌套事务，海量数据存储，在用于存储实时数据方面具有极高的可用价值，不完全免费。
2. MongoDB：独立的运行并提供相关的数据服务,主要适用于高并发的论坛或博客网站,多读少写、数据量大、逻辑关系简单，以及文档数据作为主要数据源等, 和BerkeleyDB一样，该产品的License同为GPL。
3. Redis：可以作为服务程序独立运行于自己的服务器主机,Redis除了Key/Value之外还支持List、Hash、Set和Ordered Set等数据结构, Redis的License是Apache License，就目前而言，它是完全免费。
4. memcached：与redis的比较，memcached只是提供了数据缓存服务，一旦服务器宕机，之前在内存中缓存的数据也将全部消失，因此可以看出memcached没有提供任何形式的数据持久化功能，而Redis则提供了这样的功能,再有就是Redis提供了更为丰富的数据存储结构，如Hash和Set。至于它们的相同点，主要有两个，一是完全免费，再有就是它们的提供的命令形式极为接近。

<br>

### redis的安装、配置、启动、关闭和状态查看

```shell
# ubuntu安装redis
sudo apt-get update
sudo apt-get install redis-server

# 配置redis
/etc/redis/redis.conf

# 启动或重启redis
sudo service redis-server start/restart #默认端口为6379

# 关闭redis
sudo service redis-server stop

# 查看redis状态
sudo service redis-server status

# shell命令行启动Redis客户端程序
$redis-cli  
redis 127.0.0.1:6379 >
redis 127.0.0.1:6379 > ping
PONG # 这说明你已经成功地安装Redis在您的机器上

# 远程服务器上执行redis命令
$ redis-cli -h host -p port -a password

# 在Ubuntu上安装Redis的桌面管理器
# http://redisdesktop.com/download

#选取数据库
#Redis中的数据库是通过数字来进行命名的，缺省情况下打开的数据库为0，默认有16个库（0-15）
#切换redis数据库
redis 127.0.0.1:6379 > select_ 1
```
<br>


redis.conf （redis 2.8.9）<br>

```shell
#daemonize no  默认情况下， redis 不是在后台运行的，如果需要在后台运行，把该项的值更改为 yes
daemonize yes
#  当 redis 在后台运行的时候， Redis 默认会把 pid 文件放在 /var/run/redis.pid ，你可以配置到其他地址。
#  当运行多个 redis 服务时，需要指定不同的 pid 文件和端口
pidfile /var/run/redis_6379.pid
#  指定 redis 运行的端口，默认是 6379
port 6379
#  在高并发的环境中，为避免慢客户端的连接问题，需要设置一个高速后台日志
tcp-backlog 511
#  指定 redis 只接收来自于该 IP 地址的请求，如果不进行设置，那么将处理所有请求
# bind 192.168.1.100 10.0.0.1
# bind 127.0.0.1
#  设置客户端连接时的超时时间，单位为秒。当客户端在这段时间内没有发出任何指令，那么关闭该连接
# 0 是关闭此设置
timeout 0
# TCP keepalive
#  在 Linux 上，指定值（秒）用于发送 ACKs 的时间。注意关闭连接需要双倍的时间。默认为 0 。
tcp-keepalive 0
#  指定日志记录级别，生产环境推荐 notice
# Redis 总共支持四个级别： debug 、 verbose 、 notice 、 warning ，默认为 verbose
# debug     记录很多信息，用于开发和测试
# varbose   有用的信息，不像 debug 会记录那么多
# notice    普通的 verbose ，常用于生产环境
# warning   只有非常重要或者严重的信息会记录到日志
loglevel notice
#  配置 log 文件地址
#  默认值为 stdout ，标准输出，若后台模式会输出到 /dev/null 。
logfile /var/log/redis/redis.log
#  可用数据库数
#  默认值为 16 ，默认数据库为 0 ，数据库范围在 0- （ database-1 ）之间
databases 16
################################ 快照#################################
#  保存数据到磁盘，格式如下 :
#   save <seconds> <changes>
#    指出在多长时间内，有多少次更新操作，就将数据同步到数据文件 rdb 。
#    相当于条件触发抓取快照，这个可以多个条件配合
#    比如默认配置文件中的设置，就设置了三个条件
#   save 900 1  900 秒内至少有 1 个 key 被改变
#   save 300 10  300 秒内至少有 300 个 key 被改变
#   save 60 10000  60 秒内至少有 10000 个 key 被改变
# save 900 1
# save 300 10
# save 60 10000
#  后台存储错误停止写。
stop-writes-on-bgsave-error yes
#  存储至本地数据库时（持久化到 rdb 文件）是否压缩数据，默认为 yes
rdbcompression yes
# RDB 文件的是否直接偶像 chcksum
rdbchecksum yes
#  本地持久化数据库文件名，默认值为 dump.rdb
dbfilename dump.rdb
#  工作目录
#  数据库镜像备份的文件放置的路径。
#  这里的路径跟文件名要分开配置是因为 redis 在进行备份时，先会将当前数据库的状态写入到一个临时文件中，等备份完成，
#  再把该该临时文件替换为上面所指定的文件，而这里的临时文件和上面所配置的备份文件都会放在这个指定的路径当中。
# AOF 文件也会存放在这个目录下面
#  注意这里必须制定一个目录而不是文件
dir /var/lib/redis/
################################# 复制 #################################
#  主从复制 . 设置该数据库为其他数据库的从数据库 .
#  设置当本机为 slav 服务时，设置 master 服务的 IP 地址及端口，在 Redis 启动时，它会自动从 master 进行数据同步
# slaveof <masterip><masterport>
#  当 master 服务设置了密码保护时 ( 用 requirepass 制定的密码 )
# slave 服务连接 master 的密码
# masterauth <master-password>
#  当从库同主机失去连接或者复制正在进行，从机库有两种运行方式：
# 1)  如果 slave-serve-stale-data 设置为 yes( 默认设置 ) ，从库会继续响应客户端的请求
# 2)  如果 slave-serve-stale-data 是指为 no ，出去 INFO 和 SLAVOF 命令之外的任何请求都会返回一个
#     错误 "SYNC with master in progress"
slave-serve-stale-data yes
#  配置 slave 实例是否接受写。写 slave 对存储短暂数据（在同 master 数据同步后可以很容易地被删除）是有用的，但未配置的情况下，客户端写可能会发送问题。
#  从 Redis2.6 后，默认 slave 为 read-only
slaveread-only yes
#  从库会按照一个时间间隔向主库发送 PINGs. 可以通过 repl-ping-slave-period 设置这个时间间隔，默认是 10 秒
# repl-ping-slave-period 10
# repl-timeout  设置主库批量数据传输时间或者 ping 回复时间间隔，默认值是 60 秒
#  一定要确保 repl-timeout 大于 repl-ping-slave-period
# repl-timeout 60
#  在 slave socket 的 SYNC 后禁用 TCP_NODELAY
#  如果选择“ yes ” ,Redis 将使用一个较小的数字 TCP 数据包和更少的带宽将数据发送到 slave ， 但是这可能导致数据发送到 slave 端会有延迟 , 如果是 Linux kernel 的默认配置，会达到 40 毫秒 .
#  如果选择 "no" ，则发送数据到 slave 端的延迟会降低，但将使用更多的带宽用于复制 .
repl-disable-tcp-nodelay no
#  设置复制的后台日志大小。
#  复制的后台日志越大， slave 断开连接及后来可能执行部分复制花的时间就越长。
#  后台日志在至少有一个 slave 连接时，仅仅分配一次。
# repl-backlog-size 1mb
#  在 master 不再连接 slave 后，后台日志将被释放。下面的配置定义从最后一个 slave 断开连接后需要释放的时间（秒）。
# 0 意味着从不释放后台日志
# repl-backlog-ttl 3600
#  如果 master 不能再正常工作，那么会在多个 slave 中，选择优先值最小的一个 slave 提升为 master ，优先值为 0 表示不能提升为 master 。
slave-priority 100
#  如果少于 N 个 slave 连接，且延迟时间 <=M 秒，则 master 可配置停止接受写操作。
#  例如需要至少 3 个 slave 连接，且延迟 <=10 秒的配置：
# min-slaves-to-write 3
# min-slaves-max-lag 10
#  设置 0 为禁用
#   默认 min-slaves-to-write 为 0 （禁用）， min-slaves-max-lag 为 10
################################## 安全 ###################################
#  设置客户端连接后进行任何其他指定前需要使用的密码。
#  警告：因为 redis 速度相当快，所以在一台比较好的服务器下，一个外部的用户可以在一秒钟进行 150K 次的密码尝试，这意味着你需要指定非常非常强大的密码来防止暴力破解
# requirepass foobared
#  命令重命名 .
#  在一个共享环境下可以重命名相对危险的命令。比如把 CONFIG 重名为一个不容易猜测的字符。
#  举例 :
# rename-command CONFIG b840fc02d524045429941cc15f59e41cb7be6c52
#  如果想删除一个命令，直接把它重命名为一个空字符 "" 即可，如下：
# rename-command CONFIG ""
################################### 约束###################################
#设置同一时间最大客户端连接数，默认无限制， 
#Redis 可以同时打开的客户端连接数为 Redis 进程可以打开的最大文件描述符数，
#如果设置  maxclients 0 ，表示不作限制。
#当客户端连接数到达限制时， Redis 会关闭新的连接并向客户端返回 max number of clients reached 错误信息
# maxclients 10000
#  指定 Redis 最大内存限制， Redis 在启动时会把数据加载到内存中，达到最大内存后， Redis 会按照清除策略尝试清除已到期的 Key
#  如果 Redis 依照策略清除后无法提供足够空间，或者策略设置为 ”noeviction” ，则使用更多空间的命令将会报错，例如 SET, LPUSH 等。但仍然可以进行读取操作
#  注意： Redis 新的 vm 机制，会把 Key 存放内存， Value 会存放在 swap 区
#  该选项对 LRU 策略很有用。
# maxmemory 的设置比较适合于把 redis 当作于类似 memcached 的缓存来使用，而不适合当做一个真实的 DB 。
#  当把 Redis 当做一个真实的数据库使用的时候，内存使用将是一个很大的开销
# maxmemory <bytes>
#  当内存达到最大值的时候 Redis 会选择删除哪些数据？有五种方式可供选择
# volatile-lru ->  利用 LRU 算法移除设置过过期时间的 key (LRU: 最近使用  Least RecentlyUsed )
# allkeys-lru ->  利用 LRU 算法移除任何 key
# volatile-random ->  移除设置过过期时间的随机 key
# allkeys->random -> remove a randomkey, any key
# volatile-ttl ->  移除即将过期的 key(minor TTL)
# noeviction ->  不移除任何可以，只是返回一个写错误
#  注意：对于上面的策略，如果没有合适的 key 可以移除，当写的时候 Redis 会返回一个错误
#  默认是 :  volatile-lru
# maxmemory-policy volatile-lru  
# LRU  和  minimal TTL 算法都不是精准的算法，但是相对精确的算法 ( 为了节省内存 ) ，随意你可以选择样本大小进行检测。
# Redis 默认的灰选择 3 个样本进行检测，你可以通过 maxmemory-samples 进行设置
# maxmemory-samples 3
############################## AOF###############################
#  默认情况下， redis 会在后台异步的把数据库镜像备份到磁盘，但是该备份是非常耗时的，而且备份也不能很频繁，如果发生诸如拉闸限电、拔插头等状况，那么将造成比较大范围的数据丢失。
#  所以 redis 提供了另外一种更加高效的数据库备份及灾难恢复方式。
#  开启 append only 模式之后， redis 会把所接收到的每一次写操作请求都追加到 appendonly.aof 文件中，当 redis 重新启动时，会从该文件恢复出之前的状态。
#  但是这样会造成 appendonly.aof 文件过大，所以 redis 还支持了 BGREWRITEAOF 指令，对 appendonly.aof 进行重新整理。
#  你可以同时开启 asynchronous dumps 和  AOF
appendonly no
# AOF 文件名称  ( 默认 : "appendonly.aof")
# appendfilename appendonly.aof
# Redis 支持三种同步 AOF 文件的策略 :
# no:  不进行同步，系统去操作  . Faster.
# always: always 表示每次有写操作都进行同步 . Slow, Safest.
# everysec:  表示对写操作进行累积，每秒同步一次 . Compromise.
#  默认是 "everysec" ，按照速度和安全折中这是最好的。
#  如果想让 Redis 能更高效的运行，你也可以设置为 "no" ，让操作系统决定什么时候去执行
#  或者相反想让数据更安全你也可以设置为 "always"
#  如果不确定就用  "everysec".
# appendfsync always
appendfsync everysec
# appendfsync no
# AOF 策略设置为 always 或者 everysec 时，后台处理进程 ( 后台保存或者 AOF 日志重写 ) 会执行大量的 I/O 操作
#  在某些 Linux 配置中会阻止过长的 fsync() 请求。注意现在没有任何修复，即使 fsync 在另外一个线程进行处理
#  为了减缓这个问题，可以设置下面这个参数 no-appendfsync-on-rewrite
no-appendfsync-on-rewrite no
# AOF  自动重写
#  当 AOF 文件增长到一定大小的时候 Redis 能够调用  BGREWRITEAOF  对日志文件进行重写
#  它是这样工作的： Redis 会记住上次进行些日志后文件的大小 ( 如果从开机以来还没进行过重写，那日子大小在开机的时候确定 )
#  基础大小会同现在的大小进行比较。如果现在的大小比基础大小大制定的百分比，重写功能将启动
#  同时需要指定一个最小大小用于 AOF 重写，这个用于阻止即使文件很小但是增长幅度很大也去重写 AOF 文件的情况
#  设置  percentage 为 0 就关闭这个特性
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
################################ LUASCRIPTING #############################
# 一个 Lua 脚本最长的执行时间为 5000 毫秒（ 5 秒），如果为 0 或负数表示无限执行时间。
lua-time-limit 5000
################################LOW LOG################################
# Redis Slow Log  记录超过特定执行时间的命令。执行时间不包括 I/O 计算比如连接客户端，返回结果等，只是命令执行时间
#  可以通过两个参数设置 slow log ：一个是告诉 Redis 执行超过多少时间被记录的参数 slowlog-log-slower-than( 微妙 ) ，
#  另一个是 slow log 的长度。当一个新命令被记录的时候最早的命令将被从队列中移除
#  下面的时间以微妙为单位，因此 1000000 代表一秒。
#  注意指定一个负数将关闭慢日志，而设置为 0 将强制每个命令都会记录
slowlog-log-slower-than 10000
#  对日志长度没有限制，只是要注意它会消耗内存
#  可以通过  SLOWLOG RESET 回收被慢日志消耗的内存
#  推荐使用默认值 128 ，当慢日志超过 128 时，最先进入队列的记录会被踢出
slowlog-max-len 128
################################  事件通知  #############################
#  当事件发生时， Redis 可以通知 Pub/Sub 客户端。
#  可以在下表中选择 Redis 要通知的事件类型。事件类型由单个字符来标识：
# K     Keyspace 事件，以 _keyspace@<db>_ 的前缀方式发布
# E     Keyevent 事件，以 _keysevent@<db>_ 的前缀方式发布
# g     通用事件（不指定类型），像 DEL, EXPIRE, RENAME, …
# $     String 命令
# s     Set 命令
# h     Hash 命令
# z     有序集合命令
# x     过期事件（每次 key 过期时生成）
# e     清除事件（当 key 在内存被清除时生成）
# A     g$lshzxe 的别称，因此 ”AKE” 意味着所有的事件
# notify-keyspace-events 带一个由 0 到多个字符组成的字符串参数。空字符串意思是通知被禁用。
#  例子：启用 list 和通用事件：
# notify-keyspace-events Elg
#  默认所用的通知被禁用，因为用户通常不需要改特性，并且该特性会有性能损耗。
#  注意如果你不指定至少 K 或 E 之一，不会发送任何事件。
notify-keyspace-events “”
##############################  高级配置  ###############################
#  当 hash 中包含超过指定元素个数并且最大的元素没有超过临界时，
# hash 将以一种特殊的编码方式（大大减少内存使用）来存储，这里可以设置这两个临界值
# Redis Hash 对应 Value 内部实际就是一个 HashMap ，实际这里会有 2 种不同实现，
#  这个 Hash 的成员比较少时 Redis 为了节省内存会采用类似一维数组的方式来紧凑存储，而不会采用真正的 HashMap 结构，对应的 valueredisObject 的 encoding 为 zipmap,
#  当成员数量增大时会自动转成真正的 HashMap, 此时 encoding 为 ht 。
hash-max-zipmap-entries 512
hash-max-zipmap-value 64  
#  和 Hash 一样，多个小的 list 以特定的方式编码来节省空间。
# list 数据类型节点值大小小于多少字节会采用紧凑存储格式。
list-max-ziplist-entries 512
list-max-ziplist-value 64
# set 数据类型内部数据如果全部是数值型，且包含多少节点以下会采用紧凑格式存储。
set-max-intset-entries 512
#  和 hashe 和 list 一样 , 排序的 set 在指定的长度内以指定编码方式存储以节省空间
# zsort 数据类型节点值大小小于多少字节会采用紧凑存储格式。
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
# Redis 将在每 100 毫秒时使用 1 毫秒的 CPU 时间来对 redis 的 hash 表进行重新 hash ，可以降低内存的使用
#  当你的使用场景中，有非常严格的实时性需要，不能够接受 Redis 时不时的对请求有 2 毫秒的延迟的话，把这项配置为 no 。
#  如果没有这么严格的实时性要求，可以设置为 yes ，以便能够尽可能快的释放内存
activerehashing yes
# 客户端的输出缓冲区的限制，因为某种原因客户端从服务器读取数据的速度不够快，
# 可用于强制断开连接（一个常见的原因是一个发布 / 订阅客户端消费消息的速度无法赶上生产它们的速度）。
#  可以三种不同客户端的方式进行设置：
# normal ->  正常客户端
# slave  -> slave 和 MONITOR 客户端
# pubsub ->  至少订阅了一个 pubsub channel 或 pattern 的客户端
#  每个 client-output-buffer-limit 语法 :
# client-output-buffer-limit <class><hard limit> <soft limit> <soft seconds>
#  一旦达到硬限制客户端会立即断开，或者达到软限制并保持达成的指定秒数（连续）。
#  例如，如果硬限制为 32 兆字节和软限制为 16 兆字节 /10 秒，客户端将会立即断开
#  如果输出缓冲区的大小达到 32 兆字节，客户端达到 16 兆字节和连续超过了限制 10 秒，也将断开连接。
#  默认 normal 客户端不做限制，因为他们在一个请求后未要求时（以推的方式）不接收数据，
#  只有异步客户端可能会出现请求数据的速度比它可以读取的速度快的场景。
#  把硬限制和软限制都设置为 0 来禁用该特性
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit slave 256mb 64mb60
client-output-buffer-limit pubsub 32mb 8mb60
# Redis 调用内部函数来执行许多后台任务，如关闭客户端超时的连接，清除过期的 Key ，等等。
#  不是所有的任务都以相同的频率执行，但 Redis 依照指定的“ Hz ”值来执行检查任务。
#  默认情况下，“ Hz ”的被设定为 10 。
#  提高该值将在 Redis 空闲时使用更多的 CPU 时，但同时当有多个 key 同时到期会使 Redis 的反应更灵敏，以及超时可以更精确地处理。
#  范围是 1 到 500 之间，但是值超过 100 通常不是一个好主意。
#  大多数用户应该使用 10 这个预设值，只有在非常低的延迟的情况下有必要提高最大到 100 。
hz 10  
#  当一个子节点重写 AOF 文件时，如果启用下面的选项，则文件每生成 32M 数据进行同步。
aof-rewrite-incremental-fsync yes
```
<br>


redis服务器命令<br>

```shell
redis 127.0.0.1:6379> info

# Server
redis_version:2.8.13
redis_git_sha1:00000000
redis_git_dirty:0
redis_build_id:c2238b38b1edb0e2
redis_mode:standalone
os:Linux 3.5.0-48-generic x86_64
arch_bits:64
multiplexing_api:epoll
gcc_version:4.7.2
process_id:3856
run_id:0e61abd297771de3fe812a3c21027732ac9f41fe
tcp_port:6379
uptime_in_seconds:11554
uptime_in_days:0
hz:10
lru_clock:16651447
config_file:

# Clients
connected_clients:1
client-longest_output_list:0
client-biggest_input_buf:0
blocked_clients:0

# Memory
used_memory:589016
used_memory_human:575.21K
used_memory_rss:2461696
used_memory_peak:667312
used_memory_peak_human:651.67K
used_memory_lua:33792
mem_fragmentation_ratio:4.18
mem_allocator:jemalloc-3.6.0

# Persistence
loading:0
rdb_changes_since_last_save:3
rdb_bgsave_in_progress:0
rdb_last_save_time:1409158561
rdb_last_bgsave_status:ok
rdb_last_bgsave_time_sec:0
rdb_current_bgsave_time_sec:-1
aof_enabled:0
aof_rewrite_in_progress:0
aof_rewrite_scheduled:0
aof_last_rewrite_time_sec:-1
aof_current_rewrite_time_sec:-1
aof_last_bgrewrite_status:ok
aof_last_write_status:ok

# Stats
total_connections_received:24
total_commands_processed:294
instantaneous_ops_per_sec:0
rejected_connections:0
sync_full:0
sync_partial_ok:0
sync_partial_err:0
expired_keys:0
evicted_keys:0
keyspace_hits:41
keyspace_misses:82
pubsub_channels:0
pubsub_patterns:0
latest_fork_usec:264

# Replication
role:master
connected_slaves:0
master_repl_offset:0
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0

# CPU
used_cpu_sys:10.49
used_cpu_user:4.96
used_cpu_sys_children:0.00
used_cpu_user_children:0.01

# Keyspace
db0:keys=94,expires=1,avg_ttl=41638810
db1:keys=1,expires=0,avg_ttl=0
db3:keys=1,expires=0,avg_ttl=0


----------------------------
bgrewriteaof  #异步执行一个 AOF（AppendOnly File）文件重写操作
bgsave  #在后台异步保存当前数据库的数据到磁盘
client kill [ip:port] [ID client-id]  #关闭客户端连接
client list  #获取连接到服务器的客户端连接列表
client getname  #获取连接的名称
client pause timeout  #在指定时间内终止运行来自客户端的命令
client setname connection-name  #设置当前连接的名称
cluster slots  #获取集群节点的映射数组
command  #获取 Redis 命令详情数组
command count  #获取 Redis 命令总数
command getkeys  #获取给定命令的所有键
time  #返回当前服务器时间
command info command-name [command-name ...]  #获取指定 Redis 命令描述的数组
config get parameter  #获取指定配置参数的值   
config rewrite  #对启动 Redis 服务器时所指定的 redis.conf 配置文件进行改写
config set parameter value  #修改 redis 配置参数，无需重启，【注】临时生效，redis重启失效  
config resetstat  #重置 INFO 命令中的某些统计数据
dbsize  #返回当前数据库的 key 的数量
debug object key  #获取 key 的调试信息
debug segfault  #让 Redis 服务崩溃
flushall  #删除所有数据库的所有key
flushdb  #删除当前数据库的所有key
info  #获取 Redis 服务器的各种信息和统计数值 
lastsave  #返回最近一次 Redis 成功将数据保存到磁盘上的时间，以 UNIX 时间戳格式表示
monitor  #实时打印出 Redis 服务器接收到的命令，调试用
role  #返回主从实例所属的角色
save  #异步保存数据到硬盘
shutdown [nosave] [save]  #异步保存数据到硬盘，并关闭服务器
slaveof host port  #将当前服务器转变为指定服务器的从属服务器(slave server)
slowlog subcommand [argument]   #管理 redis 的慢日志
sync  #用于复制功能(replication)的内部命令
```
<br>


### redis的数据类型

* `字符串(Strings)`<br>
字符串是Redis值的最基础的类型。Redis字符串是二进制安全的，这意味着一个Redis字符串可以包含任何种类的数据，例如一个JPEG图像或者一个序列化的Ruby对象。 一个字符串值最多可以保存512M字节的内容。你可以使用Redis的字符串做一些有趣的事情，例如你可以： 
    * 在使用命令INCR系列（ INCR, DECR, INCRBY）命令时将字符串作为的原子计数器。
    * 使用APPEND命令追加字符串。
    * 将字符串作为GETRANGE 和 SETRANGE的随机访问向量。
    * 在小空间里编码大量数据, 或者使用 GETBIT 和 SETBIT创建一个Redis支持的Bloom过滤器。
* `列表(Lists)` <br>
Redis列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素导列表的头部（左边）或者尾部（右边）LPUSH命令插入一个新的元素导头部,而RPUSH插入一个新元素导尾部。当LPUSH或RPUSH操作在一个空的Key上被执行的时候一个新的列表被创建。相似的，如果一个列表操作清空一个列表那么对应的key将被从key空间删除。这是非常方便的语义，因为他们被调用使用一个空列表完全就像他们被调用时使用一个不存在的键值（可以）做为参数。一个列表最多可以包含 2^32-1 个元素 (4294967295, 每个列表超过40亿个元素)。从时间复杂度的角度来看Redis列表的主要特征是在头和尾的元素插入和删除是固定时间，即便是数以百万计的插入。在列表的两端访问元素是非常快的。但是如果你试着访问一个非常大的列表的中间的元素是很慢的，因为那是一个O(N)操作。你可以用Redis列表做很多有趣的事情，比如你可以：
    * 在一个社交网络中建立一个时间线模型，使用LPUSH 去添加新的元素到用户的时间线， 使用LRANGE去接收一些最近插入的元素。
    * 你可以将 LPUSH 和 LTRIM 一起用去创建一个永远也不会超过指定元素数目的列表，但是记住是最后的N个元素。
    * 列表能够被用来作为消息传递的primitive[基元], 例如众所周知的用来创建后台工作的Resque Ruby库。
    * 你可以使用列表做更多的事，这个数据类型支持许多命令，包括像BLPOP这样的阻塞命令。
* `集合(Sets)` <br>
Redis 集合（Set）是一个无序的字符串集合. 你可以以O(1)的时间复杂度 (无论集合中有多少元素时间复杂度都是常量)完成添加，删除，以及测试元素是否存在。redis集合拥有令人满意的不允许包含相同成员的属性。多次添加相同的元素，最终在集合里只会有一个元素。实际上说这些就是意味着在添加元素的时候无须检测元素是否存在。一个Redis集合的非常有趣的事情是他支持一些服务端的命令从现有的集合出发去进行集合运算，因此你可以在非常短的时间内进行合并（unions）, 求交集（intersections）,找出不同的元素（differences of sets）。一个集合最多可以包含 2^32-1 个元素(4294967295, 每个集合超过40亿个元素). 你可以使用集合多很多有趣的事情，比如你能够：
    * 你可以使用集合追踪一件（独一无二的）事情，想要知道所有访问一个博客文章的独立IP?每次当你处理一个页面访问的事简单的使用SADD。你可以肯定重复的IP是不会被插入的。
    * redis 集合是很擅长表现关系的。你可以使用Redis集合创建一个tagging系统去表现每一个tag。接下来你能够使用SADD命令将有一个给定tag的所有对象的所有ID添加到一个用来展现这个特定tag的集合里。你想要同时有三个不同tag的所有对象的ID吗？使用SINTER就好了。
    * 使用 SPOP 或者 SRANDMEMBER 命令你可以使用集合去随意的抽取元素。
* `哈希（Hashes)`<br>
Redis Hashes是字符串字段和字符串值之间的映射,因此他们是展现对象的完美数据类型(例如:一个有名，姓，年龄等等属性的用户)。一个带有一些字段（这里的一些意味着高达一百左右）的hash仅仅需要一块很小的空间存储,因此你可以存储数以百万计的对象在一个小的Redis实例中。 哈希主要用来表现对象，他们有能力存储很多对象，因此你可以将哈希用于许多其他的任务。 每一个哈希可以存储超过2^32-1 字段-值(超过40亿)。
* `有序集合(Sorted Sets)`<br>
Redis有序集合与普通集合非常相似，是一个没有重复元素的字符串集合。不同之处是有序集合的没有成员都关联了一个评分，这个评分被用来按照从最低分到最高分的方式排序集合中的成员。集合的成员是唯一的，但是评分可以是重复了。 使用有序集合你可以以非常快的速度（O(log(N))）添加，删除和更新元素。因为元素是有序的, 所以你也可以很快的根据评分（score）或者次序（position）来获取一个范围的元素。访问有序集合的中间元素也是非常快的,因此你能够使用有序集合作为一个没有重复成员的智能列表。在有序集合中，你可以很快捷的访问一切你需要的东西：有序的元素，快速的存在性测试，快速访问集合的中间元素！简而言之使用有序集合你可以做完成许多对性能有极端要求的任务，而那些任务使用其他类型的数据库真的是很难完成的。 使用有序集合你可以：
    * 在一个大型的在线游戏中展示一个排行榜，在那里一旦一个新的分数被提交，你可以使用ZADD命令去更新它.你也可用使用 ZRANGE命令来得到顶级的用户,你还可以使用ZRANK命令根据用户名返回该用户在排行榜中的位次。同时使用ZRANK 和 ZRANGE 你可以显示和给定用户分数相同的所有用户。所有这些操作都非常的快速。
    * 有序集合常常被用来索引存储在Redis中的数据。举个例子，如果你有许多的哈希（Hashes）来代表用户，你可以使用一个有序集合，这个集合中的元素的年龄字段被用来当做评分，而ID作为值。因此，使用 ZRANGEBYSCORE 命令，那是微不足道的并且能够很快的接收到给定年龄段的所有用户。

<br>

### redis字符串相关操作

keys/set/get/append/strlen/del

```shell
redis 127.0.0.1:6379> keys *                #列出该库中所有的键
redis 127.0.0.1:6379> exists mykey          #判断该键是否存在，存在返回1，否则返回0
redis 127.0.0.1:6379> append mykey "hello"  #该键并不存在，因此append命令返回当前Value的长度
redis 127.0.0.1:6379> append mykey " world" #该键已经存在，因此返回追加后Value的长度
redis 127.0.0.1:6379> get mykey             #通过get命令获取该键，以判断append的结果
redis 127.0.0.1:6379> set mykey "this is a test" #通过set命令为键设置新值，并覆盖原有值
redis 127.0.0.1:6379> strlen mykey          #获取指定Key的字符长度，等效于C库中strlen函数
redis 127.0.0.1:6379> del mykey             #根据键，删除键值
```

incr/decr/incrby/decrby

```shell
redis 127.0.0.1:6379> set mykey 20     #设置Key的值为20
OK
redis 127.0.0.1:6379> incr mykey       #该Key的值递增1
(integer) 21
redis 127.0.0.1:6379> decr mykey       #该Key的值递减1
(integer) 20
redis 127.0.0.1:6379> del mykey        #删除已有键。
(integer) 1
redis 127.0.0.1:6379> decr mykey       #对空值执行递减操作，其原值被设定为0，递减后的值为-1
(integer) -1
redis 127.0.0.1:6379> del mykey   
(integer) 1
redis 127.0.0.1:6379> incr mykey       #对空值执行递增操作，其原值被设定为0，递增后的值为1
(integer) 1
redis 127.0.0.1:6379> set mykey hello  #将该键的Value设置为不能转换为整型的普通字符串。
OK
redis 127.0.0.1:6379> incr mykey       #在该键上再次执行递增操作时，Redis将报告错误信息。
(error) ERR value is not an integer or out of range
redis 127.0.0.1:6379> set mykey 10
OK
redis 127.0.0.1:6379> decrby mykey 5 
(integer) 5
redis 127.0.0.1:6379> incrby mykey 10
(integer) 15
```

getset/setex/setnx

```shell
redis 127.0.0.1:6379> incr mycounter      #将计数器的值原子性的递增1，现在mycounter的值为1
(integer) 1
redis 127.0.0.1:6379> getset mycounter 0  #在获取计数器原有值的同时，并将其设置为新值，这两个操作[原子性地]同时完成。
"1"
redis 127.0.0.1:6379> get mycounter       #查看设置后的结果。
"0"


redis 127.0.0.1:6379> setex mykey 10 "hello"  #设置指定Key的过期时间为10秒。
OK    
redis 127.0.0.1:6379> ttl mykey           #0表示已经过期，-1表示永不过期，-2表示不存在。                   
(integer) 4
redis 127.0.0.1:6379> get mykey           #在该键的存活期内我们仍然可以获取到它的Value。
"hello"
redis 127.0.0.1:6379> ttl mykey           #该ttl命令的返回值显示，该Key已经过期。
(integer) 0
redis 127.0.0.1:6379> get mykey           #获取已过期的Key将返回nil。
(nil)


redis 127.0.0.1:6379> del mykey           #删除该键，以便于下面的测试验证。
(integer) 1
redis 127.0.0.1:6379> setnx mykey "hello" #该键并不存在，因此该命令执行成功。(set if not exists)
(integer) 1
redis 127.0.0.1:6379> setnx mykey "world" #该键已经存在，因此本次设置没有产生任何效果。
(integer) 0
redis 127.0.0.1:6379> get mykey           #从结果可以看出，返回的值仍为第一次设置的值。
"hello"
```

setrange/getrange/setbit/getbit

```shell
redis 127.0.0.1:6379> set mykey "hello world"      #设定初始值。
OK
redis 127.0.0.1:6379> setrange mykey 6 dd          #从第六个字节开始替换2个字节(dd只有2个字节)
(integer) 11
redis 127.0.0.1:6379> get mykey                    #查看替换后的值。
"hello ddrld"
redis 127.0.0.1:6379> setrange mykey 20 dd         #offset已经超过该Key原有值的长度了，该命令将会在末尾补0。
(integer) 22
redis 127.0.0.1:6379> get mykey                    #查看补0后替换的结果。
"hello ddrld\x00\x00\x00\x00\x00\x00\x00\x00\x00dd"
redis 127.0.0.1:6379> del mykey                    #删除该Key。
(integer) 1
redis 127.0.0.1:6379> setrange mykey 2 dd          #替换空值。
(integer) 4
redis 127.0.0.1:6379> get mykey                    #查看替换空值后的结果。
"\x00\x00dd"   
redis 127.0.0.1:6379> set mykey "0123456789"       #设置新值。
OK
redis 127.0.0.1:6379> getrange mykey 1 2           #截取该键的Value，从第一个字节开始，到第二个字节结束。
"12"
redis 127.0.0.1:6379> getrange mykey 1 20          #20已经超过Value的总长度，因此将截取第一个字节后面的所有字节。
"123456789"


redis 127.0.0.1:6379> del mykey
(integer) 1
redis 127.0.0.1:6379> setbit mykey 2 1         #设置从0开始计算的第2位BIT值为1，返回原有BIT值0
(integer) 0
redis 127.0.0.1:6379> get mykey                #获取设置的结果，二进制的0010 0000的十六进制值为0x20,对应的ASCII值为空格
" " 
redis 127.0.0.1:6379> getbit mykey 2           #返回了指定offset的BIT值(上述00100000，返回的是下标2对应的1)。
(integer) 1
redis 127.0.0.1:6379> getbit mykey 10          #offset已经超出了value的长度，因此返回0。
(integer) 0
```

mset/mget/msetnx

```shell
#批量设置了key1和key2两个键。
redis 127.0.0.1:6379> mset key1 "hello" key2 "world"  
OK

#批量获取了key1和key2两个键的值。
redis 127.0.0.1:6379> mget key1 key2 
1) "hello"
2) "world"

#批量设置了key3和key4两个键，因为之前他们并不存在，所以该命令执行成功并返回1。
redis 127.0.0.1:6379> msetnx key3 "stephen" key4 "liu" 
(integer) 1
redis 127.0.0.1:6379> mget key3 key4                   
1) "stephen"
2) "liu"

#批量设置了key3和key5两个键，但是key3已经存在，所以该命令执行失败并返回0。
redis 127.0.0.1:6379> msetnx key3 "hello" key5 "world" 
(integer) 0

#批量获取key3和key5，由于key5没有设置成功，所以返回nil。
redis 127.0.0.1:6379> mget key3 key5                   
1) "stephen"
2) (nil)
```
<br>


### redis列表相关操作

lpush/lpushx/lrange

```shell
redis 127.0.0.1:6379> del mykey
(integer) 1

#mykey键并不存在，该命令会创建该键及与其关联的List，之后在将参数中的values从左到右依次插入。
redis 127.0.0.1:6379> lpush mykey a b c d
(integer) 4

#取从位置0开始到位置2结束的3个元素。
redis 127.0.0.1:6379> lrange mykey 0 2
1) "d"
2) "c"
3) "b"

#取链表中的全部元素，其中0表示第一个元素，-1表示最后一个元素。
redis 127.0.0.1:6379> lrange mykey 0 -1
1) "d"
2) "c"
3) "b"
4) "a"

#mykey2键此时并不存在，因此该命令将不会进行任何操作，其返回值为0。
redis 127.0.0.1:6379> lpushx mykey2 e
(integer) 0

#可以看到mykey2没有关联任何List Value。
redis 127.0.0.1:6379> lrange mykey2 0 -1
(empty list or set)

#mykey键此时已经存在，所以该命令插入成功，并返回链表中当前元素的数量。
redis 127.0.0.1:6379> lpushx mykey e
(integer) 5
#获取该键的List Value的头部元素。
redis 127.0.0.1:6379> lrange mykey 0 0
1) "e"
```

lpop/llen

```shell
redis 127.0.0.1:6379> lpush mykey a b c d
(integer) 4
redis 127.0.0.1:6379> lpop mykey
"d"
redis 127.0.0.1:6379> lpop mykey
"c"

#在执行lpop命令两次后，链表头部的两个元素已经被弹出，此时链表中元素的数量是2
redis 127.0.0.1:6379> llen mykey
(integer) 2
```

lrem/lset/lindex/ltrim

```shell
#为后面的示例准备测试数据。
redis 127.0.0.1:6379> lpush mykey a b c d a c
(integer) 6

#从头部(left)向尾部(right)变量链表，删除2个值等于a的元素，返回值为实际删除的数量。
redis 127.0.0.1:6379> lrem mykey 2 a
(integer) 2

#看出删除后链表中的全部元素。
redis 127.0.0.1:6379> lrange mykey 0 -1
1) "c"
2) "d"
3) "c"
4) "b"

#获取索引值为1(头部的第二个元素)的元素值。
redis 127.0.0.1:6379> lindex mykey 1
"d"

#将索引值为1(头部的第二个元素)的元素值设置为新值e。
redis 127.0.0.1:6379> lset mykey 1 e
OK

#查看是否设置成功。
redis 127.0.0.1:6379> lindex mykey 1
"e"

#索引值6超过了链表中元素的数量，该命令返回nil。
redis 127.0.0.1:6379> lindex mykey 6
(nil)

#设置的索引值6超过了链表中元素的数量，设置失败，该命令返回错误信息。
redis 127.0.0.1:6379> lset mykey 6 hh
(error) ERR index out of range

#仅保留索引值0到2之间的3个元素，注意第0个和第2个元素均被保留。
redis 127.0.0.1:6379> ltrim mykey 0 2
OK

#查看trim后的结果。
redis 127.0.0.1:6379> lrange mykey 0 -1
1) "c"
2) "e"
3) "c" 
```

linsert

```shell
#删除该键便于后面的测试。
redis 127.0.0.1:6379> del mykey
(integer) 1

#为后面的示例准备测试数据。
redis 127.0.0.1:6379> lpush mykey a b c d e
(integer) 5

#在a的前面插入新元素a1。
redis 127.0.0.1:6379> linsert mykey before a a1
(integer) 6

#查看是否插入成功，从结果看已经插入。注意lindex的index值是0-based。
redis 127.0.0.1:6379> lindex mykey 0
"e"

#在e的后面插入新元素e2，从返回结果看已经插入成功。
redis 127.0.0.1:6379> linsert mykey after e e2
(integer) 7

#再次查看是否插入成功。
redis 127.0.0.1:6379> lindex mykey 1
"e2"

#在不存在的元素之前或之后插入新元素，该命令操作失败，并返回-1。
redis 127.0.0.1:6379> linsert mykey after k a
(integer) -1

#为不存在的Key插入新元素，该命令操作失败，返回0。
redis 127.0.0.1:6379> linsert mykey1 after a a2
(integer) 0
```

rpush/rpushx/rpop/rpoplpush

```shell
#删除该键，以便于后面的测试。
redis 127.0.0.1:6379> del mykey
(integer) 1

#从链表的尾部插入参数中给出的values，插入顺序是从左到右依次插入。
redis 127.0.0.1:6379> rpush mykey a b c d
(integer) 4

#通过lrange的可以获悉rpush在插入多值时的插入顺序。
redis 127.0.0.1:6379> lrange mykey 0 -1
1) "a"
2) "b"
3) "c"
4) "d"

#该键已经存在并且包含4个元素，rpushx命令将执行成功，并将元素e插入到链表的尾部。
redis 127.0.0.1:6379> rpushx mykey e
(integer) 5

#通过lindex命令可以看出之前的rpushx命令确实执行成功，因为索引值为4的元素已经是新元素了。
redis 127.0.0.1:6379> lindex mykey 4
"e"

#由于mykey2键并不存在，因此该命令不会插入数据，其返回值为0。
redis 127.0.0.1:6379> rpushx mykey2 e
(integer) 0

#在执行rpoplpush命令前，先看一下mykey中链表的元素有哪些，注意他们的位置关系。
redis 127.0.0.1:6379> lrange mykey 0 -1
1) "a"
2) "b"
3) "c"
4) "d"
5) "e"

#将mykey的尾部元素e弹出，同时再插入到mykey2的头部(原子性的完成这两步操作)。
redis 127.0.0.1:6379> rpoplpush mykey mykey2
"e"

#通过lrange命令查看mykey在弹出尾部元素后的结果。
redis 127.0.0.1:6379> lrange mykey 0 -1
1) "a"
2) "b"
3) "c"
4) "d"

#通过lrange命令查看mykey2在插入元素后的结果。
redis 127.0.0.1:6379> lrange mykey2 0 -1
1) "e"

#将source和destination设为同一键，将mykey中的尾部元素移到其头部。
redis 127.0.0.1:6379> rpoplpush mykey mykey
"d"

#查看移动结果。
redis 127.0.0.1:6379> lrange mykey 0 -1
1) "d"
2) "a"
3) "b"
4) "c"
```
[注] redis链表操作的小技巧：<br>
针对链表结构的Value，Redis在其官方文档中给出了一些实用技巧，如RPOPLPUSH命令，下面给出具体的解释。
Redis链表经常会被用于消息队列的服务，以完成多程序之间的消息交换。假设一个应用程序正在执行LPUSH操
作向链表中添加新的元素，我们通常将这样的程序称之为"生产者(Producer)"，而另外一个应用程序正在执行
RPOP操作从链表中取出元素，我们称这样的程序为"消费者(Consumer)"。如果此时，消费者程序在取出消息元
素后立刻崩溃，由于该消息已经被取出且没有被正常处理，那么我们就可以认为该消息已经丢失，由此可能会
导致业务数据丢失，或业务状态的不一致等现象的发生。然而通过使用RPOPLPUSH命令，消费者程序在从主消
息队列中取出消息之后再将其插入到备份队列中，直到消费者程序完成正常的处理逻辑后再将该消息从备份队
列中删除。同时我们还可以提供一个守护进程，当发现备份队列中的消息过期时，可以重新将其再放回到主消
息队列中，以便其它的消费者程序继续处理。<br>















