---
layout: post
title:  "zookeeper安装部署主要流程（以zookeeper-3.4.6为例）"
desc: "zookeeper安装部署主要流程（以zookeeper-3.4.6为例）"
keywords: "zookeeper,安装部署,linux,kinglyjn,张庆力"
date: 2016-12-13
categories: [Linux]
tags: [Linux]
icon: fa-bookmark-o
---

### 安装主要流程
1、配置hosts、hostname，测试各节点网络的连通性<br>
2、解压zookeepr-3.4.6.tar.gz，编辑/etc/profile配置环境变量（注意. /etc/profile 或 source /etc/profile）<br>
3、修改zookeeper篇日志文件zoo.cfg<br>

```shell
ubuntu@supervisor01z:~$ vi zookeeper-3.4.6/conf/zoo.cfg 
initLimit=10
syncLimit=5
dataDir=/home/ubuntu/zookeeper-3.4.6/zkdata

#日志的存放路径
dataLogDir=/home/ubuntu/zookeeper-3.4.6/logs
#自动清理snapshot和事务日志(单位为小时,默认为0表示不开启)
autopurge.purgeInterval=1
#这个参数和上面的参数搭配使用，这个参数指定了需要保留的文件数目。默认是保留3个
autopurge.snapRetainCount=3

clientPort=2181
server.1=nimbusz:2888:3888
server.2=supervisor01z:2888:3888
server.3=supervisor02z:2888:3888
```

4、建立zookeeper节点标示文件myid<br>

```shell
ubuntu@nimbusz:~$ echo “1” > /home/ubuntu/zookeeper-3.4.6/zkdata/myid
ubuntu@supervisor01z:~$ echo “2” > /home/ubuntu/zookeeper-3.4.6/zkdata/myid
ubuntu@supervisor02z:~$ echo “3” > /home/ubuntu/zookeeper-3.4.6/zkdata/myid
```

5、启动zookeeper，并且进行状态监控<br>

```shell
zkServer.sh start 或者 zkServer.sh start-foreground
```

6、查看zookeeper节点状态<br>

```shell
zkServer.sh status
  //如果被选举为leader，则状态为 leader
  //如果没有被选举为leader，则状态一般为 follower
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
    
zk客户端命令:
ZooKeeper命令行工具类似于Linux的shell环境，不过功能肯定不及shell啦，但是使用它我们可以简单的对ZooKeeper进行访问，数据创建，数据修改等操作。使用 zkCli.sh -server 127.0.0.1:2181 连接到 ZooKeeper 服务，连接成功后，系统会输出 ZooKeeper 的相关环境以及配置信息。
命令行工具的一些简单操作如下：
  1. 显示根目录下、文件： ls / 使用 ls 命令来查看当前 ZooKeeper 中所包含的内容
  2. 显示根目录下、文件： ls2 / 查看当前节点数据并能看到更新次数等数据
  3. 创建文件，并设置初始内容： create /zk "test" 创建一个新的 znode节点“ zk ”以及与它关联的字符串
  4. 获取文件内容： get /zk 确认 znode 是否包含我们所创建的字符串
  5. 修改文件内容： set /zk "zkbak" 对 zk 所关联的字符串进行设置
  6. 删除文件： delete /zk 将刚才创建的 znode 删除
  7. 退出客户端： quit
  8. 帮助命令： help

zk常用四字命令：
ZooKeeper支持某些特定的四字命令字母与其的交互。它们大多是查询命令，用来获取 ZooKeeper 服务的当前状态及相关信息。用户在客户端可以通过 telnet 或 nc 向 ZooKeeper 提交相应的命令。
  1. 可以通过命令：echo stat|nc 127.0.0.1 2181 来查看哪个节点被选择作为follower或者leader
  2. 使用echo ruok|nc 127.0.0.1 2181 测试是否启动了该  Server，若回复imok表示已经启动。
  3. echo dump| nc 127.0.0.1 2181 ,列出未经处理的会话和临时节点。
  4. echo kill | nc 127.0.0.1 2181 ,关掉server
  5. echo conf | nc 127.0.0.1 2181 ,输出相关服务配置的详细信息。
  6. echo cons | nc 127.0.0.1 2181 ,列出所有连接到服务器的客户端的完全的连接 / 会话的详细信息。
  7. echo envi |nc 127.0.0.1 2181 ,输出关于服务环境的详细信息（区别于 conf 命令）。
  8. echo reqs | nc 127.0.0.1 2181 ,列出未经处理的请求。
  9. echo wchs | nc 127.0.0.1 2181 ,列出服务器 watch 的详细信息。
  10. echo wchc | nc 127.0.0.1 2181 ,通过 session 列出服务器 watch 的详细信息，它的输出是一个与 watch 相关的会话的列表。
  11. echo wchp | nc 127.0.0.1 2181 ,通过路径列出服务器 watch 的详细信息。它输出一个与 session 相关的路径。
```
<br>

### 运行效果图
<img src="http://img.blog.csdn.net/20161213112207892?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:80%;">



***

### Zookeeper-Zookeeper可以干什么

在Zookeeper的官网上有这么一句话：ZooKeeper is a centralized service for maintaining configuration information, naming, providing distributed synchronization, and providing group services. 

这大概描述了Zookeeper主要可以干哪些事情：配置管理，名字服务，提供分布式同步以及集群管理。那这些服务又到底是什么呢？我们为什么需要这样的服务？我们又为什么要使用Zookeeper来实现呢，使用Zookeeper有什么优势？接下来我会挨个介绍这些到底是什么，以及有哪些开源系统中使用了。

*配置管理*

在我们的应用中除了代码外，还有一些就是各种配置。比如数据库连接等。一般我们都是使用配置文件的方式，在代码中引入这些配置文件。但是当我们只有一种配置，只有一台服务器，并且不经常修改的时候，使用配置文件是一个很好的做法，但是如果我们配置非常多，有很多服务器都需要这个配置，而且还可能是动态的话使用配置文件就不是个好主意了。这个时候往往需要寻找一种集中管理配置的方法，我们在这个集中的地方修改了配置，所有对这个配置感兴趣的都可以获得变更。比如我们可以把配置放在数据库里，然后所有需要配置的服务都去这个数据库读取配置。但是，因为很多服务的正常运行都非常依赖这个配置，所以需要这个集中提供配置服务的服务具备很高的可靠性。一般我们可以用一个集群来提供这个配置服务，但是用集群提升可靠性，那如何保证配置在集群中的一致性呢？ 这个时候就需要使用一种实现了一致性协议的服务了。Zookeeper就是这种服务，它使用Zab这种一致性协议来提供一致性。现在有很多开源项目使用Zookeeper来维护配置，比如在HBase中，客户端就是连接一个Zookeeper，获得必要的HBase集群的配置信息，然后才可以进一步操作。还有在开源的消息队列Kafka中，也使用Zookeeper来维护broker的信息。在Alibaba开源的SOA框架Dubbo中也广泛的使用Zookeeper管理一些配置来实现服务治理。

*分布式锁*

其实在第一篇文章中已经介绍了Zookeeper是一个分布式协调服务。这样我们就可以利用Zookeeper来协调多个分布式进程之间的活动。比如在一个分布式环境中，为了提高可靠性，我们的集群的每台服务器上都部署着同样的服务。但是，一件事情如果集群中的每个服务器都进行的话，那相互之间就要协调，编程起来将非常复杂。而如果我们只让一个服务进行操作，那又存在单点。通常还有一种做法就是使用分布式锁，在某个时刻只让一个服务去干活，当这台服务出问题的时候锁释放，立即fail over到另外的服务。这在很多分布式系统中都是这么做，这种设计有一个更好听的名字叫Leader Election(leader选举)。比如HBase的Master就是采用这种机制。但要注意的是分布式锁跟同一个进程的锁还是有区别的，所以使用的时候要比同一个进程里的锁更谨慎的使用。

*集群管理*

在分布式的集群中，经常会由于各种原因，比如硬件故障，软件故障，网络问题，有些节点会进进出出。有新的节点加入进来，也有老的节点退出集群。这个时候，集群中其他机器需要感知到这种变化，然后根据这种变化做出对应的决策。比如我们是一个分布式存储系统，有一个中央控制节点负责存储的分配，当有新的存储进来的时候我们要根据现在集群目前的状态来分配存储节点。这个时候我们就需要动态感知到集群目前的状态。还有，比如一个分布式的SOA架构中，服务是一个集群提供的，当消费者访问某个服务时，就需要采用某种机制发现现在有哪些节点可以提供该服务(这也称之为服务发现，比如Alibaba开源的SOA框架Dubbo就采用了Zookeeper作为服务发现的底层机制)。还有开源的Kafka队列就采用了Zookeeper作为Cosnumer的上下线管理。


### Zookeeper-Zookeeper的配置

*配置-zoo.cfg*

这是zookeeper的主要配置文件，因为Zookeeper是一个集群服务，集群的每个节点都需要这个配置文件。为了避免出差错，zoo.cfg这个配置文件里没有跟特定节点相关的配置，所以每个节点上的这个zoo.cfg都是一模一样的配置。这样就非常便于管理了，比如我们可以把这个文件提交到版本控制里管理起来。其实这给我们设计集群系统的时候也是个提示：集群系统一般有很多配置，应该尽量将通用的配置和特定每个服务的配置(比如服务标识)分离，这样通用的配置在不同服务之间copy就ok了。ok，下面来介绍一些配置点：

```shell
clientPort=2181
client port，顾名思义，就是客户端连接zookeeper服务的端口。这是一个TCP port。
dataDir=/data
dataLogDir=/datalog
```

dataLogDir如果没提供的话使用的则是dataDir。zookeeper的持久化都存储在这两个目录里。dataLogDir里是放到的顺序日志(WAL)。而dataDir里放的是内存数据结构的snapshot，便于快速恢复。为了达到性能最大化，一般建议把dataDir和dataLogDir分到不同的磁盘上，这样就可以充分利用磁盘顺序写的特性。下面是集群中服务的列表:

```shell
server.1=127.0.0.1:20881:30881
server.2=127.0.0.1:20882:30882
server.3=127.0.0.1:20883:30883
```

在上面的例子中，我把三个zookeeper服务放到同一台机器上。上面的配置中有两个TCP port。后面一个是用于Zookeeper选举用的，而前一个是Leader和Follower或Observer交换数据使用的。我们还注意到server.后面的数字。这个就是myid(关于myid是什么下一节会介绍)。

上面这几个是一些基本配置。还有像 tickTime，这是个时间单位定量。比如tickTime=1000，这就表示在zookeeper里1 tick表示1000 ms，所有其他用到时间的地方都会用多少tick来表示。比如 syncLimit = 2 就表示fowller与leader的心跳时间是2 tick。

maxClientCnxns -- 对于一个客户端的连接数限制，默认是60，这在大部分时候是足够了。但是在我们实际使用中发现，在测试环境经常超过这个数，经过调查发现有的团队将几十个应用全部部署到一台机器上，以方便测试，于是这个数字就超过了。

minSessionTimeout, maxSessionTimeout-- 一般，客户端连接zookeeper的时候，都会设置一个session timeout，如果超过这个时间client没有与zookeeper server有联系，则这个session会被设置为过期(如果这个session上有临时节点，则会被全部删除，这就是实现集群感知的基础，后面的文章会介绍这一点)。但是这个时间不是客户端可以无限制设置的，服务器可以设置这两个参数来限制客户端设置的范围。

autopurge.snapRetainCount，autopurge.purgeInterval -- 客户端在与zookeeper交互过程中会产生非常多的日志，而且zookeeper也会将内存中的数据作为snapshot保存下来，这些数据是不会被自动删除的，这样磁盘中这样的数据就会越来越多。不过可以通过这两个参数来设置，让zookeeper自动删除数据。autopurge.purgeInterval就是设置多少小时清理一次。而autopurge.snapRetainCount是设置保留多少个snapshot，之前的则删除。

不过如果你的集群是一个非常繁忙的集群，然后又碰上这个删除操作，可能会影响zookeeper集群的性能，所以一般会让这个过程在访问低谷的时候进行，但是遗憾的是zookeeper并没有设置在哪个时间点运行的设置，所以有的时候我们会禁用这个自动删除的功能，而在服务器上配置一个cron，然后在凌晨来干这件事。

以上就是zoo.cfg里的一些配置了。下面就来介绍myid。
配置-myid:
在dataDir里会放置一个myid文件，里面就一个数字，用来唯一标识这个服务。这个id是很重要的，一定要保证整个集群中唯一。zookeeper会根据这个id来取出server.x上的配置。比如当前id为1，则对应着zoo.cfg里的server.1的配置。而且在后面我们介绍leader选举的时候，这个id的大小也是有意义的。OK，上面就是配置的讲解了，现在我们可以启动zookeeper集群了。进入到bin目录，执行 ./zkServer.sh start即可。


### Zookeeper-Zookeeper启动过程

Zookeeper的启动入口在org.apache.zookeeper.server.quorum.QuorumPeerMain。在这个类的main方法里进入了zookeeper的启动过程，首先我们会解析配置文件，即zoo.cfg和myid。这样我们就知道了dataDir和dataLogDir指向哪儿了，然后就可以启动日志清理任务了(如果配置了的话)。

```java
DatadirCleanupManager purgeMgr = new DatadirCleanupManager(config
                .getDataDir(), config.getDataLogDir(), config
                .getSnapRetainCount(), config.getPurgeInterval());
purgeMgr.start();
```

接下来会初始化ServerCnxnFactory，这个是用来接收来自客户端的连接的，也就是这里启动的是一个tcp server。在Zookeeper里提供两种tcp server的实现，一个是使用java原生NIO的方式，另外一个是使用Netty。默认是java nio的方式，一个典型的Reactor模型。因为java nio编程并不是本文的重点，所以在这里就只是简单的介绍一下。

```java
//首先根据配置创建对应factory的实例:NIOServerCnxnFactory 或者 NettyServerCnxnFactory
ServerCnxnFactory cnxnFactory = ServerCnxnFactory.createFactory();
//初始化配置
cnxnFactory.configure(config.getClientPortAddress(),config.getMaxClientCnxns());
```

创建几个SelectorThread处理具体的数据读取和写出。
先是创建ServerSocketChannel，bind等:

```java
this.ss =  ServerSocketChannel.open();
ss.socket().setReuseAddress(true);
ss.socket().bind(addr);
ss.configureBlocking(false);
```

然后创建一个AcceptThread线程来接收客户端的连接。
这一部分就是处理客户端请求的模块了，如果遇到有客户端请求的问题可以看看这部分。
接下来就进入初始化的主要部分了，首先会创建一个QuorumPeer实例，这个类就是表示zookeeper集群中的一个节点。初始化QuorumPeer的时候有这么几个关键点：

1. 初始化FileTxnSnapLog，这个类主要管理Zookeeper中的操作日志(WAL)和snapshot。
2. 初始化ZKDatabase，这个类就是Zookeeper的目录结构在内存中的表示，所有的操作最后都会映射到这个类上面来。
3. 初始化决议validator(QuorumVerifier->QuorumMaj) (其实这一步，是在配置)。这一步是从zoo.cfg的server.n这一部分初始化出集群的成员出来，有哪些需要参与投票(follower)，有哪些只是observer。还有决定half是多少等，这些都是zookeeper的核心。在这一步，对于每个节点会初始化一个QuorumServer对象，并且放到allMembers，votingMembers，observingMembers这几个map里。而且这里也对参与者的个数进行了一些判断。
4. leader选举 这一步非常重要，也是zookeeper里最复杂而最精华的一部分。

到这里，我们的zookeeper就启动完成了。后面我将会分三部分进一步深入理解zookeeper：
1. leader选举
2. 存储
3. 处理客户端请求


### Zookeeper-Zookeeper leader选举

那么什么是leader选举呢？zookeeper为什么需要leader选举呢？zookeeper的leader选举的过程又是什么样子的？下面的目的就是解决这三个问题。

首先我们来看看什么是leader选举。其实这个很好理解，leader选举就像总统选举一样，每人一票，获得多数票的人就当选为总统了。在zookeeper集群中也是一样，每个节点都会投票，如果某个节点获得超过半数以上的节点的投票，则该节点就是leader节点了。

国家选举总统是为了选一个最高统帅，治理国家。那么zookeeper集群选举的目的又是什么呢？其实这个要清楚明白的解释还是挺复杂的。我们可以简单点想这个问题：我们有一个zookeeper集群，有好几个节点。每个节点都可以接收请求，处理请求。那么，如果这个时候分别有两个客户端向两个节点发起请求，请求的内容是修改同一个数据。比如客户端c1，请求节点n1，请求是set a = 1; 而客户端c2，请求节点n2，请求内容是set a = 2;

那么最后a是等于1还是等于2呢？ 这在一个分布式环境里是很难确定的。解决这个问题有很多办法，而zookeeper的办法是，我们选一个总统出来，所有的这类决策都提交给总统一个人决策，那之前的问题不就没有了么。

那我们现在的问题就是怎么来选择这个总统呢？ 在现实中，选择总统是需要宣讲拉选票的，那么在zookeeper的世界里这又如何处理呢？我们还是show code吧。

在QuorumPeer的startLeaderElection方法里包含leader选举的逻辑。Zookeeper默认提供了4种选举方式，默认是第4种: FastLeaderElection。

我们先假设我们这是一个崭新的集群，崭新的集群的选举和之前运行过一段时间的选举是有稍许不同的，后面会提及。

节点状态： 每个集群中的节点都有一个状态 LOOKING, FOLLOWING, LEADING, OBSERVING。都属于这4种，每个节点启动的时候都是LOOKING状态，如果这个节点参与选举但最后不是leader，则状态是FOLLOWING，如果不参与选举则是OBSERVING，leader的状态是LEADING。

开始这个选举算法前，每个节点都会在zoo.cfg上指定的监听端口启动监听(server.1=127.0.0.1:20881:20882)，这里的20882就是这里用于选举的端口。

在FastLeaderElection里有一个Messenger的内部类，这个类里有启动了两个线程：WorkerReceiver， WorkerSender。为什么说选举这部分复杂呢，我觉得就是这些线程就像左右互搏一样，非常难以理解。顾名思义，这两个线程一个是处理从别的节点接收消息的，一个是向外发送消息的。对于外面的逻辑接收和发送的逻辑都是异步的。

这里配置好了，QuorumPeer的run方法就开始执行了，这里实现的是一个简单的状态机。因为现在是LOOKING状态，所以进入LOOKING的分支，调用选举算法开始选举了：

```java
setCurrentVote(makeLEStrategy().lookForLeader());
```

而在lookForLeader里主要是干什么呢？首先我们会更新一下一个叫逻辑时钟的东西，这也是在分布式算法里很重要的一个概念，但是在这里先不介绍，可以参考后面的论文。然后决定我要投票给谁。不过zookeeper这里的选举真直白，每个节点都选自己(汗),选我，选我，选我...... 然后向其他节点广播这个选举信息。这里实际上并没有真正的发送出去，只是将选举信息放到由WorkerSender管理的一个队列里。

```java
synchronized(this){
    //逻辑时钟           
    logicalclock++;
    //getInitLastLoggedZxid(), getPeerEpoch()这里先不关心是什么，后面会讨论
    updateProposal(getInitId(), getInitLastLoggedZxid(), getPeerEpoch());
}

//getInitId() 即是获取选谁，id就是myid里指定的那个数字，所以说一定要唯一
private long getInitId(){
        if(self.getQuorumVerifier().getVotingMembers().containsKey(self.getId()))       
            return self.getId();
        else return Long.MIN_VALUE;
}

//发送选举信息，异步发送
sendNotifications();
```

现在我们去看看怎么把投票信息投递出去。这个逻辑在WorkerSender里，WorkerSender从sendqueue里取出投票，然后交给QuorumCnxManager发送。因为前面发送投票信息的时候是向集群所有节点发送，所以当然也包括自己这个节点，所以QuorumCnxManager的发送逻辑里会判断，如果这个要发送的投票信息是发送给自己的，则不发送了，直接进入接收队列。

```java
public void toSend(Long sid, ByteBuffer b) {
    if (self.getId() == sid) {
         b.position(0);
         addToRecvQueue(new Message(b.duplicate(), sid));
    } else {
         //发送给别的节点，判断之前是不是发送过
         if (!queueSendMap.containsKey(sid)) {
             //这个SEND_CAPACITY的大小是1，所以如果之前已经有一个还在等待发送，则会把之前的一个删除掉，发送新的
             ArrayBlockingQueue<ByteBuffer> bq = new ArrayBlockingQueue<ByteBuffer>(SEND_CAPACITY);
             queueSendMap.put(sid, bq);
             addToSendQueue(bq, b);

         } else {
             ArrayBlockingQueue<ByteBuffer> bq = queueSendMap.get(sid);
             if(bq != null){
                 addToSendQueue(bq, b);
             } else {
                 LOG.error("No queue for server " + sid);
             }
         }
         //这里是真正的发送逻辑了
         connectOne(sid);
            
    }
}
```

connectOne就是真正发送了。在发送之前会先把自己的id和选举地址发送过去。然后判断要发送节点的id是不是比自己的id大，如果大则不发送了。如果要发送又是启动两个线程：SendWorker,RecvWorker(这种一个进程内许多不同种类的线程，各自干活的状态真的很难理解)。发送逻辑还算简单，就是从刚才放到那个queueSendMap里取出，然后发送。并且发送的时候将发送出去的东西放到一个lastMessageSent的map里，如果queueSendMap里是空的，就发送lastMessageSent里的东西，确保对方一定收到了。

看完了SendWorker的逻辑，再来看看数据接收的逻辑吧。还记得前面提到的有个Listener在选举端口上启动了监听么，现在这里应该接收到数据了。我们可以看到receiveConnection方法。在这里，如果接收到的的信息里的id比自身的id小，则断开连接，并尝试发送消息给这个id对应的节点(当然，如果已经有SendWorker在往这个节点发送数据，则不用了)。

如果接收到的消息的id比当前的大，则会有RecvWorker接收数据，RecvWorker会将接收到的数据放到recvQueue里。

而FastLeaderElection的WorkerReceiver线程里会不断地从这个recvQueue里读取Message处理。在WorkerReceiver会处理一些协议上的事情，比如消息格式等。除此之外还会看看接收到的消息是不是来自投票成员。如果是投票成员，则会看看这个消息里的状态，如果是LOOKING状态并且当前的逻辑时钟比投票消息里的逻辑时钟要高，则会发个通知过去，告诉谁是leader。在这里，刚刚启动的崭新集群，所以逻辑时钟基本上都是相同的，所以这里还没判断出谁是leader。不过在这里我们注意到如果当前节点的状态是LOOKING的话，接收逻辑会将接收到的消息放到FastLeaderElection的recvqueue里。而在FastLeaderElection会从这个recvqueue里读取东西。

这里就是选举的主要逻辑了：totalOrderPredicate

```java
protected boolean totalOrderPredicate(long newId, long newZxid, long newEpoch, long curId, long curZxid, long curEpoch) {
    return ((newEpoch > curEpoch) || 
                ((newEpoch == curEpoch) &&
                ((newZxid > curZxid) || ((newZxid == curZxid) && (newId > curId)))));
}
```

1. 判断消息里的epoch是不是比当前的大，如果大则消息里id对应的server我就承认它是leader
2. 如果epoch相等则判断zxid，如果消息里的zxid比我的大我就承认它是leader
3. 如果前面两个都相等那就比较一下server id吧，如果比我的大我就承认它是leader。

关于前面两个东西暂时我们不去关心它，对于新启动的集群这两者都是相等的。
那这样看来server id的大小也是leader选举的一环啊（有的人生下来注定就不平凡，这都是命啊）。
最后我们来看看，很多文章所介绍的，如果超过一半的人说它是leader，那它就是leader的逻辑吧

```java
private boolean termPredicate(
            HashMap<Long, Vote> votes,
            Vote vote) {
    HashSet<Long> set = new HashSet<Long>();
    //遍历已经收到的投票集合，将等于当前投票的集合取出放到set中
    for (Map.Entry<Long,Vote> entry : votes.entrySet()) {
        if (self.getQuorumVerifier().getVotingMembers().containsKey(entry.getKey())
                && vote.equals(entry.getValue())){
            set.add(entry.getKey());
        }
    }
    
    //统计set，也就是投某个id的票数是否超过一半
    return self.getQuorumVerifier().containsQuorum(set);
}

public boolean containsQuorum(Set<Long> ackSet) {
    return (ackSet.size() > half);
}
```


最后一关：如果选的是自己，则将自己的状态更新为LEADING，否则根据type，要么是FOLLOWING，要么是OBSERVING。到这里选举就结束了。

这里介绍的是一个新集群启动时候的选举过程，启动的时候就是根据zoo.cfg里的配置，向各个节点广播投票，一般都是选投自己。然后收到投票后就会进行进行判断。如果某个节点收到的投票数超过一半，那么它就是leader了。 

了解了这个过程，我们来看看另外一个问题：一个集群有3台机器，挂了一台后的影响是什么？挂了两台呢？ 

挂了一台：挂了一台后就是收不到其中一台的投票，但是有两台可以参与投票，按照上面的逻辑，它们开始都投给自己，后来按照选举的原则，两个人都投票给其中一个，那么就有一个节点获得的票等于2，2 > (3/2)=1 的，超过了半数，这个时候是能选出leader的。

挂了两台： 挂了两台后，怎么弄也只能获得一张票， 1 不大于 (3/2)=1的，这样就无法选出一个leader了。

在前面介绍时，为了简单我假设的是这是一个崭新的刚启动的集群，这样的集群与工作一段时间后的集群有什么不同呢？不同的就是epoch和zxid这两个参数。在新启动的集群里这两个一般是相等的，而工作一段时间后这两个参数有可能有的节点落后其他节点。


### Zookeeper-Zookeeper client

Zookeeper的client是通过Zookeeper类提供的。前面曾经说过，Zookeeper给使用者提供的是一个类似操作系统的文件结构，只不过这个结构是分布式的。可以理解为一个分布式的文件系统。我们可以通过Zookeeper来访问这个分布式的文件系统。Zookeeper的client api给我们提供以下这些API：

*create*

在给定的path上创建节点，这个path就像文件系统的路径，比如/myapp/data/1，在创建节点的时候还可以指定节点的类型：是永久节点，永久顺序节点，临时节点，临时顺序节点。这个节点类型是非常强大的。永久节点一经创建就永久保留了，就像我们在文件系统上创建一个普通文件，这个文件的生命周期跟创建它的应用没有任何关系。而临时节点呢，当创建这个临时节点的应用与zookeeper之间的会话过期之后就会被zookeeper自动删除了。这个特性是实现很多功能的关键。比如我们做集群感知，我们的应用启动的时候将自己的ip地址作为临时节点创建在某个节点下面。当我们的应用因为某些原因，比如网络断掉或者宕机，它与zookeeper的会话就会过期了，过期后这个临时节点就删除了。这样我们就可以通过这个特性来感知到我们的服务的集群有哪些机器是活者的。那么顺序节点又是什么呢。一般，如果我们在指定的path上创建节点，如果这个节点已经被创建了，则会抛出一个NodeExistsException的异常。如果我们在指定的路径上创建顺序节点，则Zookeeper会自动的在我们给定的path上加上一个顺序编号。这个特性就是实现分布式锁的关键。假设我们有几个节点共享一个资源，我们这几个节点都想争用这个资源，那我们就都向某个路径创建临时顺序节点。然后顺序最小的那个就获得锁，然后如果某个节点释放了锁，那顺序第二小的那个就获得锁，以此类推，这样一个分布式的公平锁就实现了。除此之外，每个节点上还可以保存一些数据。

*delete* 
删除给定节点。删除节点的时候还可以给定一个version，只有路径和version都匹配的时候节点才会被删除。有了这个version在分布式环境种我们就可以用乐观锁的方式来确保一致性。比如我们先读取一下节点，获得了节点的version，然后删除，如果删除成功了则说明在这之间没有人操作过这个节点，否则就是并发冲突了。

*exists* 
这个节点会返回一个Stat对象，如果给定的path不存在的话则返回null。这个方法有一个关键参数，可以提供一个Watcher对象。Wathcer是Zookeeper强大功能的源泉。Watcher就是一个事件处理器，一个回调。比如这个exists方法，调用后，如果别人对这个path上的节点进行操作，比如创建，删除或设置数据，这个Wather都会接收到对应的通知。

*setData/getData* 
设置或获取节点的数据，getData也可以设置Watcher

*getChildren* 
获取子节点，可以设置Watcher

*sync zookeeper*
是一个集群，创建节点的时候只要半数以上的节点确认就认为是创建成功了，但是如果读取的时候正好读取到一个落后的节点上，那就有可能读取到旧的数据，这个时候可以执行一个sync操作，这个操作可以确保读取到最新的数据。

*zookeeper的client* 
api基本上介绍完了。zookeeper强大的功能都是通过这些API来实现的，zookeeper通过一个简单的文件系统数据模型对外提供服务。通过临时节点，Watcher等手段我们可以实现一些在分布式环境种很难做到的事情。


