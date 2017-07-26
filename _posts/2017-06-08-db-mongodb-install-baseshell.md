---
layout: post
title:  "mongodb安装、卸载和常见客户端命令"
desc: "mongodb安装、卸载和常见客户端命令"
keywords: "mongodb安装卸载,kinglyjn,张庆力"
date: 2017-06-08
categories: [db]
tags: [db, mongodb]
icon: fa-database
---



### mongodb的标准安装与卸载

```shell
# 1. 导入包管理系统使用的公钥
# Ubuntu 的软件包管理工具（即dpkg和APT）要求软件包的发布者通过GPG密钥
# 签名来确保软件包的一致性和真实性。通过以下命令导入MongoDB公共GPG密钥：
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6


# 2. 创建list file
# 根据 Ubuntu 的版本使用适当的命令创建 list file: /etc/apt/sources.list.d/mongodb-org-3.4.list
# ubuntu14.04
echo "deb [ arch=amd64 ] http://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list
# ubuntu16.04
echo "deb [ arch=amd64,arm64 ] http://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list


# 3. 重新下载本地包数据库索引
sudo apt-get update


# 4. 安装 MongoDB
sudo apt-get install -y mongodb-org


# 5. 运行MongoDB
sudo service mongod start
# 或者
sudo systemctl start mongod

# 6. 命令行运行命令：
mongo

# 7. 卸载mongodb
# 关闭MongoDB
sudo service mongod stop
# 删除所有相关软件包
sudo apt-get purge mongodb-org*
# 删除数据和日志目录
sudo rm -r /var/log/mongodb
sudo rm -r /var/lib/mongodb
```



### mongodb直接解压安装

```shell
# 下载解压软件包
wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-ubuntu1404-3.4.4.tgz
tar -zxvf mongodb-linux-x86_64-ubuntu1404-3.4.4.tgz -C .


# 创建数据库数据、配置文件、日志文件的存放目录
mkdir -p data conf logs


# 创建和编写mongodb配置文件
touch mongod.conf
vim mongod.conf
-------------------
#指定mongod端口
port = 27017
#指定数据存放目录
dbpath = /home/ubuntu/mongodb-3.4.4/data
#指定日志文件
logpath = /home/ubuntu/mongodb-3.4.4/logs/mongod.log
#指定启动的是后台进程，这个参数在windows下无效
fork = true
#是都启用权限认证
auth = false
-------------------


# 启动mongod进程，-f等价于--config
./bin/mongod -f conf/mongod.conf
# 加入开机启动项
echo "xxx/mongod -f xxx/mongodb.conf;" >> /etc/rc.local


#关闭命令，前提是关闭了前端程序
xxx/mongod --shutdown -f xxx/mongodb.conf


#启动客户端交互
#在~/.mongorc.js中可以编写mongod启动时运行的js脚本，如这里打印的 "hello kinglyjn!"
xxx/mongo localhost:27017/test
------------------------------
MongoDB shell version v3.4.4
connecting to: mongodb://localhost:27017/test
MongoDB server version: 3.4.4
hello kinglyjn!
Server has startup warnings: 
2017-06-08T18:38:52.632+0800 I CONTROL  [initandlisten] 
2017-06-08T18:38:52.632+0800 I CONTROL  [initandlisten] ** WARNING: /sys/kernel/mm/transparent_hugepage/enabled is 'always'.
2017-06-08T18:38:52.632+0800 I CONTROL  [initandlisten] ** We suggest setting it to 'never'
------------------------------

#我们发现有警告，按照提示修改即可，脚本如下(在root用户下执行 sudo -i)：
if test -f /sys/kernel/mm/transparent_hugepage/enabled; then
   echo never > /sys/kernel/mm/transparent_hugepage/enabled
fi
if test -f /sys/kernel/mm/transparent_hugepage/defrag; then
   echo never > /sys/kernel/mm/transparent_hugepage/defrag
fi
```



### 常见客户端命令

```shell
#数据库和集合
show dbs
use dbname
db.getName()
db.stats()
db.getMongo()
show collections
db.dropDatabase()
db.cname.drop()
db.copyDatabase("src_dbname", "target_dbname", "127.0.0.1")
db.repairDatabase()


#数据库状态分析
mongostat -h 127.0.0.1:27017
# 0：profile操作记录功能关闭  
# 1：记录操作值大于slowms的操作  
# 2：记录任何操作记录
# 一般测试阶段使用2，而生产环境profilingLevel的值为0
db.getProfilingLevel()    
db.setProfilingLevel(2) 
db.getProfilingStatus() { "was" : 0, "slowms" : 100 }
# 通过mongodb的配置文件/etc/mongo.conf的 verbose参数可以配置日志的详细程序（一个v~5个v）


#配置文件开启权限模式：
--------------------------
security:
	authorization: enabled

或者
auth = true
--------------------------
#在相应的库下创建和删除用户：
#数据库管理角色
#read、readWrite、dbAdmin、dbOwner、userAdmin
read：该角色只有读取数据的权限
readWrite：该角色有数据的读写权限，但不能创建删除索引，也不能创建和删除数据库
dbAdmin：该角色拥有数据库管理权限（比如更改数据库类型）
dbOwner：是read、readWrite、dbAdmin这三者角色的集合体
userAdmin：该角色可以对其他角色权限进行管理
#集群管理角色
clusterAdmin：提供了最大的集群管理功能。相当于clusterManager, clusterMonitor, 
			  and hostManager和dropDatabase的权限组合
clusterManager：提供了集群和复制集管理和监控操作。拥有该权限的用户可以操作config
			  和local数据库（即分片和复制功能）
clusterMonitor：仅仅监控集群和复制集
hostManager：提供了监控和管理服务器的权限，包括shutdown节点，logrotate, repairDatabase等。
#所有数据库角色
readAnyDatabase：具有read每一个数据库权限。但是不包括应用到集群中的数据库。
readWriteAnyDatabase：具有readWrite每一个数据库权限。但是不包括应用到集群中的数据库。
userAdminAnyDatabase：具有userAdmin每一个数据库权限，但是不包括应用到集群中的数据库。
dbAdminAnyDatabase：提供了dbAdmin每一个数据库权限，但是不包括应用到集群中的数据库。
#超级管理员权限
root: dbadmin到admin数据库、useradmin到admin数据库以及UserAdminAnyDatabase。
	  但它不具有备份恢复、直接操作system.*集合的权限，但是拥有root权限的超级用户
	  可以自己给自己赋予这些权限。

#backup、restore：备份角色权限
#客户端创建用户：(单库读写用户需要在各个数据库创建，admin管理用户需要在admin库创建)
var user = {
	user:"usernamexx", 
 	pwd:"passwordxx",
 	customData:"xxxx",
 	roles:[{role:"userAdmin", db:"admin"}, {role:"read", db:"response"}]
}
db.createUser(user); 
db.dropUser("usernamexxx");
#使用新创建的用户登录：
mongo 127.0.0.1:27017/test -u usernamexxx -p passwordxxx
或 db.auth('usernamexxx','passwordxxx')


#添加数据
#insert添加时如果主键重复则报错，而save添加时主键重复则覆盖
db.cname.insert({name:"zhangsan", age:23}) 
db.cname.save({name:"zhangsan", age:23})


#修改数据
#定位语句，更新语句，不存在是否更新插入，是否全部更新
db.cname.update(criteria, objNew, isUpsert, isMulti)
#局部修改
db.cname.update({age:25}, {$set:{name:"lisi"}}, false, false)
db.cname.update({age:25}, {$inc:{age:2}}, false, true)
db.cname.update({age:25}, {$push:{scores:{$each:[1,2,3,4]}}})
db.cname.update({age:25}, {$addToSet:{tags:{$each:["aaa", "bbb"]}}})
#整体修改
db.cname.update({age:25}, new_user)
#修改集合名称
db.cname.renameCollection("target_name")

#删除数据
db.cname.remove({name:"zhangsan"})


#查询数据
db.cname.count()
db.cname.dataSize()
db.cname.totalSize()
db.cname.storageSize()
db.cname.getShardVersion()
db.cname.stats()
db.cname.distinct("name") #select distict name from user
db.cname.find()
db.cname.find({name:"zhangsan"})
db.cname.find({name:"zhangsan"}).pretty()  #友好显示数据
db.cname.find({name:"zhangsan"}).explain() #性能分析函数
db.cname.find({name:"zhangsan", age:23})
db.cname.find({age:{$gt:22}}, {name:1, age:1}) #显示name和age字段
db.cname.find({age:{$gte:22}})
db.cname.find({age:{$gt:23, $lt:26}}) 		#查询age >= 23 并且 age <= 26 
db.cname.find({name:/mongo/}) 				#查询name中包含 mongo的数据
db.cname.find({"data.map.key1":/regexxxx/}) #注意这里必须加引号
db.cname.find().skip(3).limit(5).sort({age:-1})
db.cname.find().skip(3).limit(5).sort({$natural:-1}) #$natural一般按写入顺序排序
db.cname.find({$or:[{age:22}, {age:25}]})
db.cname.find({age:{$exists:true}}) 		#查询存在age字段的记录
db.cname.find({age:{$ne:23}}) 				#不等于
db.cname.find({age:{$in:[23,24,25]}})
db.cname.find({name:{$in:[null], $exists:true}}) #可以匹配 {name:null,sex:1,age:18}
db.cname.find({name:{$nin:["zhangsan"]}})
db.cname.find({x:{$all:[1,5]}})				#查询x数组中全部包含1和5的记录
db.cname.find({x:[1,2,3,4]})				#精确查询为该数组的记录
db.cname.find({"x.0":1})					#匹配数组中指定下标元素的值
db.cname.find({x:{$size:4}})				#匹配x数组长度为4的记录


#分组查询
db.user.group({
	key:{name:true},
	initial:{users:[]}, #每组都分享的“初始化函数”，可以在此处初始化一些变量供每组进行使用
	$reduce:function(current, previor){previor.users.push(current.name);},
	condition:{name:{$exists:true}}, #过滤条件
	finalize:function(out){out.count=out.users.length}
})


#索引数据
#_id索引
#单建索引
#多键索引
#复合索引
#过期索引
#全文索引
#地理信息索引
#哈希索引
db.cname.getIndexes()
db.cname.ensureIndex({name:1, age:-1})

#删除索引
db.cname.dropIndex("name_1_age_-1")
db.cname.dropIndexes()

#创建唯一索引(以下创建的索引只允许集合中存在唯一一个name-address记录)
db.cname.ensureIndex({name:1, adddress:1}, {unique:true})

#创建稀疏索引(不存在name的记录不创建name索引，注意不能使用稀疏索引查找不存在name字段的记录)
db.cname.ensureIndex({name:1}, {spare:true})

#加过期索引的字段必须为ISODate或者ISODate的数组，超时后记录被自动删除（TTL）
#删除的过程是不精确的，删除过程是每60s跑一次，而且删除也需要一定的时间，所以存在误差
db.cname.ensureIndex({createTime:1}, {expireAfterSeconds:50}) 
db.cname.ensureIndex({name:1, age:-1}, {name:"name_age_idx", unique:true, sparse:true, expireAfterSeconds:50})
#强制使用索引查找
db.cname.find({name:"zs", age:23}).hint("name_1_age_-1"); 

#全文检索索引
db.cname.ensureIndex({name:"text"}) 
db.cname.ensureIndex({"$**":"text"})
#全文检索查询
db.cname.find({$text:{$search:"aaa"}}) 
db.cname.find({$text:{$search:"aa bb cc"}})  	 #查询包含aa或bb或cc的记录
db.cname.find({$search:"\"aa\" \"bb\" \"cc\""}}) #查询既包含aa又包含bb又包含cc的记录
db.cname.find({$text:{$search:"aa bb -cc"}}) 	 #查询包含aa或bb，但是不包含cc的记录
db.cname.find({$text:{$search:"/AA/i"}}) 	 	 #正则匹配
db.cname.find({$text:{$search:"aaa"}, score:{$meta:"textScore"}}) 
db.cname.find({$text:{$search:"aaa"}}, {score:{$meta:"textScore"}}).sort({score:{$meta:"textScore"}}) #根据匹配度排序

#地理位置索引
#平面地理位置索引 经纬度范围 -180~180 -90~90
db.cname.ensureIndex({p:"2d"}) 
#球面地理位置索引
db.cname.ensureIndex({p:"2dsphere"}) 
db.cname.insert({p:[0,0]})
db.cname.insert({p:[-180,-90]})
db.cname.insert({p:[180,90]})
#默认返回100个最近的点
db.cname.find({p:{$near:[0,0]}}) 
db.cname.find({p:{$near:[0,0]}, $maxDistance:10}) 
#查找圆内的点
db.cname.find({p:{$geoWithin:{$center:[[0,0], 5]}}}) 
#查找方块内的点
db.cname.find({p:{$geoWithin:{$box:[[0,0], [3,3]]}}}) 
#查找多边形内的点
db.cname.find({p:{$polygon:[[0,0],[1,3],[4,5]]}})
```