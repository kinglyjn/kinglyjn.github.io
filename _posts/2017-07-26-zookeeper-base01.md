---
layout: post
title:  "zookeeper安装及概述（以zookeeper-3.4.6为例）"
desc: "zookeeper安装及概述（以zookeeper-3.4.6为例）"
keywords: "zookeeper安装及概述,linux,kinglyjn,张庆力"
date: 2017-07-26
categories: [Linux]
tags: [Linux]
icon: fa-bookmark-o
---



### zk概述

>  zookeeper是由雅虎创造的、`源码开放`的`分布式协调服务`，它将那些复杂的、容易出错的分布式一致性服务封装起来，构成一个`高效`(三台zk集群可达到12-13w的qbs)`可靠`的原语集，并提供一系列简单易用的接口给用户使用

<br>



分布式数据的一致性问题内容：

* 顺序一致性：从一个客户端发起一个事务请求，最终会严格按照发起的顺序被应用到zk集群中去
* 原子性：所有事务请求的处理结果在整个集群所有机器上的应用情况是一致的
* 单一视图：无论客户端连接到哪个客户端zk服务器，他看到的服务端数据都是一致的
* 可靠性：一旦服务端成功的应用了一个事务并完成了对客户端的响应，那么这个事务所引起的服务端状态变更会一直保留下来，除非有另外一个事务又对它进行了修改
* 实时性：zk保证在一定时间内，客户端最终一定能从服务端读取到数据的最新的状态

<br>



zk的数据模型:

树形结构和访问控制，Zookeeper 这种数据结构有如下这些特点：

1. 每个子目录项如 NameService 都被称作为 znode，这个 znode 是被它所在的路径唯一标识，如 Server1 这个 znode 的标识为 /NameService/Server1
2. znode 可以有子节点目录，并且每个 znode 可以存储数据，注意 EPHEMERAL 类型的目录节点不能有子节点目录
3. znode 是有版本的，每个 znode 中存储的数据可以有多个版本，也就是一个访问路径中可以存储多份数据
4. znode 可以是临时节点，一旦创建这个 znode 的客户端与服务器失去联系，这个 znode 也将自动删除，Zookeeper 的客户端和服务器通信采用长连接方式，每个客户端和服务器通过心跳来保持连接，这个连接状态称为 session，如果 znode 是临时节点，这个 session 失效，znode 也就删除了
5. znode 的目录名可以自动编号，如 App1 已经存在，再创建的话，将会自动命名为 App2
6. znode 可以被监控，包括这个目录节点中存储的数据的修改，子节点目录的变化等，一旦变化可以通知设置监控的客户端，这个是 Zookeeper 的核心特性，Zookeeper 的很多功能都是基于这个特性实现的，后面在典型的应用场景中会有实例介绍

<br>



### zk的应用场景

**数据的发布订阅：**

顾名思义，数据的发布订阅是指一方把数据发布出来，另一方通过某种手段可以得到这些数据。通常数据的订阅有两种方式，及`推模式`和`拉模式`，推模式一般是服务器主动向客户端推送消息，拉模式是客户端主动去服务器获取数据（通常采用定时轮询的方式）。zk采用两种方式结合，发布者将消息发布到zk集群节点上，订阅者通过一定的方法告诉服务器我对哪个节点的数据感兴趣（通过注册watcher），那服务器在这些节点的数据发生变化时就通知客户端，客户端得到通知以后就可以去服务端获取数据信息了。

<br>

**负载均衡:**

一般是使用zk的临时节点来实现。需要应用程序编码实现均衡策略。

<br>

**命名服务：**

顾名思义，就是提供名称的服务。例如数据库表格id，一般用的比较多的是两种id，一种是自动增长的id，一种是uuid，两种id都各自有优缺点，自动增长的id一般常用在单库单表中而不能用在分布式中，uuid可以使用在分布式环境中但没有规律难以理解，我们可以借助于zk来生成一个顺序增长的，可以在集群环境中使用的、命名易于理解的id。
<br>

**配置信息管理：**

配置的管理在分布式应用环境中很常见，例如同一个应用系统需要多台 PC Server 运行，但是它们运行的应用系统的某些配置项是相同的，如果要修改这些相同的配置项，那么就必须同时修改每台运行这个应用系统的 PC Server，这样非常麻烦而且容易出错。像这样的配置信息完全可以交给 Zookeeper 来管理，将配置信息保存在 Zookeeper 的某个目录节点中，然后将所有需要修改的应用机器监控配置信息的状态，一旦配置信息发生变化，每台应用机器就会收到 Zookeeper 的通知，然后从 Zookeeper 获取新的配置信息应用到系统中。

<br>

**心跳检测：**

在分布式系统中，我们常常需要知道某些机器是否可用，传统的开发中年可以通过ping某个主机来实现，而zk中我们让所有的机器都注册一个临时节点(注意临时节点下不能再创建子节点，Ephemerals cannot have children)，判断一个机器是否可用，我们只需要判断这个节点在zk中是否存在就可以了，不需要直接去连接需要检查的机器，降低系统的复杂度。

<br>

**集群管理和leader选举：**

Zookeeper 能够很容易的实现集群管理的功能，如有多台 Server 组成一个服务集群，那么必须要一个“总管”知道当前集群中每台机器的服务状态，一旦有机器不能提供服务，集群中其它集群必须知道，从而做出调整重新分配服务策略。同样当增加集群的服务能力时，就会增加一台或多台 Server，同样也必须让“总管”知道。

Zookeeper 不仅能够帮你维护当前的集群中机器的服务状态，而且能够帮你选出一个“总管”，让这个总管来管理集群，这就是 Zookeeper 的另一个功能 Leader Election。

它们的实现方式都是在 Zookeeper 上创建一个 EPHEMERAL 类型的目录节点，然后每个 Server 在它们创建目录节点的父目录节点上调用 getChildren(String path, boolean watch) 方法并设置 watch 为 true，由于是 ephemeral（临时） 目录节点，当创建它的 Server 死去，这个目录节点也随之被删除，所以 Children 将会变化，这时 getChildren 上的 Watch 将会被调用，所以其它 Server 就知道已经有某台 Server 死去了。新增 Server 也是同样的原理。

Zookeeper 如何实现 Leader Election，也就是选出一个 Master Server。和前面的一样每台 Server 创建一个 EPHEMERAL 目录节点，不同的是它还是一个 SEQUENTIAL 目录节点，所以它是个 EPHEMERAL_SEQUENTIAL 目录节点。之所以它是 EPHEMERAL_SEQUENTIAL 目录节点，是因为我们可以给每台 Server 编号，我们可以选择当前是最小编号的 Server 为 Master，假如这个最小编号的 Server 死去，由于是 EPHEMERAL 节点，死去的 Server 对应的节点也被删除，所以当前的节点列表中又出现一个最小编号的节点，我们就选择这个节点为当前 Master。这样就实现了动态选择 Master，避免了传统意义上单 Master 容易出现单点故障的问题。

<br>

**共享锁（Locks）：**

共享锁在同一个进程中很容易实现，但是在跨进程或者在不同 Server 之间就不好实现了。Zookeeper 却很容易实现这个功能，实现方式也是需要获得锁的 Server 创建一个 EPHEMERAL_SEQUENTIAL 目录节点，然后调用 getChildren方法获取当前的目录节点列表中最小的目录节点是不是就是自己创建的目录节点，如果正是自己创建的，那么它就获得了这个锁，如果不是那么它就调用 exists(String path, boolean watch) 方法并监控 Zookeeper 上目录节点列表的变化，一直到自己创建的节点是列表中最小编号的目录节点，从而获得锁，释放锁很简单，只要删除前面它自己所创建的目录节点就行了。

<br>

**队列管理：**

Zookeeper 可以处理两种类型的队列：

1. 当一个队列的成员都聚齐时，这个队列才可用，否则一直等待所有成员到达，这种是同步队列。
2. 队列按照 FIFO 方式进行入队和出队操作，例如实现生产者和消费者模型。

同步队列用 Zookeeper 实现的实现思路如下：创建一个父目录 /synchronizing，每个成员都监控标志（Set Watch）位目录 /synchronizing/start 是否存在，然后每个成员都加入这个队列，加入队列的方式就是创建 /synchronizing/member_i 的临时目录节点，然后每个成员获取 / synchronizing 目录的所有目录节点，也就是 member_i。判断 i 的值是否已经是成员的个数，如果小于成员个数等待 /synchronizing/start 的出现，如果已经相等就创建 /synchronizing/start。

FIFO 队列用 Zookeeper 实现思路如下：实现的思路也非常简单，就是在特定的目录下创建 SEQUENTIAL 类型的子目录 /queue_i，这样就能保证所有成员加入队列时都是有编号的，出队列时通过 getChildren( ) 方法可以返回当前所有的队列中的元素，然后消费其中最小的一个，这样就能保证 FIFO。

<br>



### zk集群工作的过程:

* recovery：这个过程泛指集群服务器的启动和恢复,因为恢复也可以理解为另一种层面上的”启动”，即需要恢复历史数据的启动
* broadcast：这是启动完毕之后,集群中的服务器开始接收客户端的连接一起工作的过程,如果客户端有修改数据的改动,那么一定会由leader广播给follower,所以称为”broadcast”

展开来说,zookeeper集群大概是这样工作的:

1. 首先每个服务器读取配置文件和数据文件,根据serverid知道本机对应的配置(就是前面那些地址和端口),并且将历史数据加载进内存中.
2.  集群中的服务器开始根据前面给出的quorum port监听集群中其他服务器的请求,并且把自己选举的leader也通知其他服务器,来来往往几回,选举出集群的一个leader.
3. 选举完leader其实还不算是真正意义上的”leader”,因为到了这里leader还需要与集群中的其他服务器同步数据,如果这一步出错,将返回2)中重新选举leader.在leader选举完毕之后,集群中的其他服务器称为”follower”,也就是都要听从leader的指令.
4. 到了这里,集群中的所有服务器,不论是leader还是follower,大家的数据都是一致的了,可以开始接收客户端的连接了.如果是读类型的请求,那么直接返回就是了,因为并不改变数据;否则,都要向leader汇报,如何通知leader呢?就是通过前面讲到的leader_listen_port.leader收到这个修改数据的请求之后,将会广播给集群中其他follower,当超过一半数量的follower有了回复,那么就相当于这个修改操作哦了,这时leader可以告诉之前的那台服务器可以给客户端一个回应了.

可以看到,上面1、2、3对应的recovery过程,而4对应的broadcast过程。

**leader所做的工作：**

leader与follower同步数据的大部分操作都在LearnerHandler线程中处理的，leader接收到的来自某个follower封包一定是FOLLOWERINFO,该封包告知了该服务器保存的数据id.之后根据这个数据id与本机保存的数据进行比较:

1. 如果数据完全一致,则发送DIFF封包告知follower当前数据就是最新的了.
2. 判断这一阶段之内有没有已经被提交的提议值,如果有,那么:
   * 如果有部分数据没有同步,那么会发送DIFF封包将有差异的数据同步过去.同时将follower没有的数据逐个发送COMMIT封包给follower要求记录下来.
   * 如果follower数据id更大,那么会发送TRUNC封包告知截除多余数据.（一台leader数据没同步就宕掉了，选举之后恢复了，数据比现在leader更新）
3. 如果这一阶段内没有提交的提议值,直接发送SNAP封包将快照同步发送给follower.
4. 消息完毕之后,发送UPTODATE封包告知follower当前数据就是最新的了,再次发送NEWLEADER封包宣称自己是leader,等待follower的响应.

**follower做的工作：**

1. 会尝试与leader建立连接,这里有一个机制,如果一定时间内没有连接上,就报错退出,重新回到选举状态.
2. 其次在发送FOLLOWERINFO封包,该封包中带上自己的最大数据id,也就是会告知leader本机保存的最大数据id.
3. 根据前面对LeaderHandler的分析,leader会根据不同的情况发送DIFF,UPTODATE,TRUNC,SNAP,依次进行处理就是了,此时follower跟leader的数据也就同步上了.
4. 由于leader端发送的最后一个封包是UPTODATE,因此在接收到这个封包之后follower结束同步数据过程,发送ACK封包回复leader.

以上过程中,任何情况出现的错误,服务器将自动将选举状态切换到LOOKING状态,重新开始进行选举.

<br>



### zk集群本身leader的选举：

如何在zookeeper集群中选举出一个leader，zookeeper使用了三种算法，具体使用哪种算法,在配置文件中是可以配置的,对应的配置项是`electionAlg`，其中1对应的是LeaderElection算法，2对应的是AuthFastLeaderElection算法，3对应的是FastLeaderElection算法，默认使用FastLeaderElection算法，所以主要分析它的选举机制。

要理解这个算法,最好需要一些 `paxos算法` 的理论基础：

1) **数据恢复阶段**：首先,每个在zookeeper服务器先读取当前保存在磁盘的数据,zookeeper中的每份数据,都有一个对应的id值,这个值是依次递增的,换言之,越新的数据,对应的ID值就越大.

2) **向其他节点发送投票值**：在读取数据完毕之后,每个zookeeper服务器发送自己选举的leader（首次选自己）,这个协议中包含了以下几部分的数据:

* 所选举leader的id(就是配置文件中写好的每个服务器的id) ,在初始阶段,每台服务器的这个值都是自己服务器的id,也就是它们都选举自己为leader.
* 服务器最大数据的id,这个值大的服务器,说明存放了更新的数据.
* 逻辑时钟的值,这个值从0开始递增,每次选举对应一个值,也就是说:  如果在同一次选举中,那么这个值应该是一致的 ;  逻辑时钟值越大,说明这一次选举leader的进程更新.
* 本机在当前选举过程中的状态,有以下几种
  * LOOKING：竞选状态
  * FOLLOWING：随从状态，同步leader状态，参与投票
  * OBSERVING：观察状态,同步leader状态，不参与投票
  * LEADING：领导者状态

3) **接受来自其他节点的数据**：每台服务器将自己服务器的以上数据发送到集群中的其他服务器之后,同样的也需要接收来自其他服务器的数据,它将做以下的处理:

1. 如果所接收数据中服务器的状态还是在选举阶段(LOOKING 状态),那么首先判断逻辑时钟值,又分以下三种情况: 

   * 如果发送过来的逻辑时钟大于目前的逻辑时钟,那么说明这是更新的一次选举,此时需要更新一下本机的逻辑时钟值,同时将之前收集到的来自其他服务器的选举清空,因为这些数据已经不再有效了.然后判断是否需要更新当前自己的选举情况.在这里是根据选举leader id,保存的最大数据id来进行判断的,这两种数据之间对这个选举结果的影响的权重关系是:首先看数据id,数据id大者胜出;其次再判断leader id,leader id大者胜出.然后再将自身最新的选举结果(也就是上面提到的三种数据）广播给其他服务器).
   * 发送过来数据的逻辑时钟小于本机的逻辑时钟，说明对方在一个相对较早的选举进程中,这里只需要将本机的数据发送过去就是了
   * 两边的逻辑时钟相同,此时也只是调用totalOrderPredicate函数判断是否需要更新本机的数据,如果更新了再将自己最新的选举结果广播出去就是了.

   然后再处理两种情况:

   * 服务器判断是不是已经收集到了所有服务器的选举状态,如果是，那么这台服务器选举的leader就定下来了，然后根据选举结果设置自己的角色(FOLLOWING还是LEADER),然后退出选举过程就是了.
   * 即使没有收集到所有服务器的选举状态,也可以根据该节点上选择的最新的leader是不是得到了超过半数以上服务器的支持,如果是,那么当前线程将被阻塞等待一段时间(这个时间在finalizeWait定义)看看是不是还会收到当前leader的数据更优的leader,如果经过一段时间还没有这个新的leader提出来，那么这台服务器最终的leader就确定了,否则进行下一次选举. 

2. 如果所接收服务器不在选举状态,也就是在FOLLOWING或者LEADING状态，做以下两个判断:

   * 如果逻辑时钟相同,将该数据保存到recvset,如果所接收服务器宣称自己是leader,那么将判断是不是有半数以上的服务器选举它,如果是则设置选举状态退出选举过程
   * 否则这是一条与当前逻辑时钟不符合的消息,那么说明在另一个选举过程中已经有了选举结果,于是将该选举结果加入到outofelection集合中,再根据outofelection来判断是否可以结束选举,如果可以也是保存逻辑时钟,设置选举状态,退出选举过程.

<br>

**以一个简单的例子来说明整个选举的过程：**

假设有五台服务器组成的zookeeper集群,它们的id从1-5,同时它们都是最新启动的,也就是没有历史数据,在存放数据量这一点上,都是一样的.假设这些服务器依序启动,来看看会发生什么.

1. 服务器1启动,此时只有它自己启动了,它发出去的报告没有任何响应,所以它的选举状态一直是LOOKING状态
2. 服务器2启动,它与最开始启动的服务器1进行通信,互相交换自己的选举结果,由于两者都没有历史数据,所以id值较大的服务器2胜出,但是由于没有达到超过半数以上的服务器都同意选举它(这个例子中的半数以上是3),所以服务器1,2还是继续保持LOOKING状态.
3. 服务器3启动,根据前面的理论分析,服务器3成为服务器1,2,3中的老大,而与上面不同的是,此时有三台服务器选举了它,所以它成为了这次选举的leader.
4. 服务器4启动,根据前面的分析,理论上服务器4应该是服务器1,2,3,4中最大的,但是由于前面已经有半数以上的服务器选举了服务器3,所以它只能接收当小弟的命了.
5. 服务器5启动,同4一样,当小弟.

以上就是fastleader算法的简要分析,还有一些异常情况的处理,比如某台服务器宕机之后的处理,当leader宕机之后的处理等等。



### 安装主要流程

1、配置hosts、hostname，测试各节点网络的连通性<br>
2、解压zookeepr-3.4.6.tar.gz，编辑/etc/profile配置环境变量（注意. /etc/profile 或 source /etc/profile）<br>
3、修改zookeeper篇日志文件zoo.cfg<br>

```shell
ubuntu@supervisor01z:~$ vi zookeeper-3.4.6/conf/zoo.cfg 

#表示zk的一个基本时间单元，可用它的倍数来表示系统内部的时间间隔设置
tickTime=2000
#表示follower连接到leader，初始化连接时最长能忍受多少个心跳的的时间间隔，这里的10表示，当已经超过了10个
#tickTime心跳的时间，zk服务器还没有收到客户端的返回信息，表明这个客户端连接失败
initLimit=10	
#标识Leader与Follower之间发送消息，请求和应答最长允许时间
syncLimit=5

#用于配置存储快照文件的目录，如果没有配置dataLogDir，那么事务日志也会存储在此目录
dataDir=/home/ubuntu/zookeeper-3.4.6/zkdata
#日志的存放路径
dataLogDir=/home/ubuntu/zookeeper-3.4.6/logs
#自动清理snapshot和事务日志(单位为小时,默认为0表示不开启)
autopurge.purgeInterval=1
#这个参数和上面的参数搭配使用，这个参数指定了需要保留的文件数目。默认是保留3个
autopurge.snapRetainCount=3

#zk的运行端口，默认是2181
clientPort=2181
#server.A=host:port1:port2 
#其中A是一个数字，表示这个是第几号服务器
#host表示这个服务器的地址
#port1是follower服务器和leader服务器的通信端口
#port2是leader服务器选举过程中的投票通信端口
server.1=nimbusz:2888:3888
server.2=supervisor01z:2888:3888
server.3=supervisor02z:2888:3888
```

<br>

4、建立zookeeper节点标示文件myid<br>

```shell
ubuntu@nimbusz:~$ echo “1” > /home/ubuntu/zookeeper-3.4.6/zkdata/myid
ubuntu@supervisor01z:~$ echo “2” > /home/ubuntu/zookeeper-3.4.6/zkdata/myid
ubuntu@supervisor02z:~$ echo “3” > /home/ubuntu/zookeeper-3.4.6/zkdata/myid
```

<br>

5、启动zookeeper，并且进行状态监控<br>

```shell
zkServer.sh start 或者 zkServer.sh start-foreground
```

<br>

6、查看zookeeper节点状态<br>

```shell
zkServer.sh status
  //如果被选举为leader，则状态为 leader
  //如果没有被选举为leader，则状态一般为 follower
```
<br>

7、客户端连接zk服务器<br>

```shell
zookeeper-3.4.6/bin/zkCli.sh
zookeeper-3.4.6/bin/zkCli.sh -server 172.16.127.129:2181
zookeeper-3.4.6/bin/zkCli.sh -timeout 5000 -r -server 172.16.127.129:2181 #单位ms，-r只读
```

<br>



### 常用zookeeper命令

```shell
zk服务命令:
在准备好相应的配置之后，可以直接通过zkServer.sh 这个脚本进行服务的相关操作
  1. 启动ZK服务:    zkServer.sh start
  2. 查看ZK服务状态: zkServer.sh status
  3. 停止ZK服务:    zkServer.sh stop
  4. 重启ZK服务:    zkServer.sh restart
  
  
zk常用四字命令:
ZooKeeper支持某些特定的四字命令字母与其的交互。它们大多是查询命令，用来获取 ZooKeeper 服务的当前状态及相关信息。用户在客户端可以通过 telnet 或 nc 向 ZooKeeper 提交相应的命令。
  1. echo stat | nc 127.0.0.1 2181  #查看哪个节点被选择作为follower或者leader
  2. echo ruok | nc 127.0.0.1 2181  #测试是否启动了该  Server，若回复imok表示已经启动。
  3. echo dump | nc 127.0.0.1 2181  #列出未经处理的会话和临时节点。
  4. echo kill | nc 127.0.0.1 2181  #关掉server
  5. echo conf | nc 127.0.0.1 2181  #输出相关服务配置的详细信息。
  6. echo cons | nc 127.0.0.1 2181  #列出所有连接到服务器的客户端的完全的连接 / 会话的详细信息。
  7. echo envi |nc 127.0.0.1 2181   #输出关于服务环境的详细信息（区别于 conf 命令）。
  8. echo reqs | nc 127.0.0.1 2181  #列出未经处理的请求。
  9. echo wchs | nc 127.0.0.1 2181  #列出服务器 watch 的详细信息。
  10. echo wchc | nc 127.0.0.1 2181 #通过 session 列出服务器 watch 的详细信息，输出会话的列表。
  11. echo wchp | nc 127.0.0.1 2181 #通过路径列出服务器 watch 的详细信息,输出session相关路径。


zk客户端命令:
ZooKeeper命令行工具类似于Linux的shell环境，不过功能肯定不及shell啦，但是使用它我们可以简单的对ZooKeeper进行访问，数据创建，数据修改等操作。
命令行所有操作如下：
		connect host:port
        get path [watch]
        ls path [watch]
        set path data [version]
        rmr path
        delquota [-n|-b] path
        quit 
        printwatches on|off
        create [-s] [-e] path data acl
        stat path [watch]
        close 
        ls2 path [watch]
        history 
        listquota path
        setAcl path acl
        getAcl path
        sync path
        redo cmdno
        addauth scheme auth
        delete path [version]
        setquota -n|-b val path
```
<br>



### zk集群中的基本概念

* `集群角色`： leader、follower、observer

* `会话`：指客户端与zk服务器的连接，客户端与服务端建立一个TCP长连接来维持一个session，客户端再启动的饿时候会首先建立一个tcp长连接，通过这个连接，客户端能够通过心跳检测与服务器保持有效的会话，也能向zk服务器发送请求并获取响应。

* `数据节点`：集群中的节点分为两类，集群中的一台机器称为一个zk节点，zk的数据模型是一棵树，树的节点就是znode，znode中可以保存数据

* `版本`：

  * version：当前数据节点数据内容的版本号
  * cversion：当前数据节点子节点的版本号
  * aversion：当前数据节点acl（版本控制列表）变更版本号

  悲观锁和乐观锁：悲观锁又叫悲观并发锁，是数据库中非常严格的锁策略，具有强烈的排他性，能够避免不同事务对同一数据并发更新所造成的数据不一致，在一个事务没有完成之前，下一个事务不能访问相同的资源，适合数据更新竞争非常激烈的场景。

  相比悲观锁，乐观锁使用的场景会更多，悲观所认为事务访问相同数据的时候一定会出现相互的干扰，所以简单粗暴的使用排他的方式，而乐观锁认为不同事务访问相同资源是很少出现相互干扰的情况，因此在事务处理期间不需要进行并发控制。当然乐观锁也是锁，他还是会有并发的控制！对数据库我们通常的做法是在每个表中增加一个version字段，事务修改数据之前先读出数据版本，然后把这个读出来的版本号加入到更新语句的条件中，如果更新失败了，说明其他事务已经对这条数据进行了修改，那么当前事务需要抛出异常给客户端，让客户端自行处理（比如客户端可以选择重试）。

* `watcher`：zk的事件监听器，zk允许用户在指定节点上注册一些watcher、当数据节点节点发生变化的时候，zk服务器把这个变化的通知发送给感兴趣的客户端。

* `ACL权限控制`：这是 Access Control Lists 的简写，zk采用acl策略来进行权限控制，有以下权限：

  * CREATE：创建子节点的权限
  * READ：获取节点数据和字节点的权限
  * WRITE：更新节点数据的权限
  * DELETE：删除子节点的权限
  * ADMIN：删除节点ACL的权限

