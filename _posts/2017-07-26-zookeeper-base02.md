---
layout: post
title:  "zookeeper客户端命令"
desc: "zookeeper客户端命令"
keywords: "zookeeper客户端命令,linux,kinglyjn,张庆力"
date: 2017-07-26
categories: [Linux]
tags: [Linux]
icon: fa-bookmark-o
---



### 使用zkCli 连接zk服务器

```shell
zookeeper-3.4.6/bin/zkCli.sh
zookeeper-3.4.6/bin/zkCli.sh -server 172.16.127.129:2181
zookeeper-3.4.6/bin/zkCli.sh -timeout 5000 -r -server 172.16.127.129:2181 #单位ms，-r只读
```

<br>



### zkCli增删改查命令

```shell
#列出节点下所有的子节点
#ls path [watch]
> ls /
[storm, zookeeper]

> ls /storm
[backpressure, workerbeats, ..]


#列出节点的状态信息
#stat path [watch]
> stat /
cZxid = 0x0								#该节点被创建时的事务id
ctime = Thu Jan 01 08:00:00 CST 1970	#该节点被创建的时间
mZxid = 0x0								#该节点最后被修改时的事务id
mtime = Thu Jan 01 08:00:00 CST 1970	#该节点最后被修改的时间
pZxid = 0x41400000048	#该节点的子节点列表最后被修改时的事务id(添加或删除子节点，不含修改子节点内容)
cversion = 26							#子节点版本号
dataVersion = 0							#数据版本号
aclVersion = 0							#权限版本号
ephemeralOwner = 0x0					#创建该临时节点的事务id，如果是持久节点，这里的值固定为0
dataLength = 0							#该节点存放数据的长度
numChildren = 2							#该节点所拥有的子节点的个数


#列出节点下所有的子节点 和 该节点的状态信息
#ls2 path [watch]
> ls2 /
[storm, zookeeper]
cZxid = 0x0
ctime = Thu Jan 01 08:00:00 CST 1970
mZxid = 0x0
mtime = Thu Jan 01 08:00:00 CST 1970
pZxid = 0x41400000048
cversion = 26
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 0
numChildren = 2


#列出当前节点的数据内容
#get path [watch]
> get /znode01
123										#该节点存放的数据内容
cZxid = 0x4140000004d
ctime = Wed Jul 26 12:58:04 CST 2017
mZxid = 0x4140000004d
mtime = Wed Jul 26 12:58:04 CST 2017
pZxid = 0x4140000004d
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 3
numChildren = 0


#创建节点
#create [-s] [-e] path data acl
#-s：表示当前创建的节点是一个顺序节点
#-e：表示当前创建的节点是一个临时节点，临时节点下面不能再创建子节点，并且当前会话消失后该临时节点被删除
#-s和-e两个参数可以同时使用
> create -e /znode01/znode01_01 abc
Created /znode01/znode01_01

> ls /znode01
[znode01_01]

> get /znode01/znode01_01
abc										#
cZxid = 0x41400000050
...
ephemeralOwner = 0x15d7844fbdb000a		#退出创建该临时节点会话(ctrl+c)后，该临时节点将被删除
dataLength = 3							#zk临时节点的这种特性常常用于心跳检测
numChildren = 0

> create -s /znode01/znode01_02 456
Created /znode01/znode01_020000000001
> create -s /znode01/znode01_02 789
Created /znode01/znode01_020000000002
> ls /znode01
[znode01_020000000002, znode01_020000000001]
> get /znode01/znode01_020000000001
456
cZxid = 0x41400000052
...
numChildren = 0


#修改数据节点(只能修改已经存在的节点)
#set path data [version]
> set /znode01/znode01_020000000001 111
cZxid = 0x41400000052
ctime = Wed Jul 26 13:12:32 CST 2017
mZxid = 0x41400000056					#修改数据节点事务id发生改变
mtime = Wed Jul 26 13:18:02 CST 2017	#修改数据节点时间由初始的1970年改变为当前的修改时间
pZxid = 0x41400000052
cversion = 0
dataVersion = 1							#数据版本递增1
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 3
numChildren = 0

> set /znode01/znode01_020000000001 222 1  #修改指定版本号，注意版本号必须与当前版本号一致，否则报错，通常用于zk的乐观锁机制
cZxid = 0x41400000052
ctime = Wed Jul 26 13:12:32 CST 2017
mZxid = 0x41400000057					#修改数据节点事务id发生改变
mtime = Wed Jul 26 13:23:16 CST 2017    #修改数据节点时间发生改变
pZxid = 0x41400000052
cversion = 0
dataVersion = 2							##数据版本递增1
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 3
numChildren = 0


#删除数据节点(只能删除没有子节点的数据节点)
#delete path [version]
> delete /znode01/znode01_020000000001	  #删除该数据节点，没有任何返回就表示执行成功
> delete /znode01/znode01_020000000002 0  #删除指定数据版本号，注意版本号必须与当前版本号一致


#删除数据节点(删除当前节点及其子节点)
#rmr path
> rmr /znode01							  #删除该数据节点及其子节点，没有任何返回就表示执行成功
```

<br>



### zkCli配额命令

有时候我们希望给存在的数据节点增加一些限制条件，这是后就会用到限额命令。

当zk发现设置的数据节点超出了限额之后并不会报错，而是会在当前zk集群的节点上的 /bin/zookeeper.out 中记录警告信息，警告信息示例如下：

* 2017-07-26 13:44:14,393 [myid:1] - WARN  [CommitProcessor:1:DataTree@388] - Quota exceeded: /znode01 count=3 limit=2
* 2017-07-26 13:44:21,170 [myid:1] - WARN  [CommitProcessor:1:DataTree@388] - Quota exceeded: /znode01 count=4 limit=2

count：表示/znode01节点自身及其所有子节点的总数

limit：表示/znode01节点限制子自身及其节点的总数，由于我们设置的限制数为2，所以当count>limit之后将会有警告信息出现。<br>

```shell
#设置配额
#setquota -n|-b val path
#-n：表示限制当前节点子节点的个数
#-b：表示限制当前节点存放数据的长度
> setquota -n 2 /znode01		#表示限制 /znode01节点 子节点的个数为2
Comment: the parts are option -n val 2 path /znode01


#查看配额
#listquota path
> listquota /znode02
absolute path is /zookeeper/quota/znode02/zookeeper_limits	#当前节点配额信息存放路径
Output quota for /znode02 count=-1,bytes=2					#当前节点配额信息
Output stat for /znode02 count=1,bytes=4					#当前节点配额实际状态

> listquota /znode01
absolute path is /zookeeper/quota/znode01/zookeeper_limits
Output quota for /znode01 count=2,bytes=-1
Output stat for /znode01 count=7,bytes=25


#删除配额
#delquota [-n|-b] path
> delquota -b /znode02			#没有任何返回表示删除成功
```

<br>



### zkCli权限管理（ACL）命令

​传统的文件系统中，ACL分为两个维度，一个是属组，一个是权限，子目录/文件默认继承父目录的ACL。而在Zookeeper中，node的ACL是没有继承关系的，是独立控制的。Zookeeper的ACL，可以从三个维度来理解：一是scheme; 二是user; 三是permission，通常表示为 scheme : id : permissions, 下面从这三个方面分别来介绍：

* **scheme**：zk3 缺省支持下面几种scheme：
  * ip：它对应的id为客户机的IP地址，设置的时候可以设置一个ip段，比如ip:192.168.1.0/16, 表示匹配前16个bit的IP段
  * digest：它对应的id为username:BASE64(SHA1(password))，它需要先通过username:password形式的authentication
  * auth：它不需要id, 只要是通过authentication的user都有权限（zookeeper支持通过kerberos来进行authencation, 也支持username/password形式的authentication)
  * super：在这种scheme情况下，对应的id拥有超级权限，可以做任何事情(crwda)
  * world：它下面只有一个id, 叫anyone, world:anyone代表任何人，zookeeper中对所有人有权限的结点就是属于world:anyone的
  * sasl：zk3还提供了对sasl的支持，不过缺省是没有开启的，需要配置才能启用。sasl的对应的id，是一个通过sasl authentication用户的id，zk3中的sasl authentication是通过kerberos来实现的，也就是说用户只有通过了kerberos认证，才能访问它有权限的node.
* **id**：id与scheme是紧密相关的，具体的情况在上面介绍scheme的过程都已介绍，这里不再赘述。
* **permission：**zookeeper目前支持下面一些权限
  * CREATE(c): 创建权限，可以在在当前node下创建child node
  * READ(r): 读权限，可以获取当前node的数据，可以list当前node所有的child nodes
  * WRITE(w): 写权限，可以向当前node写数据
  * DELETE(d): 删除权限，可以删除当前的node
  * ADMIN(a): 管理权限，可以设置当前node的permission

```shell
#创建数据节点时设置acl
#create [-s] [-e] path data acl
> create /znode02 456 ip:172.16.127.131:crw
Created /znode02

> create /znode03 0 digest:tom:Aasasla+azxasU=:crwda #user:BASE64(SHA1(password))
Created /znode03


#获取当前节点的acl信息
> getAcl /znode02
'ip,'172.16.127.131
: crw


#设置节点的acl
#setAcl path acl
> setAcl /znode03 ip:172.16.127.131:crwd
cZxid = 0x4140000008f
...
aclVersion = 1	#aclVersion递增1
...
numChildren = 0


#添加认证以对节点数据进行访问
#addauth scheme auth
> addauth digest tom:123456


#当设置了znode权限，但是密码忘记了怎么办？还好Zookeeper提供了超级管理员机制
#修改zkServer.sh，加入super权限设置(只对当前zk集群节点)
> vim zkServer.sh
-----------------------------------------------
98 _ZOO_DAEMON_OUT="$ZOO_LOG_DIR/zookeeper.out"
99 SUPER_ACL="-Dzookeeper.DigestAuthenticationProvider.superDigest=super:gG7s8t3oDEtIqF6DM9LlI/R+9Ss="
100 
101 
102 case $1 in
...
111     nohup "$JAVA" "-Dzookeeper.log.dir=${ZOO_LOG_DIR}" "-Dzookeeper.root.logger=${ZOO_LOG4J_PROP}" "$SUPER_ACL"\
112     -cp "$CLASSPATH" $JVMFLAGS $ZOOMAIN "$ZOOCFG" > "$_ZOO_DAEMON_OUT" 2>&1 < /dev/null &
113     if [ $? -eq 0 ]
...

#重新启动Zookeeper，这时候使用 addauth digest super:super 进行认证
> ./zkServer.sh restart

> ./zkCli.sh

> get /znode02
Authentication is not valid : /znode02

> addauth digest super:super
> get /znode02
456
cZxid = 0x4140000008c
...
numChildren = 0

> delete /znode02
```

<br>



### zkCli其他命令

```shell
#在当前zkCli终端连接其他zk服务器
#connect host:port
> connect supervisor01z:2181

#退出当前客户端
#quit 或 ctl+c
> quit

#设置是否打印监听，默认为on
#printwatches on|off
> printwatches on

#查看所有指令的执行历史
#history
> history
0 - l
1 - printwatches on
2 - history
3 - redo
4 - ls /
5 - history

#重复执行history中某个命令
#redo
> redo 4	#相当于执行 ls /

#保证获取的数据是最新的
#Zookeeper保证了最终一致性,在十几秒可以Sync到各个节点.
#Zookeeper保证了可用性,数据总是可用的,没有锁。并且有一大半的节点所拥有的数据是最新的,实时的。
#如果想保证取得是数据一定是最新的,需要手工调用Sync()
#sync path
> sync /znode01
Sync returned 0
```



