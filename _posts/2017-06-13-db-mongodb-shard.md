### mongodb分片技技术

> 注意：MongoDB 3.4, config servers must be deployed as a replica set (CSRS).
>
> 也就是说mongodb3.4以后【config节点】强制使用副本集。



**什么是分片？**
分片是将数据进行拆分，将数据水平地分散到不同服务器的技术。

<img src="http://img.blog.csdn.net/20170609135819836?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:70%">



**为什么要分片？**

架构上：读写均衡，去中心化。

硬件上：解决单机内存和硬盘的限制。

总的来说，分片能够改善单台机器数据的存储以及数据吞吐性能，提高在大量数据下随机访问的性能。



**分片和副本集的比较**

|       | replication（副本集）         | shard（分片） |
| ----- | ------------------------ | --------- |
| 实现意义  | 数据冗余、实现读写分离（主写副读），提升读的性能 | 提升并发性能    |
| 架构上   | 中心化                      | 水平化       |
| 实现原理上 | 数据镜像                     | 数据打散分布    |
| 维护成本  | 相对容易                     | 相对较高      |



**分片节点介绍**

| 节点       | 说明                               | 启动命令                                   |
| :------- | :------------------------------- | -------------------------------------- |
| mongos节点 | 接收前端请求，进行对应消息的路由                 | mongos --configdb configdbserver:27017 |
| config节点 | 存储元数据，为路由节点mongos服务，将数据路由到shared | mongod --configsvr --rpelSet 副本集       |
| shard节点  | 存储数据的节点（单个mongod或副本集）            | mongod --shardsvr                      |



结构如下：

```default
app—>mongos---->CONFIG_RS[config]---->RS01[shard01、shard02、shard03...]
					             ---->RS02[shard01、shard02、shard03...]
					             ---->RS03[shard01、shard02、shard03...]				    
```



**分片测试细节**

分片节点信息：

```shell
# 分片节点信息：
172.16.127.128		mongos
172.16.127.129		[config_rs]configsvr0,configsvr1,configsvr2
172.16.127.130		shardsvr01
172.16.127.131		shardsvr02


# 配置信息
----------------------------
#shardsvr配置文件mongod.conf
port = 27017
dbpath = /home/ubuntu/mongodb-3.4.4/data
logpath = /home/ubuntu/mongodb-3.4.4/logs/mongod.log
logappend = true
fork = true
auth = false

#config_rs配置文件mongod.conf
port = 27017
dbpath = /home/ubuntu/mongodb-3.4.4/data
logpath = /home/ubuntu/mongodb-3.4.4/logs/mongod.log
logappend = true
fork = true
auth = false

port = 27018
dbpath = /home/ubuntu/mongodb-3.4.4-01/data
logpath = /home/ubuntu/mongodb-3.4.4-01/logs/mongod.log
logappend = true
fork = true
auth = false

port = 27019
dbpath = /home/ubuntu/mongodb-3.4.4-02/data
logpath = /home/ubuntu/mongodb-3.4.4-02/logs/mongod.log
logappend = true
fork = true
auth = false

#mongos配置文件mongod.conf
port = 27017
logpath = /home/ubuntu/mongodb-3.4.4/logs/mongod.log
logappend = true
fork = true
```

启动各个分片节点

```shell
# shardsvr01 & shardsvr02
mongodb-3.4.4/bin/mongod -f mongodb-3.4.4/conf/mongod.conf --shardsvr
mongodb-3.4.4/bin/mongod -f mongodb-3.4.4/conf/mongod.conf --shardsvr

# configsvr(正常应该配置至少三个configsvr)
mongodb-3.4.4/bin/mongod -f mongodb-3.4.4/conf/mongod.conf --configsvr --replSet config_rs
mongodb-3.4.4-01/bin/mongod -f xxx/conf/mongod.conf --configsvr --replSet config_rs
mongodb-3.4.4-02/bin/mongod -f xxx/conf/mongod.conf --configsvr --replSet config_rs
# 初始化CONFIG_RS
use admin
var config = {_id:"config_rs", configsvr:true,members:[{_id:0, host:"172.16.127.129:27017"}]}
rs.initiate(config)
rs.add({_id:1, host:"172.16.127.129:27018"})
rs.add({_id:2, host:"172.16.127.129:27019"})

# mongos
mongodb-3.4.4/bin/mongos -f mongodb-3.4.4/conf/mongod.conf --configdb config_rs/172.16.127.129:27017,172.16.127.129:27018,172.16.127.129:27019
```

给具体的库表添加分片：

```shell
#1. 连接到mongos
	mongodb-3.4.4/bin/mongo
	use admin
	
#2. add shard [需要在admin数据库下操作]
	#注意如果你的mongos和shard在同一台机器上，添加分片不能使用localhost，建议使用ip
	#对于单个数据库实例,maxSize表示分片节点最大存储大小
	#db.runCommand({addShard:"<hostname><:port>", maxSize:<size>, name:"<shard_name>"})
	#对于副本集群
	#db.runCommand({addShard:"<replica_set>/<hostname><:port>", 
	#				maxSize:<size>, name:"<shard_name>"})
	db.runCommand({addShard:"172.16.127.130:27017"})
	db.runCommand({addShard:"172.16.127.131:27017"})
	#或者
	sh.addShard('172.16.127.130:27017')
	sh.addShard('172.16.127.131:27017')
	sh.status()
	sh.status({verbose:true})
	
#3. enable sharding [admin，这里只是标识这个数据库可以启用分片，但实际上并没有进行分片]
	db.runCommand({enableSharding:"<dbname>"})
	#或者(启用test库的分片)
	sh.enableSharding('test')

#4. 对一个集合进行分片(key为片件，指定为片件的字段需要索引，punique表示对shard-key的唯一性的约束)
	db.user.ensureIndex({name:1})
	db.runCommand({shardcollection:"dbname.cname", key:{"userid":1}, unique:<true/false>})
	#或者
	db.user.ensureIndex({name:1})
    sh.shardCollection("test.user", {name:1})

#5. 如果要加验证，官网说只要在mongos上操作即可，简单地添加一个用户：
    use test
    db.createUser({
        user:'test',
        pwd:'123456',
        roles:[
            {role:'readWrite',db:'test'}
        ]
    })

#分片测试
#1.查看集合状态
db.shard_cname.stats()
#2.查看分片状态
db.printShardingStatus()
#3.在mongos节点插入数据
#for(var i=0; i<1000; i++) db.user.insert({name:String.fromCharCode(i%26+97), age:i})
for (var i=0; i<1000; i++) db.user.insert({name:"zhangsan"+i, age:i})
for (var i=0; i<1000; i++) db.user.insert({name:"lisi"+i, age:i})
#4.查看shard01和shard02两个节点，可以看到数据已经均摊到这两个节点上了(mongos节点上是不保存插入数据的)
db.user.find()
```

删除分片

```shell
#删除分片(在mongos节点操作)
use admin
#下面这句会立即返回，实际在后台执行
db.runCommand({removeshard:"172.16.127.130:27017"})
#我们可以反复执行上面语句，查看执行结果
db.runCommand({removeshard:"172.16.127.130:27017"})
{ msg: "draining ongoing" , state: "ongoing", remaining: { chunks: 42, dbs : 1 }, ok: 1 }
#从上面可以看到，正在迁移，还剩下42块没迁移完，当remain为0之后，这一步就结束了，因为我们有两个分片节点，shard01从集群被移除之后，shard01（1002条）的分片数据全部被迁往到了shard02(1001+1002条)，但是shard01物理节点上的数据背不会被删除

#有一些分片上保存上一些unsharded data，需要迁移到其他分片上，
#可以用sh.status()查看分片上是否有unsharded data，如果有则显示：
{  "_id" : "test",  "primary" : "shard0000",  "partitioned" : true }

#用下面的命令迁移(只有全部迁移完上面的命令才会返回，执行完毕后shard01的未分片的1002条数据也被删除了)
mongos> db.runCommand({movePrimary:"test", to:"shard0001"})
shards:
        {"_id":"shard0000", "host":"172.16.127.130:27017", "state":1, "draining": true }
        {"_id":"shard0001", "host":"172.16.127.131:27017", "state":1 }
databases:
		{ "primary" : "shard0001:172.16.127.131:27017", "ok" : 1 }

#最后运行命令进行彻底删除
db.runCommand({removeshard:"172.16.127.130:27017"})
sh.status()
shards:
        {"_id":"shard0001", "host":"172.16.127.131:27017", "state":1}
databases:
		{"_id":"test", "primary":"shard0001", "partitioned":true }
```

添加被删除的分片

```shell
#重新添加刚才删除的分片，将遇到以下错误，提示test数据库已经存在
db.runCommand({addShard:"172.16.127.130:27017"})
{
  "code" : 96,
  "ok" : 0,
  "errmsg" : "can't add shard '172.16.127.130:27017' 
  "because a local database 'test' exists in another shard0001"
}
#这时，就需要通过MongoDB shell连接到shard01实例，删除test数据库，然后重新添加
use test
db.dropDatabase()
#回到mongos节点
use admin
mongos> sh.addShard('172.16.127.130:27017')
{
        "code" : 11000,
        "ok" : 0,
        "errmsg" : "E11000 duplicate key error collection: 
        	"admin.system.version index: _id_ dup key: { : \"shardIdentity\" }"
}
#这时候再连接到shard01节点，删除admin.system.version集合中记录
db.system.version.remove({})
db.system.version.remove({"_id":"shardIdentity"})
WriteResult({
        "nRemoved" : 0,
        "writeError" : {
                "code" : 40070,
                "errmsg" : "cannot delete shardIdentity document while in --shardsvr mode"
        }
})
#删除时报错，意思是说不能在分片模式下删除这张表中的这条记录，然后我们关闭shard01，然后以非shardsvr的方式启动，删除这条记录后，再以shardsvr方式启动
db.shutdownServer()
mongodb-3.4.4/bin/mongod -f mongodb-3.4.4/conf/mongod.conf 
use admin
db.system.version.remove({})
db.shutdownServer()
mongodb-3.4.4/bin/mongod -f mongodb-3.4.4/conf/mongod.conf --shardsvr
#最后添加之前被删除的分片shard01，大功告成
db.runCommand({addShard:"172.16.127.130:27017"})
{ "shardAdded" : "shard0002", "ok" : 1 }
```

哈希分片（hash key）

分片过程中利用哈希索引作为分片的单个键。

- 哈希分片的片件只能使用一个字段
- 哈希片件的最大好处就是能够保证数据在各个分片节点分布基本均匀

```shell
#连接mongos，在原来test分片库的基础上新建userid_hash集合，并插入数据
db.userid_hash.insert({userid:11})
db.userid_hash.insert({userid:22})

#hash分片之前需要为相关字段建立hash索引(索引名称默认为"userid_hashed"形式)
db.userid_hash.ensureIndex({userid:"hashed"})

#切换到admin库，对test.userid_hash进行hash分片
use admin
sh.shardCollection("test.userid_hash", {userid:"hashed"})

#测试分片
use test
for (var i=0; i<1000; i++) db.userid_hash.insert({userid:i})
```

关于两个概念的说明：

```shell
片件（shardid）：
当设置分片时，需要从集合里面选择一个或几个键，把选择出来的键作为数据拆分的依据，这个键叫做片键或复合片键。选好片键后，MongoDB将不允许插入没有片键的文档，但是允许不同文档的片键类型不一样。片件的好坏跟大程度上影响集群的性能、容量和功能。
>如果片件的重复性很强，就会导致数据块不容易拆分，容易造成大的数据块，导致数据分布不均匀。
>如果片件是单调递增的_id或时间戳，这样会导致你一直往最后一个副本集中添加数据，比较合适的片件一般是复合片件，形式如{node:1,time:1}

块（chunk）：
在一个shard server内部，MongoDB还是会把数据分为chunks，每个chunk（默认64m）代表这个shard server内部一部分数据。chunk的产生，会有以下两个用途：
	- Splitting：当一个chunk的大小超过配置中的chunk size时，MongDB的后台
				进程会把这个chunk切分成更小的chunk，从而避免chunk过大的情况
	- Balancing：在MongoDB中，balancer是一个后台进程，负责chunk的迁移，
				从而均衡各个shard server的负载
```

数据块的拆分（split）和均衡(balance)：

<img src="http://img.blog.csdn.net/20170613104216695?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:100%"/> 

<img src="http://img.blog.csdn.net/20170613104311181?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:100%"/>

**手动分片**

目的：为了减少自动平衡过程中带来的io等资源的消耗。

前提：关闭auto balance，充分了解数据并对数据进行预先划分。

```shell
#步骤一，关闭自动平衡(开启使用sh.startBalancer)，此时平衡器状态为 Currently enabled:no
sh.stopBalancer()

#步骤二，预分配空chunk，这是一种提高写效率的方法。
#相当于在写入真实数据之前，就分配好了数据桶，然后再对号入座。省去了创建chunk和split的时间。
#格式 db.adminCommand({split:<database>.<collection>, <find|middle|bounds>})
#find 表示查找到的记录进行分裂
#bounds是指定[low, up]分裂
#middle是指定分裂的点
#一个预分配chunk的例子如下
#这个预分配的目的是字母顺序有一定间隔的email， 分配到不同的chunk里。
#例如aa-ag到一个chunk，ag-am到一个chunk
use admin
sh.enableSharding('test')
db.user_email.ensureIndex({email:1})
sh.shardCollection("test.user_email", {email:1})
for (var i=97; i<97+26; i++) {
  for (var j=97;j<97+26; j+=6) {
    var prefix = String.fromCharCode(i)+String.fromCharCode(j);
    sh.splitAt("test.user_email", {email:prefix});
  }
}

#步骤三，定义shServer，手动移动分割快（moveChunk）
#db.adminCommand({moveChunk:"test.user_email",
#	find:{email:prefix}, to:shServer[(j-97)%2]})
var shServers = ["172.16.127.130:27017", "172.16.127.130:27017"];
for (var i=97; i<97+26; i++) {
  for (var j=97;j<97+26; j+=2) {
    var prefix = String.fromCharCode(i)+String.fromCharCode(j);
    sh.moveChunk("test.user_email", {email:prefix}, shServers[(j-97)%2]);
  }
}
```