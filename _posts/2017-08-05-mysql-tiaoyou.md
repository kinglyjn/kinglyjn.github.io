---
layout: post
title:  "mysql 调优"
desc: "mysql 调化"
keywords: "mysql 调化,mysql,kinglyjn"
date: 2017-08-27
categories: [db]
tags: [db]
icon: fa-database
---



### 数据库优化的几个入口

从优化效果及成本考虑，可以依次从以下几个方面考虑：

* SQL及索引
* 数据库表结构
* 系统配置
* 硬件

<br>



### 数据准备([说明](https://downloads.mysql.com/docs/sakila-en.pdf))

```sql
$ curl http://dev.mysql.com/doc/index-other.html
$ wget three files: sakilaschema.sql, sakila-data.sql, and sakila.mwb
$ mysql -uxxx -p
mysql> SOURCE xxx/sakila-schema.sql;
mysql> SOURCE xxx/sakila-data.sql;
mysql> USE sakila;
mysql> SHOW TABLES;
mysql> SELECT COUNT(*) FROM film;
mysql> SELECT COUNT(*) FROM film_text;
```

<br>



### SQL及索引优化

发现有问题的sql：

```shell
## 使用mysql慢查询日志对有效率问题的sql进行监控

# 查看及设置慢查询日志存储的日志文件的位置在哪里(默认的慢查询日志在 /var/lib/mysql/xxx-slow.log)
mysql> show variables like '%query_log%';
mysql> set global slow_query_log_file='/home/ubuntu/sql_log/mysql_slow.log';
# 设置开启慢查询日志
mysql> set global slow_query_log=on;
# 设置将没有使用索引的sql记录到慢查询日志中
mysql> set global log_queries_not_using_indexes=on;
# 设置将超过多少秒的sql记录到慢查询日志中
mysql> set global long_query_time=1;


## 慢查询日志的格式
$ sudo tail -f /var/lib/mysql/xxx-slow.log
----------------
# Time: 180602  9:49:17   --执行sql的时间
# User@Host: root[root] @ localhost []  Id:11   --执行sql的主机信息
# Query_time: 0.000125  Lock_time: 0.000060 Rows_sent: 2  Rows_examined: 2  --sql执行信息
SET timestamp=1527904157; --sql执行时间的时间戳
select * from store limit 10; --sql的内容


## 慢查日志的分析工具

# 1. mysql自带的官方工具 mysqldumpslow [--help]
# 分析得出前3条慢查询日志
$ sudo mysqldumpslow -t 3 /var/lib/mysql/xxx-slow.log

# 2. pt-query-digest(提供了更为详细的慢查询日志信息)
# 安装
$ wget percona.com/get/pt-query-digest
$ sudo chmod u+x pt-query-digest
$ sudo chown root:root pt-query-digest
$ sudo mv /xxx/pt-query-digest /usr/bin/
# 使用
$ sudo pt-query-digest /var/lib/mysql/xxx-slow.log
$ sudo pt-query-digest /var/lib/mysql/xxx-slow.log -review \
h=127.0.0.1,P=3306,D=test,u=root,p=xxx,t=query_review \
--create-reviewtable \
--review-history t=hostname_show


## 如何通过慢查询日志发现有问题的sql？

# 1. 查询次数多且每次查询占用时间长的sql(通常为pt-query-digest分析的前几个查询语句)
# 2. IO大的sql(注意pt-query-digest分析中的 Rows examine 项，表示扫描了多少行)
# 3. 未命中索引的sql(注意pt-query-digest分析中的 Rows examine 和 Rows Send 的对比)
#    如果 扫描的行数 远远大于 发送回来的行数，就表示索引的命中率不是很高
```

<br>

找到了具体慢sql之后，如何对其进行优化？

```default
# 使用 explain 分析慢sql
mysql> explain xxxsql;

# 输出结果字段
table：表名
type：连接的类型，从好到差的连接类型依次为 const、eq_reg、ref、range、index、ALL
	  const：主键、索引
	  eq_reg：主键、索引的范围查找
	  ref：连接的查找（join）
	  range：索引的范围查找
	  index：索引的扫描
	  All：表扫描
possible_keys：可能用到的索引，如果为空则表示没有可能的索引
key：实际使用的索引，如果为空则表示没有使用索引
key_len：使用的索引的长度，在不损失精确性的前提下长度越短越好
ref：显示索引的那一列被使用了，如果可能的话最好是一个常数
rows：mysql认为必须检查的返回请求数据的行数(一般为需要扫描的行数)
extra：需要注意的有这两个枚举值 Using filesort 和 Using temporary，这两个值出现之后，通常需要优化
       Using filesort 表示mysql需要进行额外的步骤对返回的行进行排序，它根据l连接的类型以及存储排序键值
       和匹配条件的全部行的行指针来排序全部行；
       Using temporary 表示mysql需要创建一张临时表来存储结果，通常发生在对不同的列集进行order by上，
       而不是group by上。
```

<br>

如何查找数据库重复冗余索引？

```default
## 使用 pt-duplicate-key-checker 工具
$ pt-duplicate-key-checker -h 127.0.0.1 -uroot -p 'xxxxxx'
```

<br>



### 数据库表结构优化

```shell
## 建表选择合适的数据类型
# 1. 使用可以存储下你的数据的最小数据类型
# 2. 使用简单的数据类型，int要比varchar类型在mysql处理上简单
# 3. 尽可能的使用 not null 定义字段
# 4. 尽量少用 text 类型，非用不可时最好考虑分表


## 示例
# 使用 int 存储日期时间
create table xxx(
    id int primary key auto_increment,
    updatetime int
);
insert into xxx(updatetime) values(unix_timestamp('2017-08-05 12:12:23'));
select from_unixtime(updatetime) from xxx;

# 使用 bigint 存储ip地址
create table xxx2(
	id int primary key auto_increment,
	ip bigint
)
insert into xxx2(ip) values(inet_aton('192.168.0.23'));
select inet_ntoa(ip) from xxx2;


```

<br>





