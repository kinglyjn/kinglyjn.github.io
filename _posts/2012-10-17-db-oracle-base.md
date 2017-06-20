---
layout: post
title:  "oracle基础简介"
desc: "oracle基础简介"
keywords: "oracle基础简介,oracle,sql,kinglyjn"
date: 2012-10-17
categories: [DB]
tags: [oracle,sql]
icon: fa-database
---

### 数据库的分类

按照项目的规模分类

* 小型数据库 --负载在百十来号人，成本在1000以内，安全性要求不高。ep留言本、信息发布系统
    * access foxbase
* 中型数据库 --负载在5000到15000左右，成本在万元以内，安全性要求均等。ep比如商务网站
    * mysql
    * sqlserver
    * informix
* 大型数据库  --凡在为海量T级别，成本高，联通电信移动，安全性高
    * sybase 
    * oracle 
    * db2(IBM)	

<br>

### oracle的卸载

```default
停止Oracle所有服务
	运行oracle的universal installer卸载oracle
	运行regedit，进入注册表，删除时小心别删错
		oracle软件有关键值：
		HKEY_LOCAL_MACHINE\SOFTWARE\ORACLE
		oracle服务：
		HKEY_LOCAL_MACHINE\SYSTEM\CurrentControSet\Services下以oracle开头的键值
		oracle事件日志：
		HKEY_LOCAL_MACHINE\SYSTEM\CurrentControSet\Services\Eventlog\Application
	删除Oracle系统目录C:\program files\oracle
	删除oracle环境变量
	删除程序菜单中的oracle菜单	
	重启计算机，然后删除硬盘上的oracle目录，如果该目录不让删除，把这个目录改成别的名字，重启计算机再删除
	删除在program files下的oracle目录
```
<br>

### oracle的安装

```default
默认生成sys用户和system用户：
		1.sys用户是超级用户，具有最高权限，具有sysdba角色，有create database的权限，默认密码是manager
		2.system用户是管理操作员，权限也很大，有sysoper角色，没有create database的权限，默认密码是change_on_install
		3.一般来讲，对数据库维护，使用system用户登陆就可以了
		
		oracle安装完成后有两部分：
			C/S体系结构：
				|客户端部分
				|	sqlplus（oracle提供，对数据进行操作管理的一个client软件，可执行sqlplus和sql命令）
				|	进入：
				|	使用：查看和当前用户相关的表（sql命令）
				|		  sql>select table_name from user_tables;
				|		  查看一个表的表结构（sqlplus命令）
				|		  sql>desc emp;
				|		  退出
				|		  sql>exit;
				|	iSqlPlus:
				|      （oracle提供，基于浏览器，对数据进行操作管理的一个client软件，可执行sqlplus和sql命令）
				|	    进入：http://localhost:8080/apex
				|		 					 
				|服务器部分
				|	oracleServiceXE（核心服务）、oracleXETNSListener（提供对外支持）
```
<br>

### sql相关概念

```default
SQL: (Structure Query Language) 结构化的查询语言，对数据库中的数据进行增、删、改、查等操作。适用于所有的RDBMS.
PL-SQL: 是在SQL命令基础上进行了功能扩展，只能用在oracle.
sqlPlus：oracle提供，用于数据进行操作管理的一个client软件，可执行sqlplus命令和sql命令.

SQL分类：
	DQL:data query language 数据查询语言 （select） 【重点】
	DDL：data Defination Language 数据定义语言 (create\alter\drop)
	DML:data manipulate language 数据操纵语言 (insert\update\delete) 【重点】
	DCL:data controller language 数据控制语言 (grant\revoke)
	TCL:transaction controller language 事务控制语言 (commit\rollback)
```
<br>

### oracle管理工具

1. oracle自带的工具软件，主要用于执行sql语句，p1\sql块
    * 第一种方式：在开始->程序->oracle orachome90->application developmentp->--(sql*plus)//
    * 第二种方式：在"运行栏"输入：--(sqlplusw)即可
    * 第三种方式：在开始->程序->oracle orachome90->application developmentp->--(sql*plus worksheet)
2. sqlplus dos下操作oracle的工具，其功能和sqlplus功能类似,在"运行栏"输入--sqlplus，--也可在cmd中输入sqlplus//
3. oracle的企业管理器(oem oracle enterprise manager)
    * 开始->程序->oracle->oracle oraclehome90->enterprise manager console即可启动oracle的企业管理器是一个图形界面环境
4. 第三方软件：p1/sql develop,主要用于开发测试优化oracle p1/sql的存储过程，比如触发器，需要单独安装 --plsqldev//

<br>

### sqlplus常用命令

```default
0.备份的导入导出(需要在dos命令窗口下操作)
		exp:导出（导出路径为当前路径，文件名为EXPDAT.DMP）
		imp:导入（先进入到文件所在当前目录下执行）	

1.清屏：
	方法一：同时按SHIFT和DELETE键然后点OK就可以了 。
	方法二：如果在window窗口下sqlplus 中清屏命令：host cls 或是clear screen 或只是4位 clea scre。
	方法三：如果是在dos的窗口下进入sql/plus就要用clear SCR。

2.连接命令：
	启动：
	>>lsnrctl start （可填监听的名字，不写的话启动默认的监听）--启动监听
	>>oradim -starup -sid orcl  --启动oracle实例
	
	连接和断开等：
	conn(或connect) 用户名/密码@网络服务名 [as sysdba/sysoper]  --当用特权名登录连接时  conn sys as sysdba
	例如：conn system/"密码" --显示已连接，用户已然修改成为system	
	disc --该命令用于断开数据库的连接
	passw --该命令用于修改用户的密码，如要修改其他用户的密码，需要用sys/systemdeng登陆
	show user --显示当前用户
	exit --断开于数据库的连接，同时退出sql*plus
	
3.文件操作命令：
	start或@ --运行脚本
			 --ep: sql>@d:\a.sql 或者 sql>START d:\a.sql
	edit	 --编辑指定的sql脚本
			 --ep: sql>edit d:\a.sql
	spool	 --该命令可以将sql*plus屏幕上的内容输出到指定文件中去
			 --ep: sql>spool d:\bb.txt 
			 --    sql>select * from emp; --注意带分号表示要执行语句
			 --    sql>spool off
			 --	   上述文件执行完毕后在d:盘目录新建了bb.txt并且屏幕上刚出现的内容被输出到bb.txt
	select   --查询
			 --sql> select * from emp; //无条件查询
			 --sql> select * from emp where ename='SMITH'; //有条件查询
			 --sql> select * from emp where ename='&name';  sql>输入 name 的值:  SMITH  //等价于上述
	
	--如何知道比如scott用户可以操作哪些表？
		--建议用plsqldev来看(登陆scott用户，在tables中看)
	
4.显示和设置环境变量
	可以用来控制输出的格式，set show 如果希望永久保存相关的设置，可以去修改glogin.sql脚本
	linesize  --设置显示行的宽度，默认是80个字符
			  --sql>show linesize
			  --sql>set linesie 90
	pagesize  --设置每一页显示的行数目，默认是14,
			  --sql>show pagesize
			  --sql>set pagesize

5.创建和删除用户以及权限的维护
	在sys或system中创建用户：--sql>create user kingly identified m123 //用户kingly，密码m123
	改密码方式一：--sql>password 用户名 //或者 passw//
	改密码方式二：--用dba的权限用户或有alter权限的用户给别人修改密码：--sql>alter 用户名 identified by 新密码
	
	删除用户：一般以dba的身份去删除某个用户，若用其他用户去删除则需要有drop user的权限(不能自己删自己！)
			  --drop user 用户名 【cascade】 --级联删除
	
	创建的新用户是没有任何权限的，甚至连登陆数据库的权限都没有，需要为其指定权限。
	>>给一个用户赋权限使用grant; >>回收用户权限使用命令revoke
	  --ep: grant create table to zhangsan --把创建表的权利给zhangsan
	  --ep: grant unlimited tablespace to zhangsan --创建不受限的表空间的权利给zhangsan
	  --ep: revoke create table from zhangsan; --将zhangsan创建一张表的权限撤销掉
	  --ep: revoke unlimited from zhangsan; --将zhangsan不受限空间的权限撤销
					
		权限|——系统权限：140多种，用户对数据库的相关权限  ep:create session(登陆数据库的权限)
			|——对象权限：关键的有20多种，用户对其他用户的数据对象
			           (表、视图、函数、过程、包、类型、触发器、工作、库、序列、同义词、表空间、簇……)操作的权限
			           主要有：select
			                  insert
			                  update
			                  delete
			                  all
			                  create index……								
		角色(权限的批量授权，ep现实生活中一个人当了"省长"这个角色，他就拥有了"省长"的一系列权限) 
			|——预定义角色
			        ep: connect角色(有7种权限)
				        resource角色(这个角色可让授权的用户在任何表空间建表)
						dba角色(管理员)
			|——自定义角色
			
	
	希望xiaoming(connect角色)这个用户可以在system表空间中建表:
	--希望xiaoming(connect角色)这个用户可以在system表空间中建表  (普通用户对数据库的相关权限)
	    --(在system用户中授权给xiaoming)sql>grant resource to xiaoming;
	    --(在xiaoming用户中建表)sql>create table test(userId varchar2(30), userName varchar2(30));
	    --(查询表)sql>select * from test;
	    --(查看表的结构)sql>desc test; //(description)
	
	希望xiaoming(connect角色)这个用户可以查询scott的emp表:
	--希望xiaoming(connect角色)这个用户可以查询scott的emp表 (普通用户的对象权限)
	    --(在sys或system或scott中授权)--sql>grant select on scott.emp to xiaoming;
	                                --sql>grant select on emp to xiaoming;
	    --(在xiaoming用户中查询scott(方案)的emp表)sql>select * from scott.emp;
		
	--希望xiaoming(connect角色)这个用户可以修改scott的emp表 (普通用户的对象权限)
		--(在sys或system或scott中授权)--sql>grant update on scott.emp to xiaoming;
		--sql>grant update on emp to xiaoming;
		--(在xiaoming用户中修改scott(方案)的emp表)
	
	希望xiaoming(connect角色)这个用户可以增加、删除、修改、查询scott的emp表	
	--希望xiaoming(connect角色)这个用户可以增加、删除、修改、查询scott的emp表 (普通用户的对象权限)
		--(在sys或system或scott中授权)--sql>grant all on scott.emp to xiaoming;
		                            --sql>grant all on emp to xiaoming;
	
	希望收回小明对emp表的查询权限:
	--希望收回小明对emp表的查询权限
		--(在sys或system或scott中收回)--sql>revoke select on scott.emp from xiaoming;
		                            --sql>revoke select on emp from xiaoming;
									  
	
	希望xiaoming这个用户可以查询scott的emp表，还希望xiaoming可以把这个权限给别人: ***with grant option
	--希望xiaoming这个用户可以查询scott的emp表，还希望xiaoming可以把这个权限给别人
		--(在sys或system或scott中授权)--sql>grant select on scott.emp to xiaoming with grant option;
		                            --sql>grant select on emp to xiaoming with grant option;
	希望xiaoming这个用户可以有connect这个角色(系统权限)，还希望xiaoming可以把这个权限给别人:***with admin option
		--(在sys或system或scott中授权)--sql>grant connect to xiaoming with admin option;
	【注】:系统回收xiaoming的权限，将会影响xiaoming授予其他用户的相应权限。
		   ep:部长(系统用户)封了个省长(普通用户),省长又封了许多县长(普通用户),
			  撤掉省长之后,也会株连下面诸多县长的相应权限。 (一人得道,鸡犬升天; 一人受株,全家株连)
	
	
6.使用profile管理用户口令：
	概述：profile是口令限制，资源限制的命令集合，当建立数据库时，oracle会自动建立名为default的profile，当建立
		  用户没有指定profile选项，那么oracle就会将default分配给用户。
	账户锁定：指定该账户(用户)登录时最多输入密码的次数，也可以指定用户锁定的时间(天)。
			  一般用dba的身份去执行该命令。
			  --ep: 指定scott这个用户最多只能尝试登陆3次，锁定的时间为2天。
			  --//首先创建profile文件
              --sql>create profile dog(配置名) limit failed_login_attempts 3 password_lock_2;
			  --//指定xiaoming账户用这个配置文件
              --sql>alter user xiaoming profile dog;
	给账户解锁: (给管理员打电话吧)--sql>alter user xiaoming account  unlock;
	
	强制定期修改密码(一般是管理员才做这种事)：
			  --ep:给前面创建的用户xiaoming创建一个profile文件，要求用户每隔10天修改自家的登录密码，宽限为2天。
			  --sql>create profile dog limit password_life_time 10 password_grace_time 2;
			  --sql>alter user xiaoming profile dog; 
			  解锁:--sql>alter user xiaoming account unlock;		
	修改密码时不能使用重用密码：
			  --ep:
			  --sql>create profile password dog limit password_life_time 10 password_grace_time 2 
			  --proword_reuse_time 10 （10天后才可重用密码）
			  --sql>alter user xiaoming profile dog;
			  解锁:--sql>alter user xiaoming account unlock;
			  删除profile:--sql>drop profile dog 【cascade】;
```
<br>

### 表查询

#### 表名和列名的命名规则

* 必须以字母开头
* 只能用如下字符：A-Z a-z 0-9 $ # 等
* 不能使用oracle的保留字
* 长度不能超过30个字符

#### oracle支持的`数据类型`

* 字符型
    * char，定长，最大2000字符，查询极快，浪费空间 --char(10) '小张'  //前四个字符放'小张',后添6个空格补全 
    * varchar2，变长，最大4000字符，查询稍慢，节省空间  --varchar(20) '小张'//oracle分配4个字符存放'小张'
    * clob，字符型大对象，最大4G, 存放txt文件

* 数字型：
    * number，范围 -10^38~10^38 可表示整数，也可以表示小数(10^-38)
    * number(5,2，范围 -999.99~999.99 表示一个小数有5位有效数字，2位小数
    * number(5)，范围 -99999~99999 表示一个5位整数	
    ```default
    number(5, 2) 整数位最多为3位，小数位2位(多余的小数位四舍五入)
		如123.45 ok, 1234.5 error!	12.3456 ok
	number(5) 整数位最多为5位，无小数位
		如12345、1、123 ok  123.4 error!
	number 默认小数类型，小数最多占7位，不含小数位的数字位数是38位	
    ```
* 日期类型：
    * date，包括年月日时分秒，精确到秒
    * timestamp，对date数据类型的扩展，精确到毫秒
* 多媒体类型：
    * blob，二进制数据，可以存放图片/声音 4G，通常的老套路是在一个普通文件夹中存放图片或声音，而在数据库中存放图片或声音的路径，如果要求保密性高的话，可以将声音或者图片存在数据库中

#### 建表、删除表、修改表名

建表：<br>

```sql
--学生表
sql>create table student(  --表名
	xh number(4),  --学号
	xm varchar2(20),  --姓名
	sex char(2),  --性别
	birthday date,  --出生日期
	sal number(7,2)  --奖学金, -99999.99~99999.99 【注】最后没有逗号！
);
--显示表结构sql>disc student;
--【注】也可在图形界面工具中建表(table)

--班级表
sql>create table class(
	classid number(2),
	ename varchar2(40)
);
```

删除表：<br>

```sql
drop table student；
```

修改表名：<br>

```sql
rename student to stu;
```
<br>

#### 添加、删除、修改一个字段

```sql
--增加一个字段：
alter table student add (classid number(2));
--删除一个字段：
alter table student drop column sal;
--修改字段的长度：
alter table student modify(xm varchar2(30));
--修改字段的类型(不能有数据,危险！)
alter table student modify(xm char(30));
--修改字段名
alter table student rename column xm to name;
```
<br>

#### 插入字段内容

```sql
--插入所有字段(列)：
insert into student values(1, '张三', '男', '27-5月-90', 10); --1990年5月27日, '月'字必须带!
	--oracle中默认的日期格式'DD-MM-YY',修改日期的格式命令如下：
	--alter session set nls_date_format = 'yyyy-mm-dd'; //修改后就可以用我们熟悉的格式添加日期类型
insert into student values(1, '张三', '男', '1990-05-27', 10);		

--插入部分字段(列):
insert into student(xh, xm, sex) values(1, '张三', '男'); --不一定成功，某些字段必须要赋值	

--插入空值: 
insert into student(xh, xm, sex, birthday) values(4, '王五', '男', null); 
--【注】查询空值：select * from student where birthday is null; 
--查询非空的值：  select * from student where birthday is not null;
```
<br>

#### 删除数据内容

```sql
delete from student; --删除所有记录，写日志，可恢复，速度慢
			--sql>savepoint aa; //设置保存点aa
			--sql>rollback to aa; //回滚到保存点状态，中间的活白干！	

truncate table student; --删除表中所有的记录，表结构还在，不写日志，无法找回删除的记录，速度极快！
drop table student; --删除表的结构和数据
delete from student where xh=1; --删除一条记录	
		
--delete
--语法：delete from 表名 [where 过滤条件]
sql> delete from students where stuid=1;
sql> delete from clazz where clazz=2; --error！（原因：从表中有相关数据）
	--解决方案：
	--方案一：先删除该班的学生，再删除该班级
	--方案二：先把该班的学生转到其他班，再删除该班级
	--方案三：先去掉外键约束，再删除班级
		--【注】在删除数据时，如果删除的是主表（如：班级）中的数据，必须要考虑是否删除后会导致从表中
				--（如学生表）的数据会不会丢失引用关系
                --开发中常用删除主表通常是逻辑删除，实际上数据还是物理存在的：
		create table student(
			stuid number(4) constraint PK_id primary key, --PK_id为约束别名
			name varchar2(20) [constraint 约束名] not null,
			deleteflog number(1) default 0 check(deleteflog in(0, 1)) --逻辑删除标志
		);
```
<br>

#### 修改字段内容

```sql
update student set sex='女' where xh=1; --修改单个字段
update student set sex='女', birthday='27-5月-90' where xh=1; --修改多个字段
update student set sex='女' where birthday is null; --修改含有空值				
update student set sal=sal/2 where sex='男';--把所有男同志的薪水统统减一半
```
<br>

#### 约束相关操作

```default
主键约束：primary key
        作用：唯一标识表中的一行数据，如工号、学号……
        特点：唯一、非空
外键约束：foreign key
        语法：references 主表名(pk字段名)  --主表（如班级）、从表（如学生）
        作用：标识从表中的外键字段的值，必须引用主表中主键字段pk(唯一键(约束))已存在的值--只有俩班级
             如员工所在部门编号、学生所在班级编号
        特点：允许为空、允许重复				  
唯一约束：unique
        作用：标识该列的值，不允许重复。如邮箱、银行卡号、身份证号……
        特点：唯一、可以为空
非空约束：not null
        作用：标识该列的值必须有内容，不允许是null。如姓名、……
        特点：可重复、非空
检查约束(自定义约束)：check(约束表达式)
        作用：标识该列的值必须符合用户的自定义表达式要求。如手机号必须是11位……
			check(length(phone_number)=11)
			check(sex in('男', '女'))
			check(email like '%@%')		
给约束指定名字(创建表时)：
        create table student(
	        stuid number(4) constraint PK_id primary key, --PK_id为约束别名
	        name varchar2(20) [constraint 约束名] not null,
	        deleteflog number(1) default 0 check(deleteflog in(0, 1)) --逻辑删除标志
        )
查看约束：
        select constraint_name from user_constraints; --显示当前用户中的约束
        select constraint_name from user_constraints where table_name='SHOP_ORDER';--显示特定表的约束
删除约束：
        alter table 表名 add 约束类型(lie1 lie2)
        alter table 表名 drop constraint 约束名
```
<br>

#### 序列相关操作

```default
用于自动生成一组有顺序的数字。
创建：create sequence 序列名; --默认从1开始，每次递增1
	create sequence 序列名 start with n; --从n开始，递增1
	create sequence 序列名 start with n increment by m; --从n开始，递增m				
	create sequence student_seq start with 4;
删除：drop sequence 序列名	  

使用：序列名.nextval --获取序列中的下一个有效值	  
		insert into students values(student_seq.nextval, '小强', '男', '1234567812232')
	获取序列当前的值
		select seq_students.currval from dual --注意该命令至少要在调用nextval一次后再能用 		  
	查看和当前用户相关的序列和表
		select sequence_name from uer_sequences
		
注意：1.序列中的值被所有的表共用
	2.序列值一旦产生，则不能重新获取
```
<br>

#### 视图相关操作：

```default
起了查询名称的查询语句。	
创建：create view 视图名 as select……
		如create view view_stud as select * from student;
使用：select 列名1,列名2 from 视图名
		如select * from view_stud;
		select sequence_name from uer_sequences
		select table_name from user_tables
		select constraint_name from user_constraints; --显示当前用户中的约束
		select constraint_name from user_constraints where table_name='SHOP_ORDER';--显示特定表的约束
		
注意：1试图只是给SQL起了个别名，简化SQL。
	2不能提高查询效率。
	3通过视图查询出的数据不单独占存储空间。
	4通过视图，可以进行增删改操作，不建议使用，限制非常多。
删除：drop view 视图名
```
<br>

#### 索引相关操作

```default
类似书前面的目录(字典目录)
创建：create index 索引名 on 表名(列名)
	如create index index_student on student(name);
使用：在查询时，只要使用创建了索引的列进行查询，oracle会自动使用索引
	
注意：1索引能提高查询效率，但不是创建的越多越好(占空间、增删改操作时需要维护索引)
	2创建索引的原则：大数据表中查询小部分数据、在经常查询的列上创建索引
	3主键和唯一键oracle自动创建索引
删除：drop index 索引名
```
<br>

#### `查询表`(这是数据库部分最重要的部分)

查询语句的执行顺序<br>

```
sql语句的执行顺序：
	sql语法顺序：select……from……where……group by……order by……
	sql语句执行顺序：
		1.from 确定原始表和原始数据
		2.where 对原始表中的数据进行过滤，符合条件则留下
		3.group by 对符合条件的数据分组
		4.having 对分组后的结果进行过滤，符合条件则留下
		5.select 对符合条件的数据，根据select进行计算结果
		6.order by 对结果排序
```
<br>

基本<br>

```default
1.查询表的结构：
	sql>desc dept;
2.查询所有列：
	sql>select * from dept; --切记查询尽量不要使用*，速度会很慢
							--【注】：sql>set timing on; //打开"显示操作时间的开关" //off为关闭
3.查询指定列：
	sql>select ename, sal, job, depatno from emp;
4.查询(取消重复行)：
	sql>select deptno, job from emp; --这样查询的结果会有很多重复行
	sql>select distinct deptno, job from emp; --查询(没有重复的记录)		
5.查询SMITH所在的部门、工作、薪水
	sql>select deptno, job, sal from emp where ename='SMITH'; 

6.查询使用算数表达式：
	查询显示每位员工的年工资(不包括emp表中的comm奖金)：
		sql>select ename, sal*12 "年工资" from emp;  --""中的内容为sal*12字段的别名,注意为双引号！
													 --【注】别名除了显示之用外，只能在order by中使用！
	查询显示每位员工的年工资(包括emp表中的comm奖金)：
		sql>select ename, sal*12+comm*12 "年工资" from emp; 
			--可是我们发现这样查询出来的有的员工的工资为空，oracle中表达式中有一个值为空，计算表达是就是null
		sql>select ename, sal*12+nvl(comm, 0)*13 "年工资" from emp;--OK
			--nvl(comm, 0) 函数，表示若comm为空则用0替代，若comm不为空则用其本身的值来替代nvl(comm, 0)

7.连接字符串(||)：
	sql>select ename || 'is a' || job  "哈哈" from emp; --"哈哈"为 ename || 'is a' || job 字段的别名
		
8.查找显示工资高于3000的员工姓名：
	sql>select ename, sal from emp where sal>3000;
  查找1982.1.1以后入职的员工:
	sql>select ename, hiredate from emp where hiredate>'1-1月-1982';
  查找显示工资在2000~2500的员工：
	sql>select ename, sal from emp where sal>2000 and sal<2500; --"与"用and，"或"用 or
  
9.like操作符：
	% 表示任意0到多个字符
	_ 表示任意单个字符
	
	查询显示首字符为S的员工姓名和工资：
		sql>select ename, sal from emp where name leike 'S%';
	查询显示第三个字符为大写O的所有员工的姓名和工资：
		sql>select ename, sal from emp where ename linke '__O%';

10.where条件中使用in
	查询显示empno为123,345,800……的雇员的情况：
		sql>select * from emp where empno=123 or empno=345 or empno=800; --速度慢！
		sql>select * from emp where empno in(7521, 7876, 7782);  --速度快！
		
11.使用is null的操作符：
	查询显示没有上级的雇员的情况：
		sql>select * from emp where mgr is null;

12.使用逻辑操作符号：
	查询工资高于500或是员工岗位为MANAGER的雇员，同时还要满足他们的姓名首字母是大写的J:
		sql>select * from emp where (sal>500 or job='MANAGER') and ename like 'J%';

13.使用order by 字句：
	按照工资从低到高的顺序显示雇员的信息：
		sql>select * from emp order by sal [asc]; --asc写不写无所谓
	按照工资从高到低的顺序显示雇员的信息：
		sql>select * from emp order by sal desc;
	按照部门号升序、而雇员的工资降序的顺序显示信息：
		sql>select * from emp order by deptno, sal desc; --注意deptno和sal的顺序不要搞错！
	按照部门号升序、而雇员的入职时间降序的顺序显示信息：
		sql>select * from emp order by deptno, hiredate desc;
	
	使用字段(列)的别名排序： ep：按照年薪降序排序
		sql>select ename, sal*12+nvl(comm, 0)*12 [as] "年薪" from emp order by "年薪" desc; --as可写可不写

14.case-end语句	
		case
			when(条件1) then 语句1
			when(条件2) then 语句2
			else 语句N
		end

		--查询显示工号、姓、工资、工资级别(>20000 A, 10000-20000 B, <10000 C)
			select employee_id, last_name, salary, 
					case
						when (salary > 20000) then 'A'
						when (salary between 10000 and 20000) then 'B' --注意不加 ;
						else 'C'
					end as "级别", department_id
			from employees;
```
<br>

伪列<br>

```default
在表中字段中没有，但是实际上存在的列
rowid：唯一标识表中的一行数据，根据数据在硬盘上的实际存储地址计算得到
	比主键的查询效率高，索引的底层实现就是通过rowid实现的
	
rownum: 对查询结果（*自动）编号，符合条件的第一条数据编号1，再依次排序编号
		主要用在分页中：
		select * from emeployees where rownum<=10; 
		--【注意】rownum 只能进行<、 <= 、=1和>=1 运算
		-- 如果想用rownum进行>运算，可以给rownum起个别名rn，那么rn就变成了普通列，就可以进行>运算了。
表别名：
	--select *, rownum from employees; --Error!
	解决方案：给表起别名。
	--【注】表别名定义时，前面不能加 as
	select ee.*, rownum from employees ee --OK!	
```
<br>

子查询<br>

```default
子查询的概念：
select语句中有嵌套select语句，外面select称为主查询，里层select称为子查询
					 
1.若子查询的结果是一个值，直接代入主查询参与运算
	查询具有最高工资的员工
		select last_name||' '||first_name from employees where salary=(select max(salary) from employees);
	查询工资比平均工资高的员工
		select last_name||' '||first_name from employees where salary>(select avg(salary) from employees);

	查询各部门具有本部门最高工资的员工（自连接或多表查询）
		--方法一：
		select * from employees e where salary=(select max(salary) from employees where department_id=e.department_id group by department_id); --后面的group by也可以不写
		--方法二：
		select ee.* from employees ee, (select department_id, max(salary) maxS from employees group by department_id) ss where ee.department_id=ss.department_id and ee.salary=ss.maxS;

2.若子查询的结果是多个值(N行1列)：把子查询的结果当成集合列表直接参与集合运算符in运算
	查询和姓King的员工同部门的人
		select * from employees where department_id in(select department_id from employees 
		where last_name='King');

3.子查询的结果是虚表(N行N列)：
	查工资排前五名的员工
    select * from (select * from employees order by salary desc) where rownum<=5;
	查询工资排名在第6名到第10名的员工
    select * from (select ee.*, rownum rn from (select ename, sal from emp order by sal) ee 
    where rownum<=10) where rn>=6;	
```
<br>

#### 复杂查询

在实际的应用中经常需要执行复杂的数据统计，经常需要显示多张表的数据，下面介绍较为复杂的select语句。<br>

```default
1.数据分组-max, min, avg, sum, count
	查询显示共有多少个员工：
		sql>select count(*) from emp;
	查询显示所有员工中最高和最低工资：
		sql>select max(sal), min(sal) from emp; --字段(列)中有分组函数，其他字段中必须为分组函数！
	查询显示所有员工中最高工资及其对应的员工：
		sql>select ename, sal from emp where sal=(select max(sal) from emp); --用到子查询
		--sql>select ename, sal from emp where sal=max(sal); //Error！此处不允许用分组函数！
	查询显示工资高于平均工资的员工的信息：
		sql>select * from emp where sal>(select avg(sal) from emp);
	把那些低于平均工资而且入职时间在1980年12月18日以前的人的工资加100：
		sql>update emp set sal=sal-100 where sal<(select avg(sal) from emp) and hiredate<'18-12月-1980';
	
2.group by 和 having 子句
	查询和显示每个部门的平均工资和最高工资
		sql>select avg(sal), max(sal), deptno from emp group by deptno; --【注】被分组字段一定要出现！
	查询和显示每个部门的每个岗位的平均工资和最高工资
		sql>select avg(sal), max(sal), deptno, job from emp group by deptno, job order by deptno;
	查询和显示平均工资低于2000的部门号和他平均工资
		sql>select avg(sal), max(sal), deptno from emp group by deptno having avg(sal)>2000;
		sql>select avg(sal), max(sal), deptno from emp group by deptno having avg(sal)>2000 
		    order by avg(sal) desc;
	【小结】：
	a.分组函数只能出现在选择列表、having、order by 字句中;
	b.如果在select语句中同时包含group by、having、order by，那么他们的顺序是group by、having、order by
	c.在选择列中如果有列、表达式、分组函数，那么这些列和表达式必须有一个出现在group by字句中,否则出错！

3.多表查询：
	多表查询是指基于两个和两个以上的表或视图的查询。在实际的应用中，查询单个表可能不能满足需求(如显示
	sales部门位置和其员工的姓名)，这个时候需要使用到dept表和emp表。
	
	查询显示雇员名、雇员工资及其所在部门的名字【笛卡尔集】：--套路:最后面的子句是为了排除无效的笛卡尔集
		sql>select ee.ename, ee.sal, dd.dname from emp ee, dept dd where ee.deptno=dd.deptno;
															--【注】两张表至少需要1个约束条件
															--【注】三张表至少需要2个约束条件
															--【注】四张表至少需要3个约束条件
															--………………
	查询显示部门编号为10的部门名、员工名和工资：
		sql>select dd.dname, ee.ename, ee.sal from dept dd, emp ee where dd.deptno=ee.deptno 
			and dd.deptno=10; 
	
	查询显示员工的姓名、工资及其工资的级别：
		sql>select ee.ename, ee.sal, ss.grade from emp ee, salgrade ss 
		    where ee.sal between ss.losal and ss.hisal;

	查询显示雇员名、雇员工资及其所在部门的名字，并按照部门编号排序
		sql>select ee.ename, ee.sal, dd.dname from emp ee, dept dd 
		    where ee.deptno=dd.deptno order by ee.deptno;
	
4.自连接：
	指的是同一张表的连接查询
	
	显示某个员工(ep:FORD)的上级领导的姓名：
		sql>select worker.ename, boss.ename from emp worker, emp boss 
		    where worker.empno=boss.empno and worker.ename='FORD';
	
5.子查询：
	子查询是指嵌入在其他sql语句中的select语句，也叫嵌套查询。
		
	单行子查询：只返回一行数据的子查询语句
		查询显示与SMITH同一部门的所有员工
			sql>select ename, deptno from emp where deptno=(select deptno 
				from emp where ename='SMITH'); --数据库执行sql语句是从左到右
				--【注】：在实际中最好能将筛选能力强的语句写在最左边，会大大增加执行速度！
		
	多行子查询：返回多行数据的子查询
		查询显示和部门10中的工作相同的雇员的名字、岗位、工资、部门号
			sql>select * from emp where job in(select job from emp where deptno=10);
		查询和显示工资比部门30的所有员工的工资都高的员工的姓名、工资和部门编号
			sql>select ename, sal, deptno from emp where sal>all(select sal from emp where 
				deptno=30);--方法一，效率低
			sql>select ename, sal, deptno from emp where sal>(select max(sal) from emp where 
				deptno=30);--方法二，效率高，尽可能使用函数解决问题！
	
		在多行子查询中使用any操作符：
			查询显示工资比30号部门的任意的一个员工的工资高的员工的姓名、工资和部门号
				sql>select ename, sal, deptno from emp 
				    where sal>any(select sal from emp where deptno=30); --方法一
				sql>select ename, sal, deptno from emp 
				    where sal>(select min(sal) from emp where deptno=30); --方法二		
					--【注】只要比30号部门的随便一个员工的工资高就可以，即比其中最低工资高就行
					-- 注意与all的区别，all指的是比每一个都怎么怎么样，any是比其中一个怎么怎么样
			
	多列子查询：
		查询返回多个列数据的子查询语句
			查询显示与SMITH的部门和岗位完全相同的所有雇员
				sql>select * from emp 
				    where deptno=(select deptno from emp where ename='SMITH') 
				    and job=(select job from emp where ename='SMITH'); --方法一
				sql>select * from emp 
				    where (deptno, job)=(select deptno, job from emp 
					where ename='SMITH'); --优化方法二
	
	在from子句中使用子查询
		显示高于自己部门的平均工资的员工的信息：
			sql> select * from emp ee, 
			     (select deptno, avg(sal) avgsal from emp group by deptno) ss 
			     where ee.deptno=ss.deptno 
			     and ee.sal>ss.avgsal order by ee.deptno; 
				 --第一步：查询出各个部门的平均工资和部门编号，得到一张子表ss
				 --第二步：利用多表查询(ee, ss)，查询得到正确结果。
				【注】：当在from子句中使用子查询时，该子查询会被作为一个视图来对待，因此称作
						内嵌视图，在from子句中使用子查询时，必须给子查询指定别名。
				【注】：列取别名客加as，表取别名不加as！

	使用子查询插入数据
		当使用values子句时，一次只能插入一行数据，而使用子查询插入数据可以做到一条insert语句可插入大量的数据。当处理行迁移或者装载外部表的数据到数据库时，可以使用子查询来插入数据--先创建一张新表
			sql>create table kkk (myid number(4), myname varchar2(50), mydeptno number(5));
		--再将emp表中的数据导入到新创建的表
			sql>insert into kkk(myid, myname, mydeptno) select empno, ename, deptno from emp 
			    where deptno=10;

	使用子查询更新数据：
		使用update语句更新数据时，既可以使用表达式或者数值直接修改数据，也可以使用子查询修改数据。
			更新员工SCOTT的岗位、工资、补助与SMITH员工一样
			sql>update emp set (job, sal, comm)=(select job, sal, comm from emp 
				where ename='SMITH') where ename='SCOTT';	
```
<br>

#### 分页查询

```default
oracel分页一共有三种方式：
1.rownum分页: --70000条数据0.1秒(常用)
	 第一步:rownum分页,将ee当做一个内嵌视图 
		select ee.*, rownum rn from (select * from emp) ee; 
	 
	 第二步:二分机制来实现取指定的行(记录)出来--ep:取出6-10行记录
		select * from (select ee.*, rownum rn from (select * from emp) ee 
		where rownum<=10) where rn>=6; 
		--【注】rownum一次只能用一下！并且rownum<=和别名rn>=的顺序不能颠倒！
	
	 第三步:指定字段、排序等所有改动都只需要改最里层select就可以了！
		select * from (select ee.*, rownum rn from (select ename, sal from emp order by sal) ee 
		where rownum<=10) where rn>=6;
			
		select * from (select ee.*, rownum rn from (select avg(sal) from emp group by deptno) ee 
		where rownum<=2) where rn>=1;
	
2.rowid分页: --70000条数据0.03秒
		select * from t_xiaoxi where rowid in
			(select rid from 
				(select rownum rn, rid, from 
					(select rowid rid, cid from t_xiaoxi 
						order by cid desc) 
					where rownum<10000) 
			where rn>9980) 
		order by cid desc;
	
3.分析函数分页: --70000条记录1.01秒
			select * from
			    (select t.*, row_number() 
			        over(order by cid desc) rk 
			    from t_xiaoxi t) 
			where rk<10000 and rk>9980;
```
<br>

#### 表连接【开发中的重点】

```default
概念：需要查询的内容来自于多张表，此时需要把多张表合成到一张表，称为表连接。
	合并后的大表表结构(列)：2个表的字段之和
	数据行数：取决于表连接的类型
	
语法：select ???
        rom 表名1
        join 表名2
        on 连接条件
        where……group by……having……order by……

类型：内连接、外连接、交叉连接、自连接	  
	具体想用哪种连接，具体情况具体分析


    1.内连接([inner] join)
       特点：1.内连接的结果——符合(连接)条件的数据
            2.必须要有连接条件
            3.两个表没有顺序要求
            --【注意】表连接后的大表中，若存在同名字段，必须使用表名或是表的别名加以区分
			
			查询员工的工号、姓、工资、所在部门编号（106条）
			    select employee_id, last_name, employees.department_id
			    from employees --可加别名
			    join departments --可加别名
			    on employees.department_id=departments.department_id --连接条件
			    --【注】查不到老板107，他没有部门编号
	
    2.外连接(左外连接left[outer] join、右外连接right[outer] join、全部连接full[outer] join) --优先左外
            左外连接的特点：
            1.左外的结果——符合条件的数据+左表中不符合条件的数据
            2.必须要有连接条件
            3.两个表没有顺序要求，以左表为主！
            4.左表中不符合条件的数据，想显示右表中字段值时，用null填充
				
            查询员工的工号、姓、工资、所在部门编号（107条）
            select employee_id, last_name, employees.department_id
            from employees --可加别名
            left join departments --可加别名
            on employees.department_id=departments.department_id --连接条件
            --【注】查到了老板107，他虽然没有部门编号
            --【注】右外、全部外连接与左外类似
				
            全部连接的特点：
            1.全部的结果——符合条件的数据+左表中不符合条件的数据+右表中不符合条件的数据
            2.必须要有连接条件 
            3.两个表没有顺序要求
            4.不存在的数据，显示另外一张表中字段值时，用null填充--

            查询没有员工的部门
            select d.*, e.*
                from department d
                    left join employees e
                    on e.department_id=d.department_id
                where e.employee_id is null;					
    
    3.多表连接
            select……
                from 表名1
                join 表名2
                on 连接条件1
                join 表名3
                on 连接条件2
                ……
	
    4.交叉连接【了解】
        笛卡尔集，交叉连接后的结果是：两个表的数据行数的乘积 ep:107*27
            select *
                from  employees e
                cross join departments d --效率低
            select * from employees aa, departments bb where……
				
    5.自连接
        特殊的表连接,参与连接的两张表是同一张表
            select * from employees aa left join employees bb on aa.manager_id=bb.employee_id; 

        查询最近的一条记录
        select 
			t1.* 
		from 
			b_t_login t1
		right join
			(select id,max(modify_time) as max_modify_time from b_t_login group by user_number) t2
		on 
			t1.id=t2.id;
```
<br>

#### 用查询结果创建新表

```default
这个命令是一种快捷的创建表的方式
sql>create table newemp(newempno, newname, newsal) as (select empno, ename, sal from emp);
```

#### 合并查询(oracle特有的查询)

```default
特点：速度快！比and、or等的效率快的多

有时候在实际的应用中，为了合并多个select语句的查询结果，
可以使用集合操作符union, union all, intersect, minus

1.union 取得两个结果集的并集(自动去掉结果集中的重复行,相当于集合中运算（A+B-AB）)
  --注union all 不会去掉结果集中的重复行，相当于A+B
  sql>select ename, sal, job from emp where sal>2500 
      union select ename, sal, job from emp where job='MANAGER';

2.intersect 取得两个结果集的交集(相当于集合运算的AB)
  sql>select ename, sal, job from emp where sal>2500 
	  intersect select ename, sal, job from emp where job='MANAGER';

3.minus	取得两个结果集的差集(相当于集合运算的A-B)
  sql>select ename, sal, job from emp where sal>2500 
      minus select ename, sal, job from emp where job='MANAGER';
```
<br>

### 数据库设计

```default
1.什么样的数据库设计是好的？
	1方便企业应用的调用(表的结构清晰、关系清晰)
	2尽量减少数据冗余
	3保证数据的正确性(数据完整性)
			
2.数据库的设计过程：
	需求分析：针对用户的业务数据及处理流程 调研
	概要设计：E-R图
	详细设计：根据E-R，表结构
	编码实现：java+jdbc
	测试：……

3._E-R图：
	实体关系图(entity relationship)：~~~类似于类图
		复杂性名词：对应实体的名称，如学生、电脑	
		简单性名称：对应实体的属性，如学生的姓名、学号……
		
	实体和实体之间的关系：
		1对1
		1对N
		N对N

4.根据E-R图，转换为表结构

	1.先把实体转化为表
		实体名---->表名
		属性------>字段
		主要属性-->主键字段

	2.关系

		【1对1】：如学生借电脑
			  student(sid-PK, name)
				s1	zs
				s2	ls
			  compute(cid-PK, ctype)
				c1	hp
				c2	dell
				
					假设FK放在student表中，看行不行 
					student(sid-PK, name, computeid-FK)
						s1	zs	c1
						s2	ls	c1 --error (原因：站在需求角度，一台电脑只能借给一个学生)
											--解决方案：在FK上添加unique约束
						s2	ls	c2 --OK
					
					假设FK放在compute表中，看行不行
					compute(cid-PK, ctype, stuid-FK)
						c1	hp		s1
						c2	dell	s1--error(原因：站在需求角度，一个学生只能借一台电脑)
										--解决方案：在FK上添加unique约束
						c2	dell	s2 --OK
				--【结论】：如果两个表的关系是1对1，外键可以在任意一方创建，并且同时给FK添加unique约束
						    --通常定义在相对更重要的一方！
		
		【1对N】
			实例：clazz(cid-PK, cname)
					c1	java32班
					c2	java33班
				  student(sid-PK, name)
					s1	zs
					s2	ls
					
					假设，FK设在clazz中
						clazz(cid-PK, cname, stuid-FK)
							c1	java32	s1
							c1	java32	s2 --error(原因：c1违反了PK约束)
										   --没有解决方案，假设不成立
					假设，FK设在student中
						student(sid-PK, name, clzid-FK)
							s1	zs	c1
							s2	ls	c1	--OK
				--【结论】：如果两个表是1对N的关系，外键必须在N的一方创建外键约束
			
		【N对N】
			实例：
				student(sid-PK, name)
					s1	zs
					s2	ls
				course(cid-PK, ctype)
					c1	java
					c2	oracle
				
					 假设FK在student表中
					 student(stuid-PK, name, courseid-FK)
						s1	zs	c1
						s1	zs	c2 --error(原因：s1违反了PK约束)
					 假设FK在course表中
					 course(cid-PK, ctype, stuid-PK)
						c1	java	s1
						c1	java	s2 --error(原因：c1违反了PK约束)
				--【结论】：如果两个表是N对N，必须引入第三张关系表
					第三张关系表：
						sc(sid-FK, cid-FK)
							s1	c1
							s1	c2
							s2	c1
							s2	c2
				--【结论】：把N对N转换为1对N	


5.范式理论
	概念：数据库设计中可以遵循的一组规范，如果遵循了将会给设计人员提供型对较好的指导性意见。
		  优点：减少数据的冗余
		  缺点：导致表的数量增多，影响查询效率
			
	1.【第一范式】：属性的原子性！
		错误的方案：
			学生表sid sanme slike
					1  zs	打球、小说、游戏
					2  ls	睡觉、吃饭、打豆豆
					3  ww   音乐、电影、游戏
					--需求：把兴趣爱好中"游戏"变成"编程"，不好改，违反了第一范式！
		解决方案：把存在费原子性的
			sid slike
			1	打球
			2	小说
			3	游戏
			4	睡觉
			5	吃饭
			6	打豆豆
			引入slike外键
			sid	 sname	slike
	
	2.【第二范式】：属性不允许部分依赖于主属性！（字段不能依赖于主键的一部分）
		学生表	
			sid	 sname	cid	 cname	score
			s1	 zs		c1	 java	90
			s1	 zs		c2	 oracle	89
			s2	 ls		c2	 oracle	99
		PK:sid+cid
					
		--需求：加一门新课
		学生表	
				sid	 sname	cid	 cname	score
				s1	 zs		c1	 java	90
				s1	 zs		c2	 oracle	89	--数据冗余，但查询效率高
				s2	 ls		c2	 oracle	99	
				null null   c3	 test	null --error(PK不能为空)
				s3	 ww		null null	null --error(PK不能为空)
		PK:sid+cid		
		
		解决方案：把存在部分依赖的属性拆分出来，构成新表
			sid sname	--查询效率相对低
			cid cname
			sid cid score

	3.【第三范式】：属性不允许传递依赖于主属性
		学号表：
				sid-PK	sname	 clzid	  clzname ……
				s1		zs		 1		  java32
				s2		ls		 1		  java32
					--需求：新建班
		学号表：
				sid-PK	sname	 clzid	  clzname ……
				s1		zs		 1		  java32
				s2		ls		 1		  java32			
				null	null	 2		  java34 --error(PK不能为null)
		
		解决方案：把传递依赖的属性拆分出来构成新表
				clzid-PK	clzname
				sid-PK	sname	clzid
```
<br>

### jdbc (java database connector)

JDBC的发展<br>

1. java->odbc(open-)->DB (jdbc-odbc桥连接)--不能跨平台、效率低、依赖C语言！
2. java->本地clientAPI->DB (本地客户端API连接) --本地用户
3. java->jdbc->DB (直连)--接口OK，统一标准
4. java->webserver->DB (连接池)--在服务器上做配置就可以
5. oracle[sun]提供一堆jdbc接口规范,数据库厂商以jar包的形式提供接口的jdbc实现类（驱动）

包含的内容<br>

接口-SUN公司提供jdk中 java.sql.\* javax.sql.\* <br>
实现类-数据库厂商提供,在数据库厂商主页下载(ojdbc5.jar适用于jdk5.0,6.0|ojdbc6.jar 适用于 jdk7.0)<br>

简单实用<br>

```java
//六个步骤：
//1.注册驱动类	
//2.创建连接
//3.创建Statement
//4.执行SQL语句
//5.处理执行结果
//6.释放资源

import java.sql.*;
class Test {
	public static void main(String[] args) throws ClassNotFoundException, SQLException {
		//注册驱动类
		Class.forName("oracle.jdbc.driver.OracleDriver");
		//--[注]父包oracle.jdbc中也有OracleDriver类
		//--第二种方式DriverManager.registerDriver(new oracle.jdbc.driver.OracleDriver);
		//--第三种System.setProperty("jdbc.drivers", "oracle.jdbc.driver.OracleDriver");
		//--不建议使用第二种，会产生同名垃圾注册
		
		//--创建连接
		//--url:协议:子协议……:@ip:port:sid 或者 协议:子协议……://ip:port/sid    	
		//--本机ip：localhost 或 127.0.0.1
		//--port：oracle默认端口号为1521
		//--sid:(DB的唯一标识)在服务中可看到
		//--【连接原则】：尽量晚建立，尽量早释放！
		String url = "jdbc:orale:thin:@127.0.0.1:1521:orcl";
		String username = "scott";
		String password = "oracle";
		Connection conn = DriverManager.getConnection(url, username, password);
		
		//--创建Statement
		Statement stm = conn.createStatement();
		
		//--执行SQL语句
		String sql1 = "update student set name='张三', age=23 where stuid=1";
		int row = stm.executeUpdate(str);
		//--执行insert/update/delete语句
		//--有返回值，int，代表影响的行数
		//--【注】存在第三方客户端对服务器数据库执行增删改操作后，需第三方客户端commit之后，才能用jdbc对数据库进行增删改操作
		
		String sql2 = "select * from emp";
		ResultSet rs = sm.executeQuery("select * from emp"); 
		//--执行select语句，返回结果集(虚表)
		while (rs.next()) { //--指针的默认位置在第一行数据的上方，返回boolean
			String name = getString("name");
			int age = getInt(2); //--getXXX的参数可重载，字段名或是字段在返回结果集中的标号
			double score = getDouble(3);
		}
		
		//处理执行结果（只针对select）
		
		//释放资源（先创建的后关闭）
		if (stm!=null)
			stm.close();
		if (conn!=null)
			conn.close();
	}
}	

/*
6.开发常见问题：
1）ClassNotFoundException: oracle.jdbc.OraclDriver 
	解决：检查是否正确导入ojdbc5.jar ； 检查驱动类名是否拼写正确
2）SQLException: IO 错误: The Network Adapter could not establish the connection
	解决：检查ip和port ； 检查数据库服务是否启动，以及oracle监听器是否启动正常	 
3）SQLException: ORA-01017: invalid username/password; logon denied
	解决：检查用户名和密码
4）SQLException: No suitable driver found for jdbc:orle:thin:@127.0.0.1:1521:xe
	解决：检查协议是否正确
5）SQLException: 缺失逗号/引号/无效标示符/序列不存在/表和视图不存在.....
	解决：检查sql命令
*/
```
<br>

事务处理<br>

```default
事务: 用于保证数据的一致性，它由一组相关的dml语句(数据操作语言,增删改语句)组成，
	  该组的dml语句要么全部成功，要么全部失败。
	  如网上转账就是典型的要用事务来处理的例子，可以保证数据的一致性。

事务和锁: 当执行事务操作时(dml语句)，oracle会在被作用的表上加锁(看门狗)，防止其他用户修改表的结构。
		  这点对用户来说是非常重要的。

提交事务: 执行commit语句可以提交事务。当执行了commit语句后，会确认事务的变化、结束事务、删除保存点、释放锁
		  当使用commit语句结束事务后，其他事务可以查看到事物变化后的新数据。
		  --【注】commit;语句执行后，提前做好的保存点(savepoint)统统都不存在了，现在所有事务全部生效。
		  --【注】exit;语句默认自动commit提交。
回退事务: 保存点(savepoint)是事务中的一点，用于取消部分事务，当结束事务时，会自动删除该事务所定义的所有保存
		  点。当执行rollback时，通过指定保存点可以回退到指定的点的状态。
		  --【注】事务的几个重要的动作：
			 --设置保存点：savepoint aa
			 --取消部分事务: rollback to aa;
			 --取消全部事务: rollback
```
<br>

在java程序中使用事务<br>

```java
//在java程序小左数据库时，为了保证数据的一致性，比如转账10$操作：

import java.sql.*;
class Test {
	public static void main(String[] args) {
		Connection ct = null;
		try {
			Class.forName("oracle.jdbc.driver.OracleDriver");	
			String url = "jdbc:oracle:thin:@127.0.0.1:1521:orcl";
			String username = "scott";
			String password = "oracle";
			ct = DriverManager.getConnection(url, username, password);
			
			ct.setAutoCommit(false); //--加入事务处理,设置为不能默认提交
			Statement sm = ct.createStatement();
			//--从SCOTT的工资中减去100
			sm.excuteUpdate("update emp set sal=sal-100 where ename='SCOTT'");
			//--抛个异常
			int i = 2/0;
			//--给SMITH的工资加上100
			sm.excuteUpdate("update emp set sal=sal+100 where ename='SMITH'");	
			ct.commit(); //--提交事务，从ct.setAutoCommit(false)到ct.commit()是一个事务整体,整体提交
			
			//--关闭资源
			sm.close();
			ct.close();	
		} catch (Exception e) {
			//--如果发生了异常,就回滚
			try {
				ct.rollback();
			} catch (Exception ee) {
				ee.printStackTrace();
			}
			e.printStackTrace();
		}
	}
}
```
<br>

只读事务：

```default
只读事务是指只允许执行查询的动作，而不允许执行其他任何的dml操作的事务，使用只读事务可以确保用户只能取得某时间点的数据。
假定机票代售点每天18点开始统计今天的销售情况，这是可以使用只读事务。设置了只读事务后，尽管其他回话可能会提交新的事务，
但是只读事务将不会取得最新的数据的变化，从而可以保证特定时间点的数据信息。
设置只读事务：
    set transaction read only
--设置了只读事务之后，该用户就看不到设置点之后发生的事务结果
```
<br>

### oracle常用sql函数

```default
【字符函数】：
	字符函数式oracle中最常用的函数，常用的有下面的函数：
		lower(char) --将字符串转化为小写的格式
		upper(char) --将字符串转化为大写的格式
		length(char) --返回字符串的长度
		substr(char, offm, lengthn) --取子字符的子串
		replace(char1, search_string, replace_string) --替换指定字符串
		instr(字符串, 字符串) --取子串在字符串中的位置，不存在则返回0
		concat(字符串1, 字符串2)--将1、2字符串连接成一个新的字符串		
		lpad(字段, 总的大小, 填充字符) --左填充(即右对齐)
		rpad(字段, 总的大小, 填充字符) --右填充(即左对齐)
		trim(一个字符 from 字符串) --去掉字符串首尾的字符
		to_char() --将非字符串类型转化成字符串类型--对于日期型可以控制其格式to_char(日期, 格式)
		
		将所有员工的名字按照小写的方式显示
			sql>select lower(ename) from emp;
		将所有员工的名字按照大写的方式显示
			sql>select upper(ename) from emp;
		显示正好为5个字符的员工的姓名
			sql>select ename from length(ename)=5;
		显示所有员工姓名的前三个字符
			sql>select substr(ename, 1, 3) from emp; 
		以首字符大写的方式显示所有员工的姓名
			sql>select (upper(substr(ename, 1, 1))||lower(substr(ename, 2, length(ename)-1))) 
				as "Name" from emp;
		以首字符小写的方式显示所有员工的姓名
			sql>select (lower(substr(ename, 1, 1))||upper(substr(ename, 2, length(ename)-1))) 
				as "Name" from emp;
		显示所有员工的姓名，用"我是A"替换所有的"A"
			sql>select replace(ename, 'A', '我是A') from emp;
		查询显示'SM'在'SMITH'所在字段的索引
			sql>select instr(ename, 'SM') from emp where ename='SMITH';
		连接字符串
			sql>select concat(ename, job) from emp;
		左填充和右填充
			sql>select lpad(sal, 10, '*') from emp;
			sql>select rpad(sal, 10, '*') from emp;
		去掉字符串首尾的字符串
			sql>select trim('S' from ename) from emp;
		
【数学函数】：
	常见的数学函数如下：--【注】在oracle中有一张虚拟的表(dual),一般用来做测试,没有实际含义
		cos、sin、tan、acos、asin、atan、cosh、sinh、tanh
		exp、sqrt、ln、abs
		log(m, n)--以m为底n的对数
		power(m, n)--m的n次方
		ceil(m)、floor(n)--向上、向下取整
		mod(m, n)--取m除以n的余数--sql>select mod(10, 3) from dual;
		round(数字, 从第几位开始四舍五入)--远离0四舍五入, 若没有后面的参数(可正可负)则四舍五入到整数
		trunc(数字, 从第几位开始截取)--若没有后面的参数(可正可负)则截取到整数
		
		查询显示一个月为30天的情况下所有员工的日薪金，忽略忽略余数
		sql>select floor(sal/30) from emp;

【日期函数】：
	常见的日期函数如下：--【注】默认的日期格式是dd-mon-yy，如27-5月-90
		*sysdate --返回oracle服务器系统时间，默认yy-mon-dd格式 如:sql>select sysdate from dual;
			显示昨天这个时候的时期
				sql>select sysdate-1 from dual;--sysdate默认的单位是天		
		*add_months(日期1, 8) --返回从日期1开始再加8个月的时间		
			显示上个月这个时期的时期
				sql>select add_months(sysdate, -1) from dual;
			显示上一年这个时期的时期
				sql>select add_months(sysdate, -12) from dual;
		*trunc(myDate, '日期格式字符串') --截断日期   yyyy mm dd hh mi ss day
			sql>select trunc(sysdate, 'yyyy') from dual; --按年截断，结果是'01-1月'
			sql>select trunc(sysdate, 'mm') from duan;
			sql>select trunc(sysdate, 'day') from duan;	
		*last_day(myDate) --获取date日期的最后一天
			显示一年是不是闰年(看看那一年的2月是28天还是29天)
				sql>select last_day(add_months(trunc(sysdate, 'yyyy'), 1)) from dual;		
		*to_char(myDate, '日期格式字符串') 
			作用1：根据指定格式显示日期
				select to_char(sysdate, 'yyyy-mm-dd hh24:mi:ss day') from dual;
			作用2：获取日期的不同部分
				select to_char(sysdate, 'yyyy') from dual; --获取当前年
				select * from employees where to_char(hire_date, 'yyyy')='1995';
				select * from employees where to_char(sysdate, 'yyyy')= to_char(hire_date, 'yyyy');
				select * from employees where to_char(sysdate, 'mm')= to_char(hire_date, 'mm');
				select to_char(sysdate, 'day') from dual;
		*to_date(myStr, '日期格式字符串')
			将字符串按照指定的格式转换为日期类型
				select to_char(to_date('2008-08-08', 'yyyy-mm-dd'), 'day') from dual;--显示奥运会是星期几
				
				查找8个月多月以前入职的员工
					sql>select * form emp where sysdate>add_months(hiredate, 300);
				返回满10年服务年限的员工的姓名和受雇日期
					sql>select * from emp where sysdate>=add_months(hiredate, 12*10);
				查询显示每个员工加入公司的天数
					sql>select floor(sysdate-hiredate) "入职天数" from emp;
					sql>select trunc(sysdate-hiredate) "入职天数" from emp;
				找出各月份倒数第三天受雇的员工
					sql>select * from emp where last_day(hiredate)-2=hiredate;
				打印本周的星期一时间信息
					sql>select trunc(sysdate, 'day')+1 from dual;
【转换函数】:
	*to_char(myDate, '日期格式字符串') --将日期类型转换为字符串
	*to_char(salary, 'L99,999.99') --将工资显示货币单位--L表示显示本地货币符号，99999.99五位整数，两位小数
								   --9:显示数字，并忽略前面的0
								   --0:显示数字，如位数不够则用0补齐
								   --.:在指定的位置显示小数点
								   --,:在指定的位置显示逗号
								   --$:在数字前(后)加美元
								   --L:在数字前(后)加本地货币符号
								   --C:在数字前(后)加国际货币符号
								   --G:在指定位置显示组分隔符
								   --D:在指定位置显示小数点符号(.)
	*to_date(myStr, '日期格式字符串')
	转换函数用于将数据从一种类型转化为另一种类型。在某些情况下，oracle_sever允许值的数据类型和实际的不一样
	这时候oracle_server会隐含地转化数据类型。
	ep: sql>create table t1(id int);
		sql>insert into t1 values('10'); --这样oracle会自动将'10'-->10
	ep: sql>create table t2(id varchar2(10));
		sql>insert into t2 values(10); --这样oracle会自动将10-->'10'
		--【注】尽管oracle可以隐含进行隐含地数据类型的转换，但是并不适应所有的情况，为了提高数据的可靠性，
			  --我们还是应用转换函数进行转换

【系统函数】：
	使用该函数可以查询系统一些重要信息：
		sys_context:
			terminal：当前回话客户所对应的终端的标识符
			language：语言
			db_name：当前数据库名称
			nls_date_format：当前绘画客户所对应的日期格式
			session_user：当前回话客户所对应的数据库用户名
			current_schema：当前回话客户所对应的数据库方案名(与用户名相同)，
			                一个用户一个方案，oracle以方案的方式管理数据库，
			                方案中有各种数据对象(表、视图、触发器、存储过程……)
			host：返回数据库所在主机的名称
		查询显示正在使用哪个数据库：
			sql>select sys_context('userenv', 'db_name') from dual;
		查询显示正在使用哪种语言：
			sql>select sys_context('userenv', 'language') from dual;
		查询显示当前回话客户所对应的终端的标识符：
			sql>select sys_context('userenv', 'terminal') from dual;
		查询显示数据库所在主机的名称
			sql>select sys_context('userenv', 'host') from dual;
				
【专门处理空数据函数】：
	nvl(comm, 0)--表示若comm为空则用0替代，若comm不为空则用其本身的值来替代nvl(comm, 0)
		sql>select ename, sal*12+nvl(comm, 0)*12 "年工资" from emp;--OK	


【组函数】
	作用于一组数据，有一组数据，函数执行一次。
	--【注】不能使用在where子句中！
		max(字段名)
		min(字段名)
		sum(字段名)
		avg(字段名)
		--【注】这四个组函数执行时自动忽略null值！
			显示所有员工的最大工资
				select max(salary) from employees;
			
		count(字段名) --对该列的非空值统计数量 --不为空计数器加1，为空则计数器不加1
		count(*)		--直接对查询结果统计数量
			统计总的员工的数量
				sql>select count(*) from employees;
				sql>select count(employee_id) from employees; --employee_id是主键(非空, 唯一)
			统计有奖金的员工的个数
				sql>select count(*) from employees where commission_pct is not null;
				sql>select count(commission_pct) from employees;
				
				sql>select count(1) from employees;    --结果为107，因为1不为空
				sql>select count(null) from employees; --结果为0，因为每次都拿null和null作比较
				sql>select count(2) from dual; --结果为1，dual表为1行1列的哑表	

【分组】--需求中出现（各个或者每个）
	语法：select……from……where……group by 分组字段名1, 分组字段名1…… having……order by……
		查询显示各部门的最高工资
			--第一步：确定分组依据
			--第二步：每个小组求最高工资
			sql>select department_id, max(salary) from employees group by department_id;
	
	group by的使用规则
		a.只有在group by中出现的列，才能出现在select中！ --ep:department_id
		b.如果在group by中没有出现的列，那么配合组函数后可以出现在select中  --ep:salary
		c.如果在group by中应用了某些函数，在select和order by中必须使用完全相同的函数 
		--【注】出错：不是group by表达式
		--组函数不能使用在where中！
			查询各岗位的平均工资
				sql>select department_id, round(avg(salary)) 
				from employees group by department_id;
			统计各个部门、各个岗位的人数
				sql>select department_id, job_id, count(*) 
				from employees group by department_id, job_id 	
				order by department_id, job_id;
			查询1997年各月份入职的员工人数
				sql>select to_char(hire_date, 'mm') "月份", count(*) "入职人数" 
				from employees where to_char(hire_date, 'yyyy')='1997' 
				group by to_char(hire_date, 'mm') order by "月份";
			
	having 对分组后的条件过滤，符合条件的留下	
		--where是对原始表条件进行过滤，having是对分组后的数据进行过滤
		--where效率要比having效率高，优先使用where
		--如果使用分组后的结果(通常是组函数)进行过滤，必须使用having
		
			查询平均工资高于8000元的部门的最高工资
				select department_id, avg(salary) 
				from employees 
				group by department_id 
				having avg(salary)>8000 
				order by department_id;
			查询50号部门,60号部门,70号部门的平均工资
				sql>select department_id, avg(salary) 
				from employees 
				where department_id in(50, 60, 70)  
				group by department_id 
				order by department_id; --效率高！
				
				sql>select department_id, avg(salary) 
				from employees 
				group by department_id 
				having department_id in(50, 60, 70)
				order by department_id; --效率低！
```
<br>

### 数据库管理

#### 数据库管理员

```default
每个oracle数据库应该至少有一名数据库管理员(dba)，对于小的数据库，
一个dba就够了，但是对于一个大的数据库可能需要多个dba分担不同的管理职责。

职责：安装和升级oracel数据库
  建库、表空间、表、视图、索引……
  指定并实施备份与恢复计划
  数据库权限管理、调优、排除故障
  高级dba参与项目开发、编写sal语句、存储过程、触发器、规则、约束、包

主要用户：
	sys --董事长
	system --总经理

	区别：
	1.最重要的区别是存储的数据的重要性不同
		sys：所有oracle的数据字典的基表和视图都存放在sys用户中，这些基表和视图对于oracle是至关重要
		的，由数据库自己维护，任何用户都不能手动更改。sys用户拥有dba、sysdba、sysoper角色或权限，
		是oracle权限最高的用户。
		system：用于存放次级的内部数据，如oracle的一些特性或工具的管理信息。system拥有dba，sysdba
		角色或权限。
	2.其次的区别是权限的不同
		sys：必须以as sysdba或者as sysoper形式登录，不能以normal方式登录数据库。
		system：如果正常登录，它其实就是一个普通的dba用户，但如果以as sysdba登录，其结果实际上是作
		为sys用户登录的，从登录信息里面可以看出来。
	
	3.三个角色的权限
        sysdba        >            sysoper          >               dba
        ------------------------------------------------------------------------------------------
        startup--(启动数据库)                        有              可启动实例					
        shutdown--(关闭数据库)                       有              关闭实例
        alter database open/mount/backup --	    有            只有在启动数据库之后才能	
        create database(创建数据库)	--          无            执行各项操作
        drop database(删除数据库) --                 无	
        create spfile --                            有
        alter database archivelog(归档日志) --       有
        alter database recover(恢复数据库) --        只能完全恢复，不能部分恢复
        拥有restricted session(会话限制)权限 --       有	
        可以让用户作为sys用户连接 --                   有一些基本操作，但不能查看用户数据
        登陆后用户是sys	--                          登陆后是public
        ------------------------------------------------------------------------------------------

	4.重要操作：
		显示初始化参数：
			show parameter;
		修改初始化参数：
			到文件D:\oracle\product\10.2.0\admin\orcl\pfile\init.ora文件中去修改实例的名字
		
		数据库(表)备份与恢复：
		--【注】所有导出导入操作需要借助oracle安装目录（D:\oracle\product\10.2.0\db_1\bin）下的
		--exp.exe和imp.exe工具，需要将这两个工具的路径加入到系统环境变量path中。
		--【注】导出导入操作需要在dos窗口中操作
			导出：具体分为导出表、导出方案、导出数据库三种方式
				  都是通过exp命令完成，改命令的常用的选项有：
						userid：用于指定执行导出操作的用户名、口令、连接字符串
						tables：用于指定执行导出操作的表
						owner：用于指定执行导出操作的方案
						full=y：用于指定执行导出操作的数据库
						inctype：用于执行导出操作的增量类型
						rows：用于指定执行导出操作是否要导出表中的数据
						file：用于指定导出文件名
			导出自己的表：
				--在dos下输入命令：
				exp userid=scott/tiger@orcl tables=(emp, dept……) file=d:\e1.dmp
			导出其他方案的表：
				如果用户要导出其他方案的表，则需要dba的权限或exp_full_database的权限，比如
				system就可以导出scott的表：
				exp userid=system/manager@orcl tables=(scott.emp, scott.dept……) file=d:\e1.dmp
			导出表的结构：
				exp userid=scott/tiger@orcl tables=(emp, dept……) file=d:\e1.dmp rows=n
			使用直接导出的方式：
				exp userid=scott/tiger@orcl tables=(emp, dept……) file=d:\e1.dmp direct=y
				--这种方式比默认的常规方式要快，当数据量很大时，可以考虑使用这种方式
				--这是需要数据库的字符集和客户端的字符集完全一致，否则会报错。
			导出自己的方案：
				exp userid=scott/tiger@orcl owner=scott file=d:\e1.dmp [direct=y]
			导出其他方案：
				如果用户要导出其他方案，则需要dba的权限或是exp_full_database的权限，例如
				system用户就可以导出任何方案。
				exp system/manager@orcl owner=(scott) file=d:\e1.dmp [direct=y]
			导出数据库：
				导出数据库是指利用exp导出所有数据库中的对象及数据，要求用户具备dba的权限或
				exp_full_database权限
				exp userid=system/manager@orcl full=y inctype=complete file=e1.dmp [direct=y]
			

			导入：具体分为导入表、导入方案、导入数据库三种方式
				  常用选项：
					userid：用于指定执行导出操作的用户名、口令、连接字符串
					tables：用于指定执行导出操作的表
					formuser：用于指定源用户
					touser：用于指定目标用户
					full=y：用于指定执行导出操作的数据库
					inctype：用于执行导出操作的增量类型
					rows：用于指定执行导出操作是否要导出表中的数据
					file：用于指定导出文件名
					ignore：若表存在，则只导入数据
					
			导入自己的表：
				imp userid=scott/tiger@orcl tables=(emp) file=d:\e1.dmp
				--【注】注意导入的表含有主外键的关系时，如果待导入的方案中没有对应的主外键
				--导入时就会失败
			导入表结构：
				imp userid=scott/tiger@orcl tables=(emp) file=d:\e1.dmp rows=n
			导入数据：
				如果对象(如表)已经存在可以只导入表的数据：
				imp userid=scott/tiger@orcl table=(emp) file=d:\e1.dmp ignore=y
			导入自身的方案：
				import userid=scott/tiger file=d:\e.dmp
			导入其他方案：
				要求该用户具有dba的权限
				imp userid=system/manager file=d:\e.dmp fromuser=system touer=scott
			导入数据库：
				在默认的情况下，会导入所有对象结构和数据。
				imp userid=system/manager full=y file=d:\e1.dmp
```
<br>

#### 数据字典和动态性能视图

```default
数据字典是oracle数据库中最重要的组成部分，它提供了数据库的一些系统信息。
动态性能视图记载了例程启动后的相关信息。

数据字典的组成：
	数据字典基表--存储数据库的基本信息，放在sys方案中，只能查询，维护由系统自动完成，普通用户不能直接访问
	数据字典动态视图--基于数据字典基表所建立的视图，普通用户可通过查询数据字典视图取得系统信息
		user xxx
			如：user_tables:用于显示当前用户所拥有的所有表，它只返回用户所对应方案的所有表
				select table_name from user_tables;
		
		all xxx
			如：all_tables：用于显示当前用户可以访问的所有表。它不仅返回当前用户方案的表，还会返回当前用
							户可以访问的其他方案的表。
				select table_name from all_tables;
		
		dba xxx
			用于显示所有方案拥有的数据库表。要求用户必须是dba角色或是有select any table系统权限。
			如：当用system用户查询数据字典视图dba_tables时，会返回system、sys、scott……方案所对应的数据库表
			
		用户名、权限、角色
		在建立用户时，oracle会把用户的信息存放到数据字典中，当给用户授权或授予角色(当官了?)
		时,oracle会将权限和角色信息存放到数据字典。

		通过查询数据库字典视图 dba_users 可以显示所有数据库用户的详细信息	
		通过查询数据库字典视图 dba_sys_privs 可以显示用户所具有的系统权限
		通过查询数据库字典视图 dba_role_privs 可以显示用户所具有的角色
		通过查询数据库字典视图 dba_tab_privs 可以显示用户所具有的对象权限
		通过查询数据库字典视图 dba_col_privs 可以显示用户所具有的列权限
		desc dba_users; --查询表结构
		select username from dba_users;--显示所有数据库用户的名字
		select * from dba_role_privs where grantee='SCOTT';--显示scott用户拥有的角色信息
		
		如何查询一个角色包含的权限？
			
		Oracle究竟包含多少种角色？
		select * from dba_roles;
```










