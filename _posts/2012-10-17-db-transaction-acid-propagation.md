---
layout: post
title:  "事务的基本属性（ACID）和传播属性（Propagation）"
desc: "事务的基本属性（ACID）和传播属性（Propagation）"
keywords: "事务的基本属性（ACID）和传播属性（Propagation）,ACID,Propagation,kinglyjn"
date: 2012-10-17
categories: [DB]
tags: [DB]
icon: fa-database
---

### 数据库事务的四个基本性质简介

* `原子性（Atomicity）`，在一个事务中的所有操作，相当于一个原子操作，要么全部成功，要么全部失败。
* `一致性（Consistency）`，就是在事务执行前后，对于事务本身的用意而言，数据库中的数据是保持一致的，数据库的一致性是建立在原子性的基础之上的，更多的由编码的程序员保证，最经典的案例是A，B帐号之间的转账。
* `隔离性（Isolation）`，事务的隔离性是指事务和事务之间的数据可见性，这也是本文需要详细介绍的地方。数据库定义了各种隔离级别，以在并发性和数据完整性中权衡。
* `持久性（Durability）`，事务完成以后，所有的数据都将持久到数据库中，不会因为其他原因而丢失。

<br>



### 事务的隔离级别

数据库的隔离级别主要分为四级，隔离级别越高，并发性就越低，一致性就越高。<br>

* `序列化读`：所有事务的执行都是顺序执行；
* `可重复读`：事务A修改或插入了某些记录并提交，事务B是无法看到的，【除非事务B重新修改了这条记录（在事务B中没被修改的事务A的那些数据还是看不到的）或事务B提交】，这时候才能够看到事务A的修改或插入。事务B修改这条记录导致这条记录在事务A中的更新或插入出现在事务B中的现象就是幻读（从幻读这个名字就可以看出来，就像是在事务B中出现幻觉一样！）。
* `读已提交`：事务A修改或插入了某些记录并提交，事务B能够直接读取到，这会导致事务B除了出现幻读现象外，还会出现不可重复读取的现象（即前后查询读到的两个结果不一样）。
* `读未提交`：事务A修改或插入了某些记录还没有提交，事务B就能够读到，这会导致事务B除了出现幻读、不可重复读现象外，还会出现脏读的情况（因为事务B能够看到事务A中还未提交的事务，如果事务A的事务发生了回滚，那么事务B中读到的就是脏数据！）



以上这些隔离级别都是定义在java.sql. Connection中，在获取连接的时候我们可以进行设置，但是一般情况下，系统会用数据库默认的级别来设置。<br>

```default
Connection.TRANSACTION_SERIALIZABLE;
Connection.TRANSACTION_REPEATABLE_READ;
Connection.TRANSACTION_READ_COMMITTED;
Connection.TRANSACTION_READ_UNCOMMITTED;
```

小结：<br>

<img src="http://img.blog.csdn.net/20170112165319819?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:60%"/><br><br>



### 事务的传播属性

我们在使用Spring或者EJB声明式编程的时候，还会接触到事务的传播属性（Propagation），在spring的 TransactionDefinition 类里面进行定义。具体有以下几个数值：<br>

* Required(需要)
* Mandatory(强制必须)
* RequiresNew(需要新的)
* Supports(支持)
* NotSupported(不支持)
* Never(不用)

`Required`：当前方法必须要求开启事务，如果当前线程不存在事务，则开启新的事务，如果当前线程已经存在事务，就加入到当前事务。这个是经常使用的。但是要注意的就是一旦事务中某一个方法回滚，当前事务上下文里面所有的操作都回滚，考虑到下面一个例子：<br>

<img src="http://img.blog.csdn.net/20170112165912281?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:60%"/><br>

假设A和B的事务都是Required，那么当调用MethodA的时候，如果methodB回滚了，对A的修改也就回滚了。所以上面的代码不会达到预期的结果，也就是说A不可能修改成为99。

`Required New`：当前方法必须要求开启新的事务，如果当前线程已经存在事务上下文，就暂停当前事务，等到新事务结束之后，再继续恢复之前的事务。就拿上面的例子来说，methodB的对事务的修改不会影响到methodA。两个事务之间不会互相影响。经常可以用到的场景就是在业务发生异常的时候发送短消息。如果业务发生异常，业务回滚，但是由于发送段消息是新的事务，不会受到业务异常的影响。<br>

`Mondary`：当前方法必须要求事务，如果当前线程不存在事务，就抛出异常，如果存在，就加入到事务里。<br>

`Support`：当前方法支持事务，如果当前线程存在事务，就加入到事务中去，如果不存在，不做任何操作。<br>

`Not Support`：当前方法不支持事务，如果当前线程存在事务，就挂起当前事务，执行完当前方法，恢复事务。一般情况下在查询的时候使用，如果一个方法只是查询，并且非常耗时，就可以使用Not Support，避免事务时间超长。<br>

`Never`：当前方法不支持事务，如果当前线程存在事务，则抛出异常。这种用的比较少。<br>

<br>



### Mysql查看自动提交 和 隔离级别实验

```sql
--查看全局是否是自动提交事务
select @@autocommit;

--查看全局级别的事务隔离级别
select @@GLOBAL.tx_isolation;
REPEATABLE-READ

--查看会话级别的事务隔离级别
select @@SESSION.tx_isolation;
REPEATABLE-READ

--设置会话级别的事务的隔离级别
set session transaction isolation level serializable;
set session transaction isolation level repeatable read;
set session transaction isolation level read committed;
set session transaction isolation level read uncommitted;
```

