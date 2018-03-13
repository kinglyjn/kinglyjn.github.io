---
layout: post
title:  "CDH5 的安装和配置"
desc: "CDH5 的安装和配置"
keywords: "CDH5 的安装和配置,cdh5,kinglyjn"
date: 2018-01-25
categories: [Java]
tags: [java]
icon: fa-coffee
---



### ubuntu14.04 or centos7 CDH5 离线安装

1、机器准备

```shell
cdh01  192.168.56.106
cdh02  192.168.56.101
cdh03  192.168.56.102

[注] 每个节点已安装JDK (安装位置/usr/java/目录下,cdh规定位置)
```

<br>



2、创建普通账号，设置sudo无密码权限

```shell
# 创建hadoop用户
$ sudo adduser hadoop
$ sudo passwd xxx

# 给hadoop用户s和盒子无密码sudo权限
$ sudo chmod u+w /etc/sudoers
$ sudo vim /etc/sudoers
	[编辑增加]
	hadoop ALL=(root) NOPASSWD:ALL
$ sudo chmod u-w /etc/sudoers

# 测试是否成功
$ sudo ifconfig
```

<br>



3、配置无密码SSH访问，关闭selinux，开机禁用大页面，swappiness设置为最大值10

```shell
# A->B
# 在A上生成公私秘钥对
$ ssh-keygen -t 'rsa'

# 在A上将公钥发送给B
$ ssh-copy-id hadoop@cdh02


# 关闭selinux，重启机器生效 (每个节点)
$ sudo vim /etc/selinux/config
[编辑文件，将SELINUX改为disabled]
SELINUX=disabled


# 开机禁用大页面 (每个节点)
$ sudo vim /etc/rc.local
  [编辑增加]
  echo never > /sys/kernel/mm/transparent_hugepage/defrag
  echo never > /sys/kernel/mm/transparent_hugepage/enabled
$ sudo chmod +x /etc/rc.local


# swappiness设置为最大值10 (每个节点)
$ sudo vim /etc/sysctl.conf
  [编辑增加]
  vm.swappiness=10
```

<br>



4、修改主机名称、hosts映射，设置静态ip，禁用ipv6，禁用防火墙

```shell
# 修改主机名称
$ sudo vim /etc/hostname


# 设置hosts映射
$ sudo vim /etc/hosts
127.0.0.1  localhost
192.168.56.106 cdh01
192.168.56.101 cdh02
192.168.56.102 cdh03


# 设置静态ip [ubuntu]
$ sudo vim /etc/network/interfaces
  [编辑增加]
  auto eth0
  iface eth0 inet static
  address 192.168.56.101
  netmask 255.255.255.0
  gateway 192.168.56.1
  dns-nameservers 192.168.56.1
  
  [重启网络服务]
  sudo service networking restart

# 设置静态ip [centos]
$ sudo vim /etc/sysconfig/network-scripts/ifcfg-eth0
  [编辑增加]
  BOOTPROTO=static
  IPADDR=192.168.56.101
  ONBOOT=yes
  GATEWAY=192.168.56.1
  DNS1=8.8.8.8
  DNS2=114.114.114.114
 
  [重启网络服务]
  sudo service network restart
  
  
# 禁用ipv6(适用于ubuntu、centos等环境)
$ sudo vim /etc/sysctl.conf
  [编辑增加]
  [如果是 OpenVZ 环境，请把最后一条的 eth0 改成 venet0 ，
  同理如果网卡是 eth1 或者其他的名字，也要相应修改。]
  net.ipv6.conf.all.disable_ipv6 = 1
  net.ipv6.conf.default.disable_ipv6 = 1
  net.ipv6.conf.lo.disable_ipv6 = 1
  net.ipv6.conf.eth0.disable_ipv6 = 1
$ sudo sysctl -p 
$ ifconfig


# 禁用防火墙
$ sudo apt-get install ufw
$ sudo ufw disable
$ sudo ufw status
```

<br>



5、安装上传和下载工具、安装ntp时间同步工具

```shell
# 安装rz和sz工具 [ubuntu]
$ sudo apt-get install lrzsz


# 安装时间同步工具(每个节点都需要安装)
$ sudo apt-get install ntp

#时间被同步节点ntp配置(cdh01)
$ sudo vim /etc/ntp.conf
server 127.127.1.0
fudge 127.127.1.0 stratum 10
restrict 192.168.56.0 mask 255.255.255.0 nomodify notrap

# 时间同步节点ntp配置(cdh01、cdh02)
$ sudo vim /etc/ntp.conf
server cdh01 prefer
server 127.127.1.0
fudge 127.127.1.0 stratum 10
restrict 192.168.56.0 mask 255.255.255.0 nomodify notrap

# 重启所有节点的ntp服务，并设置ntp自动开机重启
$ sudo service ntp restart
$ sudo sysv-rc-conf   [$ sudo apt-get install sysv-rc-conf]

# 验证
$ ntpq -c peers
```

<br>



6、安装文件的准备

```shell
# ubuntu14.04(trusty)对应的 cdh parcel、parcel.sha1、manifest.json
$ wget http://archive.cloudera.com/cdh5/parcels/5.11/CDH-5.11.2-1.cdh5.11.2.p0.4-trusty.parcel
$ wget http://archive.cloudera.com/cdh5/parcels/5.11/CDH-5.11.2-1.cdh5.11.2.p0.4-trusty.parcel.sha1
$ wget http://archive.cloudera.com/cdh5/parcels/5.11/manifest.json

# ubuntu14.04(trusty)对应的 cm
$ wget http://archive.cloudera.com/cm5/cm/5/cloudera-manager-trusty-cm5.11.2_amd64.tar.gz


#[注]
#必要的安装包
[ubuntu 在各个节点安装]
$ sudo apt-get install lsb-base psmisc bash libsasl2-modules libsasl2-modules-gssapi-mit zlib1g libxslt1.1 libsqlite3-0 libfuse2 fuse rpcbind
# 防止 hue 的mysql数据库连接测试错误
$ sudo apt-get install python-lxml python-mysql.connector python-mysqldb

[centos 在安装节点安装]
$ yum install psmisc krb5-devel cyrus-sasl-gssapi cyrus-sasl-devel
$ yum install libxml2-devel libxslt-devel mysql mysql-devel  
$ yum install openldap-devel python-devel sqlite-devel 
$ yum install python-setuptools gcc gcc-c++ make
$ yum install perl
$ easy_install simplejson
```

<br>



7、安装mysql 或 MariaDB （本文用mysql5.6），并创建各组件对应的数据库（官方强烈建议安装在cm节点上）

```sql
> create database hive default charset latin1;
> create database amon default charset utf8;
> create database oozie default charset utf8;
> create database hue default charset utf8;

> grant all privileges on *.* to 'root'@'%' identified by 'xxxxxx' with grant option;
> flush privileges;

$ sudo vim /etc/mysql/my.cnf
  [编辑配置 bind-address改为0.0.0.0]
  bind-address  0.0.0.0
$ sudo service mysql restart
```

<br>



8、安装cloudera-manager (shell操作部分)

```shell
# 1.将cloudera-manager解压至/opt目录下，并将其属组、属主改为root:root
$ sudo tar -zxvf xxx/cloudera-manager-trusty-cm5.11.2_amd64.tar.gz -C /opt
$ sudo chown -R root:root cm-5.11.2

# 2.配置mysql-connector
$ sudo cp /opt/cm-5.11.2/share/cmf/common_jars/mysql-connector-java-5.1.15.jar /opt/cm-5.11.2/share/cmf/lib/
$ sudo mkdir /usr/share/java
$ sudo cp /opt/cm-5.11.2/share/cmf/common_jars/mysql-connector-java-5.1.15.jar /usr/share/java


# 3.先重启cm节点，及数据库节点
# 先在mysql节点操作
mysql> select distinct concat('User: ''',user,'''@''',host,''';') as query from mysql.user;
+---------------------------------------+
| query                                 |
+---------------------------------------+
| User: 'root'@'%';                     |
| User: 'sonar'@'%';                    |
| User: 'root'@'127.0.0.1';             |
| User: 'root'@'::1';                   |
| User: 'scm'@'cdh01';                  |
| User: 'root'@'dbserver';              |
| User: 'debian-sys-maint'@'localhost'; |
| User: 'root'@'localhost';             |
| User: 'sonar'@'localhost';            |
+---------------------------------------+

mysql> select password('scm');
+-------------------------------------------+
| password('scm')                           |
+-------------------------------------------+
| *45E6E3C68BDF1AC7EBB5C5A3BCBD5E9437B293BE |
+-------------------------------------------+

mysql> GRANT ALL PRIVILEGES ON *.* TO 'scm'@'%' IDENTIFIED BY PASSWORD '*45E6E3C68BDF1AC7EBB5C5A3BCBD5E9437B293BE' WITH GRANT OPTION;


# 4.然后在cm节点初始化数据库参数 (mysql的cm库事先不能存在)
$ sudo /opt/cm-5.11.2/share/cmf/schema/scm_prepare_database.sh mysql cm -hdbserver -uroot -p23wesdxc --scm-host cdh01 scm scm scm


# 5.创建agent目录
$ sudo mkdir /opt/cm-5.11.2/run/cloudera-scm-agent

# 6.修改config.ini文件
$ sudo vim /opt/cm-5.11.2/etc/cloudera-scm-agent/config.ini
  [编辑修改 config.ini文件中server_host]
  server_host=cdh01
  
# 7.准备parcels安装包
# 在cm节点，创建 /opt/cloudera/parcel-repo 目录，并将以下三个包复制至该目录(需创建该目录)中
# 最后将 *.sha1文件重命名为*.sha，否则系统会重新下载*.sha文件。
$ sudo mkdir -p /opt/cloudera/parcel-repo
$ sudo cp /opt/software/CDH-5.11.2-1.cdh5.11.2.p0.4-trusty.parcel /opt/cloudera/parcel-repo/
$ sudo cp /opt/software/CDH-5.11.2-1.cdh5.11.2.p0.4-trusty.parcel.sha1 /opt/cloudera/parcel-repo/CDH-5.11.2-1.cdh5.11.2.p0.4-trusty.parcel.sha
$ sudo cp /opt/software/manifest.json /opt/cloudera/parcel-repo/

# 8.将cm节点下的相应目录及文件传至其他节点的相同目录
$ scp -r cloudera/ cm-5.11.2/ root@cdh02:/opt/
$ scp -r cloudera/ cm-5.11.2/ root@cdh03:/opt/

# 9.在每个节点创建cloudera-scm用户
$ sudo useradd --system --home-dir=/opt/cm-5.11.2/run/cloudera-scm-server/ --no-create-home --shell=/bin/false --comment "Cloudera SCM User" cloudera-scm

# 10.启动
# 在cm节点启动server
$ sudo /opt/cm-5.11.2/etc/init.d/cloudera-scm-server start
# 在agent节点启动agent
$ sudo /opt/cm-5.11.2/etc/init.d/cloudera-scm-agent start
```

<br>



9、安装cloudera-manager (cm页面操作部分)

```shell
# 1.通过浏览器打开 cloudera-manager 客户端 (主节点ip)，admin/admin
$ curl cdh01:7180

# 2.安装过程中查看每个节点的日志
$ sudo tail -f /opt/cm-5.11.2/log/cloudera-scm-server/cloudera-scm-server.log 
$ sudo tail -f /opt/cm-5.11.2/log/cloudera-scm-agent/cloudera-scm-agent.log 


# 3. hive的安装需要 mysql-connector-java.jar (每个节点操作)
$ sudo cp /opt/cm-5.11.2/share/cmf/common_jars/mysql-connector-java.jar /opt/cloudera/parcels/CDH/lib/hive/lib/

# 4. oozie的安装需要 mysql-connector-java.jar (每个节点操作)
$ sudo cp /opt/cm-5.11.2/share/cmf/common_jars/mysql-connector-java.jar /opt/cloudera/parcels/CDH/lib/oozie/libext/
```

<br>



10、解决红色警告

```shell
# 1.HDFS-副本不足的块
# 警告原因
# 是设置的副本备份数与DataNode的个数不匹配。dfs.replication属性默认是3，也就是说副本数（块的备份数）默认为3份，但是我们这里集群只有1个DataNode，所以导致了达不到目标---副本备份不足。
# 解决方法
# a.点击集群-HDFS-配置，搜索dfs.replication，设置为1后保存更改。
# b.dfs.replication这个参数其实只在文件被写入dfs时起作用，虽然更改了配置文件，但是不会改变之前写入的文件的备份数。所以我们还需要步骤b，在 cdh01 中通过命令更改备份数 
$ sudo passwd hdfs
$ su hdfs
$ hadoop fs -setrep -R 1 /
```

<br>



11、安装kafka等外在组件

```shell
# 0.cdh对应支持的kafka版本
$ curl https://www.cloudera.com/documentation/enterprise/release-notes/topics/rn_consolidated_pcm.html#pcm_kafka

# 1.下载kafka包文件，放到 /opt/cloudera/parcel-repo 目录中（注意将以前的 manifest.json文件备份）
$ sudo wget http://archive.cloudera.com/kafka/parcels/2.2/KAFKA-2.2.0-1.2.2.0.p0.68-trusty.parcel
$ sudo wget http://archive.cloudera.com/kafka/parcels/2.2/KAFKA-2.2.0-1.2.2.0.p0.68-trusty.parcel.sha1
$ sudo wget http://archive.cloudera.com/kafka/parcels/2.2/manifest.json

# 2.更改parcel.sha1文件名为parcel.sha
$ sudo mv KAFKA-2.2.0-1.2.2.0.p0.68-trusty.parcel.sha1 KAFKA-2.2.0-1.2.2.0.p0.68-trusty.parcel.sha

# 3.在cm管理页面中点击 主机->Parcel->检查新Parcel(右上角)->KAFKA(分配->激活)

# 4.在cm管理主页面添加kafka服务（注意不要将 zookeeper.chroot 修改为 /kafka，使用默认的 /）

# 5.修改每个节点kafka的配置文件的 brooker.id 和 zookeeper.connect (视情况而定)

# 6.kafka 配置中 搜索 "java heap size"，设置 Java Heap Size of Broker 为1G (默认)，不然broker进程启动不起来
```







