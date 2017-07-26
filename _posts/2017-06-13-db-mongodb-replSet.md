---
layout: post
title:  "mongodb副本集"
desc: "mongodb副本集"
keywords: "mongodb replSet,mongodb副本集,kinglyjn,张庆力"
date: 2017-06-13
categories: [DB]
tags: [DB, mongodb]
icon: fa-database
---



### 副本集的特点

* 主是唯一的，但不是固定的：具体哪个节点是主节点由仲裁决定，相对于传统的主从结构副本集可以进行自动容灾。默认读写操作指向主节点，主节点记录写操作的oplog（写日志），从节点根据oplog进行数据的复制，注意mongodb的从节点是绝对不可写的（即使是root权限也不行）
* 二分之一原则：集群存活节点小于等于1/2时，集群不可写，所有节点都降为从节点
* 复制集成员：包括数据节点、投票节点两种。数据节点存放数据（实体物理文件 \*.ns，\*.0等，包括主节点和从节点），投票节点不存放数据，仅作选举和充当复制集节点。



### 搭建mongodb副本集

```shell
#准备三台机器
172.16.127.129
172.16.127.130
172.16.127.131

#配置文件信息(这些配置参数在 xxx/mongod --help 中都可以找到，注意一组副本集的replSet必须相同)
replSet = rs01
bind_ip = 0.0.0.0
port = 27017
dbpath = /home/ubuntu/mongodb-3.4.4/data
logpath = /home/ubuntu/mongodb-3.4.4/logs/mongod.log
logappend = true
pidfilepath = home/ubuntu/mongodb-3.4.4/logs/mongod.pid
fork = true
auth = false
oplogSize = 1024


#分别在这三台机器上启动mongodb
mongodb-3.4.4/bin/mongod -f mongodb-3.4.4/conf/mongod.conf [--replSet rs01]
mongodb-3.4.4/bin/mongod -f mongodb-3.4.4/conf/mongod.conf [--replSet rs01]
mongodb-3.4.4/bin/mongod -f mongodb-3.4.4/conf/mongod.conf [--replSet rs01]


#启动以后我们发现副本集还没有初始化配置信息,接下来便是初始化副本集
#在三台机器上任意一台登陆mongodb（我这里在129上登陆）
#使用admin数据库
use admin
#输入如下内容:
#注：红色字体的repset要和上面参数 “--replSet rs01” 保持一致.
#131节点设置为arbiter节点(仲裁投票节点)，仲裁节点不参与副本数据的存储
#config.members[2] = {_id:2,host:"172.16.127.131:27017", arbiterOnly:true}
#除了_id、host、arbiterOnly外的其他参数(见注释)

config=
{_id:"rs01",members:[
...{_id:0,host:"172.16.127.129:27017"},
...{_id:1,host:"172.16.127.130:27017"},
...{_id:2,host:"172.16.127.131:27017"}]
}

#初始化副本集配置
#输出成功{"ok",1}
rs.initiate(config)
#config = rs.conf();
#更改config之后，可以重新初始化副本集
#副本集version在每次初始化副本集之后将会自增，必要的话也可以更改local.system.replset集合来更改version
#rs.reconfig(config)的瞬间会有短暂的断开连接，所以执行该操作需要避开写入高峰期
#rs.reconfig(config)

#查看日志，副本集启动成功后，130为主节点PRIMARY，129、131为副本节点SECONDARY。(选举机制似乎类似于zk)
#查看集群节点的状态
rs.status()
rs.isMaster()


#至此整个副本集就搭建成功了，接下来测试副本集数据复制功能
#回到130主节点连接进入终端
#建立test数据库
use test
#往user测试表中插入数据
db.user.insert({name:"zhangsan", age:23})
#在副节点129,131上查看数据是否复制过来
use test
show tables
"errmsg" : "not master and slaveOk=false",
#mongodb默认是从主节点读写数据的，副本节点上不允许读，需要设置副本节点可以读(主节点可读可写，从节点可读)
db.getMongo().setSlaveOk()
或者
rs.slaveOk(true)
#可以看到数据已经复制到了副本集
db.user.find()


#容灾测试
#此时如果主节点130宕机(关闭主节点)
mongodb-3.4.4/bin/mongod --shutdown -f mongodb-3.4.4/conf/mongod.conf
#查看129和131节点状态发现，此时129节点变成了主节点(可以show log rs查看日志)，向主节点写入数据ok
use test
db.user.insert({name:"lisi", age:24})
#查看131从节点，发现数据副本已经生成
db.user.find()


#在主库130中添加从库
rs.add("172.16.127.132:27017")
{ "ok" : 1 }


#手动将主节点降级为从节点(在30s内将当前节点降级为从节点)
rs.stepDown(30)
```

注释：

* `priority`: 取值范围为0~1000，默认为1，当设置为0时，该节点的永远不能被选为主节点
* `hidden`: true/false，当为true表示该节点为隐藏节点(对rs.isMaster()函数不可见)，`前提是priority为0`。前端应用通过 rs.isMaster 函数判断当前节点的主从性，当一个节点为hidden时也就意味着当前节点对应用不可见
* `votes`: 0/1，2.6版本之前副本集最多12个节点，最多7个投票节点，多余的5个节点votes必须为0
* `slaveDelay`: 单位为秒，slaveDelay=3600表示延时一个小时复制主库操作，常用于对误操作进行恢复。前提是`priority=0、hidden=true`
* `buildIndexes`: true/false，默认为true，表示当前节点是否复制主库的索引，`false前提priority为0`



### 副本集keyFile认证以及为副本集创建用户

 ```shell
# 副本集总体思路是用户名、密码和keyfile文件，keyfile需要各个副本
# 集服务启动时加载而且要是同一文件，然后在操作库是需要用户名、密码

# KeyFile文件必须满足条件:
（1）至少6个字符，小于1024字节
（2）认证时候不考虑文件中空白字符
（3）连接到副本集的成员和mongos进成的keyfile文件内容必须一样
（4）必须是base64编码,但是不能有等号
（5）文件权限必须是600

# 1. 生成KeyFile文件
openssl rand -base64 1024 > ~/mongodb-3.4.4/keyFile
chmod 600 ~/mongodb-3.4.4/keyFile

# 2. 配置文件需要增加如下配置(注意每个节点都要增加如下配置，并且每个节点的keyFile文件内容都是相同的)
shardsvr=true
replSet=rs01
auth = true
keyFile = /home/ubuntu/mongodb-3.4.4/keyFile

# 3. 分别在这三台机器上启动mongodb
mongodb-3.4.4/bin/mongod -f mongodb-3.4.4/conf/mongod.conf
mongodb-3.4.4/bin/mongod -f mongodb-3.4.4/conf/mongod.conf
mongodb-3.4.4/bin/mongod -f mongodb-3.4.4/conf/mongod.conf

# 4. 使用root账户连接mongodb主节点(以test库为例)，在test中操作增加testuser用户
#    在其他节点的admin.system.users表中也会保存新创建的testuser用户信息
use test
var user = {
	user:"testuser", 
 	pwd:"xxx",
 	customData:"xxx",
 	roles:[{role:"readWrite", db:"test"}]
}
db.createUser(user);

# 5. 退出客户端，在各个节点再次使用testuser用户连接test库
#    注意在从节点使用rs.slaveOk(true)命令获取从节点的读取权限
mongodb-3.4.4/bin/mongo 127.0.0.1:27017/test -u testuser -p xxx

# 6. 测试往主节点test.user表中插入数据，在副节点129,131上查看数据是否复制过来
db.user.insert({name:"zhangsan", age:23})
 ```

