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
dataLogDir=/home/ubuntu/zookeeper-3.4.6/logs
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
<img src="http://img.blog.csdn.net/20161213112207892?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:700px;">