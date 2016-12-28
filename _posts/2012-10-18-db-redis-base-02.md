---
layout: post
title:  "redis基础简介（二）- hash（哈希）、set（集合）、sorted set（有序集合）以及键操作"
desc: "redis基础简介（二）- hash（哈希）、set（集合）、sorted set（有序集合）以及键操作"
keywords: "redis基础简介,哈希,hash,集合,set,有序集合,sorted set,redis,kinglyjn"
date: 2012-10-18
categories: [DB]
tags: [redis]
icon: fa-database
---

### 哈希类型(hash)相关操作

我们可以将Redis中的Hashes类型看成具有String Key和String Value的map容器。所以该类型非常适合于存储值对象的信息。
如Username、Password和Age等。如果Hash中包含很少的字段，那么该类型的数据也将仅占用很少的磁盘空间。每一个Hash可
以存储4294967295个键值对。<br><br>

hset/hget/hdel/hexists/hlen/hsetnx

```shell
#给键值为myhash的键设置字段为field1，值为stephen。
redis 127.0.0.1:6379> hset myhash field1 "stephen"
(integer) 1

#获取键值为myhash，字段为field1的值。
redis 127.0.0.1:6379> hget myhash field1
"stephen"

#myhash键中不存在field2字段，因此返回nil。
redis 127.0.0.1:6379> hget myhash field2
(nil)

#给myhash关联的Hashes值添加一个新的字段field2，其值为liu。
redis 127.0.0.1:6379> hset myhash field2 "liu"
(integer) 1

#获取myhash键的字段数量。
redis 127.0.0.1:6379> hlen myhash
(integer) 2

#判断myhash键中是否存在字段名为field1的字段，由于存在，返回值为1。
redis 127.0.0.1:6379> hexists myhash field1
(integer) 1

#删除myhash键中字段名为field1的字段，删除成功返回1。
redis 127.0.0.1:6379> hdel myhash field1
(integer) 1

#再次删除myhash键中字段名为field1的字段，由于上一条命令已经将其删除，因为没有删除，返回0。
redis 127.0.0.1:6379> hdel myhash field1
(integer) 0

#判断myhash键中是否存在field1字段，由于上一条命令已经将其删除，因为返回0。
redis 127.0.0.1:6379> hexists myhash field1
(integer) 0

#通过hsetnx命令给myhash添加新字段field1，其值为stephen，因为该字段已经被删除，所以该命令添加成功并返回1。
redis 127.0.0.1:6379> hsetnx myhash field1 stephen
(integer) 1

#由于myhash的field1字段已经通过上一条命令添加成功，因为本条命令不做任何操作后返回0。
redis 127.0.0.1:6379> hsetnx myhash field1 stephen
(integer) 0
```

hgetall/hkeys/hvals/hmget/hmset

```shell
#删除该键，便于后面示例测试。
redis 127.0.0.1:6379> del myhash
(integer) 1

#为该键myhash，一次性设置多个字段，分别是field1 = "hello", field2 = "world"。
redis 127.0.0.1:6379> hmset myhash field1 "hello" field2 "world"
OK

#获取myhash键的多个字段，其中field3并不存在，因为在返回结果中与该字段对应的值为nil。
redis 127.0.0.1:6379> hmget myhash field1 field2 field3
1) "hello"
2) "world"
3) (nil)

#返回myhash键的所有字段及其值，从结果中可以看出，他们是逐对列出的。
redis 127.0.0.1:6379> hgetall myhash
1) "field1"
2) "hello"
3) "field2"
4) "world"

#仅获取myhash键中所有字段的名字。
redis 127.0.0.1:6379> hkeys myhash
1) "field1"
2) "field2"

#仅获取myhash键中所有字段的值。
redis 127.0.0.1:6379> hvals myhash
1) "hello"
2) "world"
```

hincrby

```shell
#删除该键，便于后面示例的测试。
redis 127.0.0.1:6379> del myhash
(integer) 1

#准备测试数据，该myhash的field字段设定值1。
redis 127.0.0.1:6379> hset myhash field 5
(integer) 1

#给myhash的field字段的值加1，返回加后的结果。
redis 127.0.0.1:6379> hincrby myhash field 1
(integer) 6

#给myhash的field字段的值加-1，返回加后的结果。
redis 127.0.0.1:6379> hincrby myhash field -1
(integer) 5

#给myhash的field字段的值加-10，返回加后的结果。
redis 127.0.0.1:6379> hincrby myhash field -10
(integer) -5
```
<br>

### 集合类型(set)相关操作

在Redis中，我们可以将Set类型看作为没有排序的字符集合，和List类型一样，我们也可以在该类型的数据值上执行添加、删除或判断某一元素是否存在等操作。需要说明的是，这些操作的时间复杂度为O(1)，即常量时间内完成次操作。Set可包含的最大元素数量是2^32-1 (4294967295)。<br>

和List类型不同的是，Set集合中不允许出现重复的元素，这一点和C++标准库中的set容器是完全相同的。换句话说，如果多次添加相同元素，Set中将仅保留该元素的一份拷贝。和List类型相比，Set类型在功能上还存在着一个非常重要的特性，即在服务器端完成多个Sets之间的聚合计算操作，
如unions、intersections和differences。由于这些操作均在服务端完成，因此效率极高，而且也节省了大量的网络IO开销。<br>

应用范围：

1. 可以使用Redis的Set数据类型跟踪一些唯一性数据，比如访问某一博客的唯一IP地址信息。对于此场景，我们仅需在每
次访问该博客时将访问者的IP存入Redis中，Set数据类型会自动保证IP地址的唯一性。
2. 充分利用Set类型的服务端聚合操作方便、高效的特性，可以用于维护数据对象之间的关联关系。比如所有购买某一电子
设备的客户ID被存储在一个指定的Set中，而购买另外一种电子产品的客户ID被存储在另外一个Set中，如果此时我们想
获取有哪些客户同时购买了这两种商品时，Set的intersections命令就可以充分发挥它的方便和效率的优势了。

sadd/smembers/scard/sismember

```shell
#插入测试数据，由于该键myset之前并不存在，因此参数中的三个成员都被正常插入。
redis 127.0.0.1:6379> sadd myset a b c
(integer) 3

#由于参数中的a在myset中已经存在，因此本次操作仅仅插入了d和e两个新成员。
redis 127.0.0.1:6379> sadd myset a d e
(integer) 2

#判断a是否已经存在，返回值为1表示存在。
redis 127.0.0.1:6379> sismember myset a
(integer) 1

#判断f是否已经存在，返回值为0表示不存在。
redis 127.0.0.1:6379> sismember myset f
(integer) 0

#通过smembers命令查看插入的结果，从结果可以，输出的顺序和插入顺序无关。
redis 127.0.0.1:6379> smembers myset
1) "c"
2) "d"
3) "a"
4) "b"
5) "e"

#获取Set集合中元素的数量。
redis 127.0.0.1:6379> scard myset
(integer) 5
```

spop/srem/srandmember/smove

```shell
 #删除该键，便于后面的测试。
redis 127.0.0.1:6379> del myset
(integer) 1

#为后面的示例准备测试数据。
redis 127.0.0.1:6379> sadd myset a b c d
(integer) 4

#查看Set中成员的位置。
redis 127.0.0.1:6379> smembers myset
1) "c"
2) "d"
3) "a"
4) "b"

#从结果可以看出，该命令确实是随机的返回了某一成员。
redis 127.0.0.1:6379> srandmember myset
"c"

#Set中尾部的成员b被移出并返回，事实上b并不是之前插入的第一个或最后一个成员。
redis 127.0.0.1:6379> spop myset
"b"

#查看移出后Set的成员信息。
redis 127.0.0.1:6379> smembers myset
1) "c"
2) "d"
3) "a"

#从Set中移出a、d和f三个成员，其中f并不存在，因此只有a和d两个成员被移出，返回为2。
redis 127.0.0.1:6379> srem myset a d f
(integer) 2

#查看移出后的输出结果。
redis 127.0.0.1:6379> smembers myset
1) "c"
```

sdiff/sdiffstore/sinter/sinterstore/sunion/sunionstore

```shell
#为后面的smove命令准备数据。
redis 127.0.0.1:6379> sadd myset a b
(integer) 2
redis 127.0.0.1:6379> sadd myset2 c d
(integer) 2

#将a从myset移到myset2，从结果可以看出移动成功。
redis 127.0.0.1:6379> smove myset myset2 a
(integer) 1

#再次将a从myset移到myset2，由于此时a已经不是myset的成员了，因此移动失败并返回0。
redis 127.0.0.1:6379> smove myset myset2 a
(integer) 0

#分别查看myset和myset2的成员，确认移动是否真的成功。
redis 127.0.0.1:6379> smembers myset
1) "b"
redis 127.0.0.1:6379> smembers myset2
1) "c"
2) "d"
3) "a"


#为后面的命令准备测试数据。
redis 127.0.0.1:6379> sadd myset a b c d
(integer) 4
redis 127.0.0.1:6379> sadd myset2 c
(integer) 1
redis 127.0.0.1:6379> sadd myset3 a c e
(integer) 3

#myset和myset2相比，a、b和d三个成员是两者之间的差异成员。再用这个结果继续和myset3进行差异比较，b和d是myset3不存在的成员。
redis 127.0.0.1:6379> sdiff myset myset2 myset3
1) "d"
2) "b"

#将3个集合的差异成员存储在diffkey关联的Set中，并返回插入的成员数量。
redis 127.0.0.1:6379> sdiffstore diffkey myset myset2 myset3
(integer) 2

#查看一下sdiffstore的操作结果。
redis 127.0.0.1:6379> smembers diffkey
1) "d"
2) "b"

#从之前准备的数据就可以看出，这三个Set的成员交集只有c。
redis 127.0.0.1:6379> sinter myset myset2 myset3
1) "c"

#将3个集合中的交集成员存储到与interkey关联的Set中，并返回交集成员的数量。
redis 127.0.0.1:6379> sinterstore interkey myset myset2 myset3
(integer) 1

#查看一下sinterstore的操作结果。
redis 127.0.0.1:6379> smembers interkey
1) "c"

#获取3个集合中的成员的并集。    
redis 127.0.0.1:6379> sunion myset myset2 myset3
1) "b"
2) "c"
3) "d"
4) "e"
5) "a"

#将3个集合中成员的并集存储到unionkey关联的set中，并返回并集成员的数量。
redis 127.0.0.1:6379> sunionstore unionkey myset myset2 myset3
(integer) 5

#查看一下suiionstore的操作结果。
redis 127.0.0.1:6379> smembers unionkey
1) "b"
2) "c"
3) "d"
4) "e"
5) "a" 
```
<br>

### 有序集合类型(sorted set)相关操作

sorted-sets和Sets类型极为相似，它们都是字符串的集合，都不允许重复的成员出现在一个Set中。它们之间的主要差别是sorted-sets
中的每一个成员都会有一个分数(score)与之关联，Redis正是通过分数来为集合中的成员进行从小到大的排序。然而需要额外指出的是，
尽管sorted-sets中的成员必须是唯一的，但是分数(score)却是可以重复的。<br>

在sorted-set中添加、删除或更新一个成员都是非常快速的操作，其时间复杂度为集合中成员数量的对数。由于sorted-sets中的成员在
集合中的位置是有序的，因此，即便是访问位于集合中部的成员也仍然是非常高效的。事实上，Redis所具有的这一特征在很多其它类型
的数据库中是很难实现的，换句话说，在该点上要想达到和Redis同样的高效，在其它数据库中进行建模是非常困难的。<br>

应用范围:

1. 可以用于一个大型在线游戏的积分排行榜。每当玩家的分数发生变化时，可以执行ZADD命令更新玩家的分数，此后再通过ZRANGE命令获取积分TOP TEN的用户信息。当然我们也可以利用ZRANK命令通过username来获取玩家的排行信息。最后我们将组合使用ZRANGE和ZRANK命令快速的获取和某个玩家积分相近的其他用户的信息。
2. Sorted-Sets类型还可用于构建索引数据。

zadd/zcard/zcount/zrem/zincrby/zscore/zrange/zrank:

```shell
#添加一个分数为1的成员。
redis 127.0.0.1:6379> zadd myzset 1 "one"
(integer) 1

#添加两个分数分别是2和3的两个成员。
redis 127.0.0.1:6379> zadd myzset 2 "two" 3 "three"
(integer) 2

#0表示第一个成员，-1表示最后一个成员。WITHSCORES选项表示返回的结果中包含每个成员及其分数，否则只返回成员。
redis 127.0.0.1:6379> zrange myzset 0 -1 WITHSCORES
1) "one"
2) "1"
3) "two"
4) "2"
5) "three"
6) "3"

#获取成员one在Sorted-Set中的位置索引值。0表示第一个位置。
redis 127.0.0.1:6379> zrank myzset one
(integer) 0

#成员four并不存在，因此返回nil。
redis 127.0.0.1:6379> zrank myzset four
(nil)

#获取myzset键中成员的数量。    
redis 127.0.0.1:6379> zcard myzset
(integer) 3

#返回与myzset关联的Sorted-Set中，分数满足表达式1 <= score <= 2的成员的数量。
redis 127.0.0.1:6379> zcount myzset 1 2
(integer) 2

#删除成员one和two，返回实际删除成员的数量。
redis 127.0.0.1:6379> zrem myzset one two
(integer) 2

#查看是否删除成功。
redis 127.0.0.1:6379> zcard myzset
(integer) 1

#获取成员three的分数。返回值是字符串形式。
redis 127.0.0.1:6379> zscore myzset three
"3"

#由于成员two已经被删除，所以该命令返回nil。
redis 127.0.0.1:6379> zscore myzset two
(nil)

#将成员one的分数增加2，并返回该成员更新后的分数(没有该成员则创建)。
redis 127.0.0.1:6379> zincrby myzset 2 one
"3"

#将成员one的分数增加-1，并返回该成员更新后的分数。
redis 127.0.0.1:6379> zincrby myzset -1 one
"2"

#查看在更新了成员的分数后是否正确。
redis 127.0.0.1:6379> zrange myzset 0 -1 WITHSCORES
1) "one"
2) "2"
3) "two"
4) "2"
5) "three"
6) "3"
```

zrangebyscore/zremrangebyrank/zremrangebyscore

```shell
redis 127.0.0.1:6379> del myzset
(integer) 1
redis 127.0.0.1:6379> zadd myzset 1 one 2 two 3 three 4 four
(integer) 4

#获取分数满足表达式1 <= score <= 2的成员。
redis 127.0.0.1:6379> zrangebyscore myzset 1 2
1) "one"
2) "two"

#获取分数满足表达式1 < score <= 2的成员。
redis 127.0.0.1:6379> zrangebyscore myzset (1 2
1) "two"

#-inf表示第一个成员，+inf表示最后一个成员，limit后面的参数用于限制返回成员的自己，
#2表示从位置索引(0-based)等于2的成员开始，取后面3个成员。
redis 127.0.0.1:6379> zrangebyscore myzset -inf +inf limit 2 3
1) "three"
2) "four"

#删除分数满足表达式1 <= score <= 2的成员，并返回实际删除的数量。
redis 127.0.0.1:6379> zremrangebyscore myzset 1 2
(integer) 2

#看出一下上面的删除是否成功。
redis 127.0.0.1:6379> zrange myzset 0 -1
1) "three"
2) "four"

#删除位置索引满足表达式0 <= rank <= 1的成员。
redis 127.0.0.1:6379> zremrangebyrank myzset 0 1
(integer) 2

#查看上一条命令是否删除成功。
redis 127.0.0.1:6379> zcard myzset
(integer) 0
```

zrevrange/zrevrangebyscore/zrevrank:

```shell
#为后面的示例准备测试数据。
redis 127.0.0.1:6379> del myzset
(integer) 0
redis 127.0.0.1:6379> zadd myzset 1 one 2 two 3 three 4 four
(integer) 4

#以位置索引从高到低的方式获取并返回此区间内的成员。
redis 127.0.0.1:6379> zrevrange myzset 0 -1 WITHSCORES
1) "four"
2) "4"
3) "three"
4) "3"
5) "two"
6) "2"
7) "one"
8) "1"

#由于是从高到低的排序，所以位置等于0的是four，1是three，并以此类推。
redis 127.0.0.1:6379> zrevrange myzset 1 3
1) "three"
2) "two"
3) "one"

#由于是从高到低的排序，所以one的位置是3。
redis 127.0.0.1:6379> zrevrank myzset one
(integer) 3

#由于是从高到低的排序，所以four的位置是0。
redis 127.0.0.1:6379> zrevrank myzset four
(integer) 0

#获取分数满足表达式3 >= score >= 0的成员，并以相反的顺序输出，即从高到底的顺序。
redis 127.0.0.1:6379> zrevrangebyscore myzset 3 0
1) "three"
2) "two"
3) "one"

#该命令支持limit选项，其含义等同于zrangebyscore中的该选项，只是在计算位置时按照相反的顺序计算和获取。
redis 127.0.0.1:6379> zrevrangebyscore myzset 4 0 limit 1 2
1) "three"
2) "two"
```
<br>

### 键有关操作

keys/rename/del/exists/move/renamenx

```shell
#清空当前选择的数据库，以便于对后面示例的理解。
redis 127.0.0.1:6379> flushdb
OK

#添加String类型的模拟数据。
redis 127.0.0.1:6379> set mykey 2
OK
redis 127.0.0.1:6379> set mykey2 "hello"
OK

#添加Set类型的模拟数据。
redis 127.0.0.1:6379> sadd mysetkey 1 2 3
(integer) 3

#添加Hash类型的模拟数据。
redis 127.0.0.1:6379> hset mmtest username "stephen"
(integer) 1

#根据参数中的模式，获取当前数据库中符合该模式的所有key，从输出可以看出，该命令在执行时并不区分与Key关联的Value类型。
redis 127.0.0.1:6379> keys my*
1) "mysetkey"
2) "mykey"
3) "mykey2"

#删除了两个Keys。
redis 127.0.0.1:6379> del mykey mykey2
(integer) 2

#查看一下刚刚删除的Key是否还存在，从返回结果看，mykey确实已经删除了。
redis 127.0.0.1:6379> exists mykey
(integer) 0

#查看一下没有删除的Key，以和上面的命令结果进行比较。
redis 127.0.0.1:6379> exists mysetkey
(integer) 1

#将当前数据库中的mysetkey键移入到ID为1的数据库中，从结果可以看出已经移动成功。
redis 127.0.0.1:6379> move mysetkey 1
(integer) 1

#打开ID为1的数据库。
redis 127.0.0.1:6379> select_ 1
OK

#查看一下刚刚移动过来的Key是否存在，从返回结果看已经存在了。
redis 127.0.0.1:6379[1]> exists mysetkey
(integer) 1

#在重新打开ID为0的缺省数据库。
redis 127.0.0.1:6379[1]> select_ 0
OK

#查看一下刚刚移走的Key是否已经不存在，从返回结果看已经移走。
redis 127.0.0.1:6379> exists mysetkey
(integer) 0

#准备新的测试数据。    
redis 127.0.0.1:6379> set mykey "hello"
OK

#将mykey改名为mykey1
redis 127.0.0.1:6379> rename mykey mykey1
OK

#由于mykey已经被重新命名，再次获取将返回nil。
redis 127.0.0.1:6379> get mykey
(nil)

#通过新的键名获取。
redis 127.0.0.1:6379> get mykey1
"hello"

#由于mykey已经不存在了，所以返回错误信息。
redis 127.0.0.1:6379> rename mykey mykey1
(error) ERR no such key

#为renamenx准备测试key
redis 127.0.0.1:6379> set oldkey "hello"
OK
redis 127.0.0.1:6379> set newkey "world"
OK

#由于newkey已经存在，因此该命令未能成功执行。
redis 127.0.0.1:6379> renamenx oldkey newkey
(integer) 0

#查看newkey的值，发现它也没有被renamenx覆盖。
redis 127.0.0.1:6379> get newkey
"world"
```

persist/expire/expireat/ttl

```shell
#为后面的示例准备的测试数据。
redis 127.0.0.1:6379> set mykey "hello"
OK

#将该键的超时设置为100秒。
redis 127.0.0.1:6379> expire mykey 100
(integer) 1

#以时间戳的形式设置过期时间
redis 127.0.0.1:6379> expireat mykey 1293840000

#通过ttl命令查看一下还剩下多少秒。
redis 127.0.0.1:6379> ttl mykey
(integer) 97

#立刻执行persist命令，该存在超时的键变成持久化的键，即将该Key的超时去掉。
redis 127.0.0.1:6379> persist mykey
(integer) 1

#ttl的返回值告诉我们，该键已经没有超时了。
redis 127.0.0.1:6379> ttl mykey
(integer) -1

#为后面的expire命令准备数据。
redis 127.0.0.1:6379> del mykey
(integer) 1
redis 127.0.0.1:6379> set mykey "hello"
OK

#设置该键的超时被100秒。
redis 127.0.0.1:6379> expire mykey 100
(integer) 1

#用ttl命令看一下当前还剩下多少秒，从结果中可以看出还剩下96秒。
redis 127.0.0.1:6379> ttl mykey
(integer) 96

#重新更新该键的超时时间为20秒，从返回值可以看出该命令执行成功。
redis 127.0.0.1:6379> expire mykey 20
(integer) 1

#再用ttl确认一下，从结果中可以看出果然被更新了。
redis 127.0.0.1:6379> ttl mykey
(integer) 17

#立刻更新该键的值，以使其超时无效。
redis 127.0.0.1:6379> set mykey "world"
OK

#从ttl的结果可以看出，在上一条修改该键的命令执行后，该键的超时也无效了。
redis 127.0.0.1:6379> ttl mykey
(integer) -1
```

type/randomkey/sort

```shell
#由于mm键在数据库中不存在，因此该命令返回none。
redis 127.0.0.1:6379> type mm
none

#mykey的值是字符串类型，因此返回string。
redis 127.0.0.1:6379> type mykey
string

#准备一个值是set类型的键。
redis 127.0.0.1:6379> sadd mysetkey 1 2
(integer) 2

#mysetkey的键是set，因此返回字符串set。
redis 127.0.0.1:6379> type mysetkey
set

#返回数据库中的任意键。
redis 127.0.0.1:6379> randomkey
"oldkey"

#清空当前打开的数据库。
redis 127.0.0.1:6379> flushdb
OK

#由于没有数据了，因此返回nil。
redis 127.0.0.1:6379> randomkey
(nil)
```
<br>
















