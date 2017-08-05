---
layout: post
title:  "mysql内置函数、自定义函数、存储过程、存储引擎"
desc: "mysql内置函数、自定义函数、存储过程、存储引擎"
keywords: "mysql内置函数、自定义函数、存储过程、存储引擎,kinglyjn,张庆力"
date: 2012-10-17
categories: [DB]
tags: [DB]
icon: fa-database
---



### 字符串函数

```sql
concat(...) 			--连接函数
concat_ws('-', ...) 	--使用指定的分隔符连接
format(1232.2123, 2) 	--格式化数字，千分位划分数字，保留两位有效数字
lower('AAA') 			--小写
upper('aaa') 			--大写
left('abc', 2) 			--获取左侧2个字符，即前2位字符
right('abc', 2) 		--获取右侧2个字符，即后两位
length('abc')
ltrim('  as')
rtrim('as  ')
trim(' as ')
substring('Mysql', 1,2)	--结果为My,注意这里的字符串的编号从1开始，一课时负数表示倒着取字串
[not] like '%k_k%'		--%表示任意多个字符，_表示任意一个字符		
regexp					--WHERE email REGEXP '\w+@keyllo.com'
replace('aaa$ddf$$$', '$' '') --将字符串中的$替换成空串(全部替换)
```

<br>



### 数值运算函数

```sql
ceil(2.3)				--3
floor(2.3)				--2					
div						--整数除法 select 4 div 3 的结果为 1
mod(4,3)				--取余数
power(2,3)				--指数
exp(0)					--以e为底的指数
ln(2.71828)				--以e为底的对数
log(10)					--以10为底的对数
round(4.523, 2);		--4.52，四舍五入，保留两位小数
truncate(3.436, 2)		--3.43，截断，保留两位小数				
radians(180)			--弧度
degrees(PI())			--角度
sin(PI()/2)				--正弦(以弧度为计算单位)
asin(1)					--反正弦(以弧度为计算单位)
inet_aton(192.168.1.1)	--3232235777, 将ip转换为一个十进制的数字 
inet_ntoa(3232235777)	--192.168.1.1，讲一个十进制数字转换为ip
```

<br>



### 比较运算符

```sql
35 not between 1 and 4	--true(或1，mysql使用tinyint表示boolean类型) 
1 in (1,2,3)			--true(因为1在这个set集合中)
'aaa' is null			--false
```

<br>



### 日期时间函数

```sql
now()			--当前日期和时间  2012-10-17 23:29:35
curdate()		--当前日期 2012-10-17
curtime()		--当前时间 23:29:35
date_add('2012-12-12', interval 365 day)	--当前日期的变化	
datediff('2012-12-12', '2014-12-12')		--两个时间之间的差值
date_format('2012-12-11', '%m/%d/%Y')		--日期格式化
```

<br>



### 信息函数

```sql
connection_id()		--连接的id
database()			--当前数据库
last_insert_id()	--最后插入记录的id(注意如果一次插入多条记录，则只返回第一条插入记录的id)
user()				--当前用户
version()			--mysql版本信息
row_count()			--插入、删除、更新的被影响的记录总数
```

<br>



### 聚合函数(都是返回一个值)

```sql
select avg(score) from t_stu group by clazz_id;
select count(id) from t_stu group by clazz_id;
select max(id) from t_stu group by clazz_id;
select min(id) from t_stu group by clazz_id;
select sum(score) from t_stu group by clazz_id;
```

<br>



### 加密函数

```sql
--AES加密
select aes_encrypt('keyllo', 'abc');
--AES解密
select cast(aes_decrypt(aes_encrypt('keyllo', 'abc'), 'abc') as char);

--md5摘要
select md5('keyllo')
--sha摘要
select sha('keyllo')
```

<br>



### mysql自定义函数（UDF）

```sql
/*
自定义函数的两个必要条件：
1. 参数(输入)
2. 返回值(输出)

自定义函数的创建语法：
create funtion func_name
returns {string|integer|real|decimal}
routine_body

关于函数体routine_body：
1. 函数体可以是简单的增删改查语句
2. 函数体如果是复合结构则使用 begin...end
3. 函数体可以包含声明、分支、循环等流程控制语句

注意: 使用函数的字段将导致索引失效!
*/

--一些示例：
create function f1() returns varchar(30)
return date_format(now(), '%Y年%m月%d日 %H点:%i分:%s秒');   

select f1();
show create function f1;



drop function f2;

create function f2(num1 smallint, num2 smallint) returns float(10,2)
return (num1+num2)/2;

select f2(1,2);



DELIMITER // 
create function add_clazz(clazz_name varchar(40)) returns int
begin
	insert into t_clazz2(name) values(clazz_name);
    return last_insert_id();
end//
DELIMITER ;

select add_clazz('25班');
```

<br>



### 存储过程

存储过程是sql语句和控制语句的预编译集合，以一个名称存储作为一个单元处理，存储过程可以接收输入、输出类型的参数，并且可以输出多个返回值。存储过程执行的效率一般会比单一的sql执行的效率要高，原因在于mysql要对我们写的sql语句进行逐一的语法分析、编译和执行，而如果我们采用了存储过程以后，只有第一次执行才进行语法分析和编译，以后客户端再次调用就会直接来调用编译的结果即可，省略了sql的语法分析和编译到的环节。

存储过程的优点：

* 增强了sql语句的功能和灵活性
* 实现了较快的执行速度（存储过程是预编译的，省略了sql的语法分析和编译的过程）
* 减少了网路流量

<br>

```sql
/*
存储过程的创建语法：
create [definer={user|current_user}] procedure pro_name([param[,..]])
[characteristic ...]
routine_body

存储过程的参数：
[IN|OUT|INOUT] param_name type
IN: 表示该参数的值必须在调用存储过程的时候被指定
OUT: 表示该参数的值可以被村粗过程所改变，并且可以返回
INOUT: 表示该参数必须在调用时被指定，并且可以被改变和返回

存储过程的调用：
call pro_name(param[,..])  --调用有参数的存储过程
call pro_name			   --调用无参数的存储过程
*/


--一些示例
create procedure p1()
select version();

call p1();



delimiter //
create procedure delete_clazz_by_id(IN id int unsigned)
begin
	delete from t_clazz2 where id=id;  
end //
delimiter ;

--调用该存储过程的时候我们发现，t_clazz2表中的所有记录都被删除了，原因在于where id=id，
--输入参数id的名称和t_clazz2表中的id名称重复了，mysql将这两个id都当成了t_clazz2数据表的id
--由此我们也就直到，创建存储过程的时候，需要尽量避免IN参数和数据库表的字段名称相同！
call(1) 


--drop procedure if exists delete_clazz_by_id;
delimiter //
create procedure delete_clazz_by_id(IN p_id int unsigned)
begin
	delete from t_clazz2 where id=p_id;
end //
delimiter ;
--call(1) 



delimiter //
create procedure delete_clazz_and_return_nums(IN p_id int unsigned, OUT p_nums int unsigned)
begin 
	declare aaa int default 0;  --declare必须位于第一行，declare声明的变量称为局部变量
	set aaa = 1;
	delete from t_clazz2 where id=p_id;
    select count(*) from t_clazz2 into p_nums;
end //
delimiter ;

call delete_clazz_and_return_nums(27, @nums);
select @nums; --@nums称为用户变量，他是跟mysql的客户端绑定的，而@@xxx称为全局变量
```

<br>



### 存储引擎

mysql可以将数据以不同的技术存储在文件（内存）中，这种技术就称之为存储引擎。每一种存储引擎使用了不同的存储机制、索引技巧、锁定水平，最终提供广泛且不同的功能。

mysql支持的存储引擎：

* MyISAM：适用于事务处理不多的情况

  存储限制256TB，不支持事务，支持索引，锁颗粒为表锁，支持数据压缩

* InnoDB：适用于事务处理比较多，且需要外键支持的情况

  存储限制64TB，支持事务，支持索引，锁颗粒为行锁，不支持数据压缩，支持外键

* Memery

  存储限制有，不支持事务，支持索引，锁颗粒为表锁，不支持数据压缩

* Archive

  存储限制无，不支持事务，不支持索引，锁颗粒为行锁，支持数据压缩

```shell
# 通过配置文件修改默认的存储引擎
default-storage-engin = InnoDB

# 通过创建数据库表的时候指定存储引擎
cerate table xxx(
	...
) engine=InnoDB default charset=utf8;

# 修改表的存储引擎
alter table table_name engine=InnoDB;

# 查询默认存储引擎
show engines;
```



<br>

> [注1] 常见索引类型：
>
> btree索引、hash索引、普通索引、唯一索引、全文索引等。



> [注2] 所以失效的情形：
>
> - 请求表上的数据行超出表总记录数30%，变成全表扫描
> - 谓词上的索引列上存在NULL值
> - 谓词上的索引列条件使用函数
> - 谓词上的索引列条件进行了相关运算
> - 谓词上的索引列条件上使用了<>，NOT IN操作符
> - 复合索引中，第一个索引列使用范围查询--只能用到部份或无法使用索引
> - 复合索引中，第一个查询条件不是最左索引列
> - 模糊查询条件列最左以通配符%开始
> - 内存表（HEAP表）使用HASH索引时，使用范围检索或者ORDER BY
> - 表关联字段类型不一样（包括某些长度不一样，但像varchar(10)与char(10)则可以，MYSQL经过内部优化处理）



> [注2] 索引类型（按用途非严格划分）：
>
> - 普通索引，这是最基本的索引，无任何限制
> - 唯一索引，与普通索引类似，索引列值必须唯一，允许NULL值
> - 全文索引，基于词干方式创建索引，多用于BLOB数据类型
> - 单列索引，仅基于一列创建的索引
> - 多列索引，基于多列创建的索引，列顺序非常重要
> - 空间索引，用作地理数据存储
> - 主键索引，是一种特殊的唯一索引，不允许有NULL值，通常在建表时创建。



> [注4] 索引的优缺点       
>
> 索引的优点
>
> - 大大减少了服务器需要扫描的数据量
> - 可以帮助服务器避免排序或减少使用临时表排序
> - 索引可以随机I/O变为顺序I/O
>
> ​       
>
>  索引的缺点
>
> - 需要占用磁盘空间，因此冗余低效的索引将占用大量的磁盘空间
> - 降低DML性能，对于数据的任意增删改都需要调整对应的索引，甚至出现索引分裂
> - 索引会产生相应的碎片，产生维护开销

