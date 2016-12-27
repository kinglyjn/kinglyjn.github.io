---
layout: post
title:  "redis基础简介"
desc: "redis基础简介"
keywords: "redis基础简介,redis,kinglyjn"
date: 2012-10-18
categories: [DB]
tags: [redis]
icon: fa-database
---

### 常见nosql产品之间的比较

1. BerkeleyDB：支持事务及嵌套事务，海量数据存储，在用于存储实时数据方面具有极高的可用价值，不完全免费。
2. MongoDB：独立的运行并提供相关的数据服务,主要适用于高并发的论坛或博客网站,多读少写、数据量大、逻辑关系简单，以及文档数据作为主要数据源等, 和BerkeleyDB一样，该产品的License同为GPL。
3. Redis：可以作为服务程序独立运行于自己的服务器主机,Redis除了Key/Value之外还支持List、Hash、Set和Ordered Set等数据结构, Redis的License是Apache License，就目前而言，它是完全免费。
4. memcached：与redis的比较，memcached只是提供了数据缓存服务，一旦服务器宕机，之前在内存中缓存的数据也将全部消失，因此可以看出memcached没有提供任何形式的数据持久化功能，而Redis则提供了这样的功能,再有就是Redis提供了更为丰富的数据存储结构，如Hash和Set。至于它们的相同点，主要有两个，一是完全免费，再有就是它们的提供的命令形式极为接近。

<b>redis的优势：</b>

1. 和其他NoSQL产品相比，Redis的易用性极高，因此对于那些有类似产品使用经验的开发者来说，一两天，甚至是几个小时之后就可以利用Redis来搭建自己的平台了。
2. 在解决了很多通用性问题的同时，也为一些个性化问题提供了相关的解决方案，如索引引擎、统计排名、消息队列服务等。
3. 在数据存储方面，Redis遵循了现有NoSQL数据库的主流思想，即Key作为数据检索的唯一标识，我们可以将其简单的理解为关系型数据库中索引的键，而Value则作为数据存储的主要对象，其中每一个Value都有一个Key与之关联，这就好比索引中物理数据在数据表中存储的位置。在Redis中，Value将被视为二进制字节流用于存储任何格式的数据，如Json、XML和序列化对象的字节流等，因此我们也可以将其想象为RDB中的BLOB类型字段。由此可见，在进行数据查询时，我们只能基于Key作为我们查询的条件，当然我们也可以应用Redis中提供的一些技巧将Value作为其他数据的Key。

<b>redis的持久化：</b> <br>

缺省情况下，Redis会参照当前数据库中数据被修改的数量，在达到一定的阈值后会将数据库的快照存储到磁盘上，这一点我们可以通过配置文件来设定该阈值。通常情况下，我们也可以将Redis设定为定时保存。如当有1000个以上的键数据被修改时，Redis将每隔60秒进行一次数据持久化操作。缺省设置为，如果有9个或9个以下数据修改是，Redis将每15分钟持久化一次。<br>

从上面提到的方案中可以看出，如果采用该方式，Redis的运行时效率将会是非常高效的，既每当有新的数据修改发生时，仅仅是内存中的缓存数据发生改变，而这样的改变并不会被立即持久化到磁盘上，从而在绝大多数的修改操作中避免了磁盘IO的发生。然而事情往往是存在其两面性的，在该方法中我们确实得到了效率上的提升，但是却失去了数据可靠性。如果在内存快照被持久化到磁盘之前，Redis所在的服务器出现宕机，那么这些未写入到磁盘的已修改数据都将丢失。为了保证数据的高可靠性，redis还提供了另外一种数据持久化机制，即append模式。如果Redis服务器被配置为该方式，那么每当有数据修改发生时，都会被立即持久化到磁盘。<br>
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

# 在Ubuntu上安装Redis的桌面管理器
# http://redisdesktop.com/download

#选取数据库
#Redis中的数据库是通过数字来进行命名的，缺省情况下打开的数据库为0，默认有16个库（0-15）
#切换redis数据库
redis 127.0.0.1:6379 > select_ 1
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















