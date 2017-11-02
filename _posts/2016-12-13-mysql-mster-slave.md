---
layout: post
title:  "mysql主备"
desc: "mysql主备"
keywords: "mysql主备,ubuntu,linux,kinglyjn,张庆力"
date: 2016-12-13
categories: [Linux]
tags: [Linux]
icon: fa-bookmark-o
---



### 复制概述

MySQL支持三种复制方式：基于行(Row)的复制、基于语句(Statement)的复制和混合类型(Mixed)的复制。

基于语句的复制早在3.23版本中就存在，而基于行的复制方式在5.1版本中才被加进来。这两种方式都是通过在主库上记录二进制日志、在备库重放日志的方式来实现异步的数据复制。

混合类型的复制：默认采用基于语句的复制，一旦发现基于语句的无法精确的复制时，就会采用基于行的复制。

复制通常不会增加主库的开销，主要是启用二进制日志带来的开销，但出于备份或及时从崩溃中恢复的目的，这点开销也是必要的。除此之外，每个备库也会对主库增加一些负载（例如网络I/O开销），尤其当备库请求从主库读取旧的二进制日志文件时，可能会造成更高的I/O开销。另外锁竞争也可能阻碍事务的提交。最后，如果是从一个高吞吐量的主库上复制到多个备库，唤醒多个复制线程发送事件的开销将会累加。

<br>



### 工作原理

mysql主备复制实现分成三个步骤：

1. master将改变记录到二进制日志(binary log)中（这些记录叫做二进制日志事件，binary log events，可以通过show binlog events进行查看）；
2. slave将master的binary log events拷贝到它的中继日志(relay log)；
3. slave重做中继日志中的事件，将改变反映它自己的数据。

以上只是概述，实际上每一步都很复杂： 

<img src="http://img.blog.csdn.net/20171102172113638?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:70%"/>

- 第一步是在主库上记录二进制日志。在每次准备提交事务完成数据更新前，主库将数据更新的事件记录到二进制日志中。MySQL会按事务提交的顺序而非每条语句的执行顺序来记录二进制日志。在记录二进制日志后，主库会告诉存储引擎可以提交事务了。
- 下一步，备库将主库的二进制日志复制到其本地的中继日志中。首先，备库会启动一个工作线程。称为I/O线程，I/O线程跟主库建立一个普通的客户端连接，然后在主库上启动一个特殊的二进制转储（binlog dump）线程，这个二进制转储线程会读取主库上二进制日志中的事件。**它不会对事件进行轮询**。如果该线程追赶上了主库，它将进入睡眠状态，直到主库发送信号量通知其有新的事件产生时才会被唤醒，备库I/O线程会将接收到的事件记录到中继日志中。
- 备库的SQL线程执行最后一步，该线程从中继日志中读取事件并在备库执行，从而实现备库数据的更新。当SQL线程赶上I/O线程时，中继日志通常已经在系统缓存中，所以中继日志的开销很低。SQL线程执行的事件也可以通过配置选项来决定是否写入其自己的二进制日志中，它对我们稍后提到的场景非常有用。

<br>



### 配置过程

权限配置

```sql
mysql>GRANT SELECT, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'root@'%' IDENTIFIED BY 'root';
```

复制账户事实上只需要有主库上的REPLICATION SLAVE权限，并不一定需要每一端服务器都有REPLICATION CLIENT权限，那么为什么我们要把这两种权限给主/备库都赋予呢？这有两个原因： 

1. 用来监控和管理复制的账号需要REPLICATION CLIENT权限，并且针对这两种目的使用同一个账号更加容易。 
2. 如果在主库上建立了账号，然后从主库将数据克隆到备库上时，备库也就设置好了——变成主库所需要的配置。这样后续有需要可以方便地交换主备库的角色。 
   如果无脑式配置可以：

<br>



主备库配置

关停Master服务器，将Master中的数据拷贝到B服务器中，使得Master和slave中的数据同步，并且确保在全部设置操作结束前，禁止在Master和slave服务器中进行写操作，使得两数据库中的数据一定要相同！ 
备注：文中采用的案例中主备库都有5个schema：

```sql
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| canal_test         |
| mysql              |
| performance_schema |
| test               |
+--------------------+
```

主库的/etc/my.cnf配置(主机host:10.198.197.73)

```shell
[mysqld]
log-bin=mysql-bin
server-id=1
```

备库上也需要在/ect/my.cnf进行配置(备机host:10.198.197.60)

```sql
[mysqld]
log-bin=mysql-bin
server-id=2
relay_log=mysql-relay-bin
log_slave_updates=1
read_only=1
```

server_id 是必须的，而且唯一。slave没有必要开启二进制日志，但是在一些情况下，必须设置，例如，如果slave为其它slave的master，必须设置 bin_log。在这里，我们开启了二进制日志，而且显示的命名(默认名称为hostname，但是，如果hostname改变则会出现问题)。

relay_log配置中继日志，log_slave_updates表示slave将复制事件写进自己的二进制日志(后面会看到它的用处)。

有 些人开启了slave的二进制日志，却没有设置log_slave_updates，然后查看slave的数据是否改变，这是一种错误的配置。所以，尽量 使用read_only，它防止改变数据(除了特殊的线程)。但是，read_only并是很实用，特别是那些需要在slave上创建表的应用。

<br>



启动slave

接下来就是让slave连接master，并开始重做master二进制日志中的事件。你不应该用配置文件进行该操作，而应该使用CHANGE MASTER TO语句，该语句可以完全取代对配置文件的修改，而且它可以为slave指定不同的master，而不需要停止服务器。如下：

```sql
mysql> CHANGE MASTER TO 
    -> MASTER_HOST='10.198.197.73',
    -> MASTER_USER='root',
    -> MASTER_PASSWORD='root',
    -> MASTER_LOG_FILE='mysql-bin.000004',
    -> MASTER_LOG_POS=0;
```

MASTER_LOG_POS的值为0，因为它是日志的开始位置。 
你可以用SHOW SLAVE STATUS语句查看slave的设置是否正确：

```sql
mysql> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: 
                  Master_Host: 10.198.197.73
                  Master_User: root
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000004
          Read_Master_Log_Pos: 4
               Relay_Log_File: mysql-relay-bin.000001
                Relay_Log_Pos: 4
        Relay_Master_Log_File: mysql-bin.000004
             Slave_IO_Running: No
            Slave_SQL_Running: No
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 4
              Relay_Log_Space: 107
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: NULL
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 0
1 row in set (0.00 sec)
```

Slave_IO_State, Slave_IO_Running, 和Slave_SQL_Running是No表明slave还没有开始复制过程。日志的位置为4而不是0，这是因为0只是日志文件的开始位置，并不是日志位置。实际上，MySQL知道的第一个事件的位置是4。

为了开始复制，你可以运行：

```sql
mysql> start slave;
```

运行show slave status查看输出结果：

```shell
mysql> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 10.198.197.73
                  Master_User: root
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000004
          Read_Master_Log_Pos: 2395
               Relay_Log_File: mysql-relay-bin.000002
                Relay_Log_Pos: 253
        Relay_Master_Log_File: mysql-bin.000004
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 2395
              Relay_Log_Space: 409
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 1
```

在这里主要是看:

```shell
    				Slave_IO_Running=Yes
                    Slave_SQL_Running=Yes
```

slave的I/O和SQL线程都已经开始运行，而且Seconds_Behind_Master不再是NULL。日志的位置增加了，意味着一些事件被获取并执行了。如果你在master上进行修改，你可以在slave上看到各种日志文件的位置的变化，同样，你也可以看到数据库中数据的变化。

<br>



> 如果此时Slave_SQL_Running=No，可以参考下下面的方法进行解决：

**解决方案1** 
程序可能在slave上进行了写操作，也可能是slave机器重启后事务回滚造成的。 
如果是事务回滚造成的，可以：

```sql
mysql> slave stop;
Query OK, 0 rows affected (0.00 sec)

mysql> set GLOBAL SQL_SLAVE_SKIP_COUNTER=1;
Query OK, 0 rows affected (0.00 sec)

mysql> slave start;
Query OK, 0 rows affected (0.00 sec)
```

最后通过show slave status进行查看。

<br>

**解决方案2** 

首先停掉slave服务：

```sql
mysql> slave stop;
```

到master上查看主机状态：

```sql
mysql> show master status;
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000004 |     2395 |              |                  |
+------------------+----------+--------------+------------------+
1 row in set (0.00 sec)
```

然后到slave服务器上执行手动同步：

```sql
mysql> change master to 
    -> master_host='10.198.197.73',
    -> master_user='root',
    -> master_password='root',
    -> master_port=3306,
    -> master_log_file='mysql-bin.000004',
    -> master_log_pos=2395;
mysql> slave start;
```

<br>



你可查看master和slave上线程的状态。在master上，你可以看到slave的I/O线程创建的连接(**Binlog Dump**)： 
在master上输入show processlist\G;

```sql
mysql> show processlist\G
*************************** 1. row ***************************
     Id: 30
   User: root
   Host: localhost
     db: canal_test
Command: Query
   Time: 0
  State: NULL
   Info: show processlist
*************************** 2. row ***************************
     Id: 33
   User: root
   Host: zhuzhonghua1-c6uu8.sh.vclound.com:49005
     db: NULL
Command: Binlog Dump
   Time: 33
  State: Master has sent all binlog to slave; waiting for binlog to be updated
   Info: NULL
```

同样，在备库也可以看到两个线程，一个是I/O线程，一个是SQL线程（**Connect**）：

```shell
mysql> show processlist\G
*************************** 1. row ***************************
     Id: 3
   User: root
   Host: 10.198.197.60:62159
     db: NULL
Command: Binlog Dump
   Time: 67811
  State: Master has sent all binlog to slave; waiting for binlog to be updated
   Info: NULL
*************************** 2. row ***************************
     Id: 14
   User: root
   Host: localhost
     db: canal_test
Command: Query
   Time: 0
  State: NULL
   Info: show processlist
*************************** 3. row ***************************
     Id: 19
   User: root
   Host: 10.198.197.60:62390
     db: NULL
Command: Sleep
   Time: 187
  State: 
   Info: NULL
*************************** 4. row ***************************
     Id: 20
   User: system user
   Host: 
     db: NULL
Command: Connect
   Time: 64
  State: Waiting for master to send event
   Info: NULL
*************************** 5. row ***************************
     Id: 21
   User: system user
   Host: 
     db: NULL
Command: Connect
   Time: 64
  State: Slave has read all relay log; waiting for the slave I/O thread to update it
   Info: NULL
```

<br>

案例测试

在master上的Schema Name: canal_test中有一个perosn的表，表结构如下：

```sql
mysql> describe person;
+-------+--------------+------+-----+---------+-------+
| Field | Type         | Null | Key | Default | Extra |
+-------+--------------+------+-----+---------+-------+
| id    | int(11)      | NO   | PRI | NULL    |       |
| name  | varchar(100) | YES  |     | NULL    |       |
| age   | int(11)      | YES  |     | NULL    |       |
| sex   | char(1)      | YES  |     | NULL    |       |
+-------+--------------+------+-----+---------+-------+
```

表中有一条记录：

```sql
mysql> select * from person;
+----+------+------+------+
| id | name | age  | sex  |
+----+------+------+------+
|  2 | zzh2 |   21 | m    |
+----+------+------+------+
```

（注意此时slave中的数据是一样的） 
往master上插入一条数据，之后查看：

```sql
mysql> insert into person values(1,'zzh',22,'m');
mysql> select * from person;
+----+------+------+------+
| id | name | age  | sex  |
+----+------+------+------+
|  1 | zzh  |   22 | m    |
|  2 | zzh2 |   21 | m    |
+----+------+------+------+
```

可以看到master中成功插入了一条数据，之后可以同样在slave中输入select * from person来查看，如果结果master和slave相同，那么恭喜你主备复制已经成功了。

