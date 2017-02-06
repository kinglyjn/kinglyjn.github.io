---
layout: post
title:  "MySQL存储过程"
desc: "MySQL存储过程"
keywords: "MySQL存储过程,存储过程,mysql,kinglyjn"
date: 2012-10-17
categories: [DB]
tags: [mysql,sql]
icon: fa-database
---

### 存储过程简介
 
我们常用的操作数据库语言SQL语句在执行的时候需要要先编译，然后执行，而存储过程（Stored Procedure）是一组为了完成特定功能的SQL语句集，经编译后存储在数据库中，用户通过指定存储过程的名字并给定参数（如果该存储过程带有参数）来调用执行它。<br>

一个存储过程是一个可编程的函数，它在数据库中创建并保存。它可以有SQL语句和一些特殊的控制结构组成。当希望在不同的应用程序或平台上执行相同的函数，或者封装特定功能时，存储过程是非常有用的。数据库中的存储过程可以看做是对编程中面向对象方法的模拟。它允许控制数据的访问方式。<br>

存储过程通常有以下优点：<br>
1. 存储过程增强了SQL语言的功能和灵活性。存储过程可以用流控制语句编写，有很强的灵活性，可以完成复杂的判断和较复杂的运算。
2. 存储过程允许标准组件式编程。存储过程被创建后，可以在程序中被多次调用，而不必重新编写该存储过程的SQL语句。而且数据库专业人员可以随时对存储过程进行修改，对应用程序源代码毫无影响。
3. 存储过程能实现较快的执行速度。如果某一操作包含大量的Transaction-SQL代码或分别被多次执行，那么存储过程要比批处理的执行速度快很多。因为存储过程是预编译的。在首次运行一个存储过程时查询，优化器对其进行分析优化，并且给出最终被存储在系统表中的执行计划。而批处理的Transaction-SQL语句在每次运行时都要进行编译和优化，速度相对要慢一些。
4. 存储过程能过减少网络流量。针对同一个数据库对象的操作（如查询、修改），如果这一操作所涉及的Transaction-SQL语句被组织程存储过程，那么当在客户计算机上调用该存储过程时，网络中传送的只是该调用语句，从而大大增加了网络流量并降低了网络负载。
5. 存储过程可被作为一种安全机制来充分利用。系统管理员通过执行某一存储过程的权限进行限制，能够实现对相应的数据的访问权限的限制，避免了非授权用户对数据的访问，保证了数据的安全。

<br>

### 关于MySQL的存储过程

存储过程是数据库存储的一个重要的功能，但是MySQL在5.0以前并不支持存储过程，这使得MySQL在应用上大打折扣。好在MySQL 5.0终于开始已经支持存储过程，这样即可以大大提高数据库的处理速度，同时也可以提高数据库编程的灵活性。<br>


### MySQL存储过程的创建
 
(1). 格式<br>

```default
MySQL存储过程创建的格式：CREATE PROCEDURE 过程名 ([过程参数[,...]])
[特性 ...] 过程体
```

这里先举个例子：

```default
mysql> DELIMITER //  
mysql> CREATE PROCEDURE proc1(OUT s int)  
    -> BEGIN 
    -> SELECT COUNT(*) INTO s FROM user;  
    -> END 
    -> //  
mysql> DELIMITER ; 
``` 

注：<br>
* 这里需要注意的是DELIMITER //和DELIMITER ;两句，DELIMITER是分割符的意思，因为MySQL默认以";"为分隔符，如果我们没有声明分割符，那么编译器会把存储过程当成SQL语句进行处理，则存储过程的编译过程会报错，所以要事先用DELIMITER关键字申明当前段分隔符，这样MySQL才会将";"当做存储过程中的代码，不会执行这些代码，用完了之后要把分隔符还原。<br>
* 存储过程根据需要可能会有输入、输出、输入输出参数，这里有一个输出参数s，类型是int型，如果有多个参数用","分割开。<br>
* 过程体的开始与结束使用 BEGIN 与 END 进行标识。<br>
这样，我们的一个MySQL存储过程就完成了，是不是很容易呢?看不懂也没关系，接下来，我们详细的讲解。<br><br>
 
(2). 声明分割符<br>
 
其实，关于声明分割符，上面的注解已经写得很清楚，不需要多说，只是稍微要注意一点的是：如果是用MySQL的Administrator管理工具时，可以直接创建，不再需要声明。<br><br>
 
(3). 参数<br>
MySQL存储过程的参数用在存储过程的定义，共有三种参数类型,IN,OUT,INOUT,形式如：<br>

```default
CREATE PROCEDURE([[IN |OUT |INOUT ] 参数名 数据类形...])
```

IN 输入参数: 表示该参数的值必须在调用存储过程时指定，在存储过程中修改该参数的值不能被返回，为默认值<br>
OUT 输出参数: 该值可在存储过程内部被改变，并可返回<br>
INOUT 输入输出参数: 调用时指定，并且可被改变和返回<br><br>

 
### MySQL存储过程的基本函数

(1).字符串类<br>

```default
CHARSET(str) //返回字串字符集
CONCAT (string2 [,... ]) //连接字串
INSTR (string ,substring ) //返回substring首次在string中出现的位置,不存在返回0
LCASE (string2 ) //转换成小写
LEFT (string2 ,length ) //从string2中的左边起取length个字符
LENGTH (string ) //string长度
LOAD_FILE (file_name ) //从文件读取内容
LOCATE (substring , string [,start_position ] ) 同INSTR,但可指定开始位置
LPAD (string2 ,length ,pad ) //重复用pad加在string开头,直到字串长度为length
LTRIM (string2 ) //去除前端空格
REPEAT (string2 ,count ) //重复count次
REPLACE (str ,search_str ,replace_str ) //在str中用replace_str替换search_str
RPAD (string2 ,length ,pad) //在str后用pad补充,直到长度为length
RTRIM (string2 ) //去除后端空格
STRCMP (string1 ,string2 ) //逐字符比较两字串大小,
SUBSTRING (str , position [,length ]) //从str的position开始,取length个字符,
TRIM([[BOTH|LEADING|TRAILING] [padding] FROM]string2) //去除指定位置的指定字符
UCASE (string2 ) //转换成大写
RIGHT(string2,length) //取string2最后length个字符
SPACE(count) //生成count个空格

注：mysql中处理字符串时，默认第一个字符下标为1，即参数position必须大于等于1 
```
 
```sql 
mysql> select substring('abcd',0,2);  
+-----------------------+  
| substring('abcd',0,2) |  
+-----------------------+  
|                       |  
+-----------------------+  
1 row in set (0.00 sec)  
 
mysql> select substring('abcd',1,2);  
+-----------------------+  
| substring('abcd',1,2) |  
+-----------------------+  
|     ab                |  
+-----------------------+  
1 row in set (0.02 sec)  
```
<br> 


(2).数学类

```default
ABS (number2 ) //绝对值
BIN (decimal_number ) //十进制转二进制
CEILING (number2 ) //向上取整
CONV(number2,from_base,to_base) //进制转换
FLOOR (number2 ) //向下取整
FORMAT (number,decimal_places ) //保留小数位数
HEX (DecimalNumber ) //转十六进制
注：HEX()中可传入字符串，则返回其ASC-11码，如HEX('DEF')返回4142143
也可以传入十进制整数，返回其十六进制编码，如HEX(25)返回19
LEAST (number , number2 [,..]) //求最小值
MOD (numerator ,denominator ) //求余
POWER (number ,power ) //求指数
RAND([seed]) //随机数
ROUND (number [,decimals ]) //四舍五入,decimals为小数位数]
SIGN (number2 ) //
```

注：返回类型并非均为整数，如：<br>
(1)默认变为整形值<br>

```sql
mysql> select round(1.23);  
+-------------+  
| round(1.23) |  
+-------------+  
|           1 |  
+-------------+  
1 row in set (0.00 sec)  
 
mysql> select round(1.56);  
+-------------+  
| round(1.56) |  
+-------------+  
|           2 |  
+-------------+  
1 row in set (0.00 sec) 
```

(2)可以设定小数位数，返回浮点型数据<br>

```sql
mysql> select round(1.567,2);  
+----------------+  
| round(1.567,2) |  
+----------------+  
|           1.57 |  
+----------------+  
1 row in set (0.00 sec) 
```
<br>

 
(3).日期时间类<br>

```default
ADDTIME (date2 ,time_interval ) //将time_interval加到date2
CONVERT_TZ (datetime2 ,fromTZ ,toTZ ) //转换时区
CURRENT_DATE ( ) //当前日期
CURRENT_TIME ( ) //当前时间
CURRENT_TIMESTAMP ( ) //当前时间戳
DATE (datetime ) //返回datetime的日期部分
DATE_ADD (date2 , INTERVAL d_value d_type ) //在date2中加上日期或时间
DATE_FORMAT (datetime ,FormatCodes ) //使用formatcodes格式显示datetime
DATE_SUB (date2 , INTERVAL d_value d_type ) //在date2上减去一个时间
DATEDIFF (date1 ,date2 ) //两个日期差
DAY (date) //返回日期的天
DAYNAME (date ) //英文星期
DAYOFWEEK (date ) //星期(1-7) ,1为星期天
DAYOFYEAR (date ) //一年中的第几天
EXTRACT (interval_name FROM date ) //从date中提取日期的指定部分
MAKEDATE (year ,day ) //给出年及年中的第几天,生成日期串
MAKETIME (hour ,minute ,second ) //生成时间串
MONTHNAME (date ) //英文月份名
NOW() //当前时间
SEC_TO_TIME (seconds ) //秒数转成时间
STR_TO_DATE (string ,format ) //字串转成时间,以format格式显示
TIMEDIFF (datetime1 ,datetime2 ) //两个时间差
TIME_TO_SEC (time ) //时间转秒数]
WEEK (date_time [,start_of_week ]) //第几周
YEAR (datetime ) //年份
DAYOFMONTH(datetime) //月的第几天
HOUR(datetime) //小时
LAST_DAY(date) //date的月的最后日期
MICROSECOND(datetime) //微秒
MONTH(datetime) //月
MINUTE(datetime) //分返回符号,正负或0
SQRT(number2) //开平方 
```
<br>


### SQL示例

```sql
create TABLE user (
  id int(11) NOT NULL AUTO_INCREMENT,
  userName varchar(50) DEFAULT NULL,
  userAge int(11) DEFAULT NULL,
  userAddress varchar(200) DEFAULT NULL,
  PRIMARY KEY (id)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8;
insert into user values(1, 'summer', 100, 'Shanghai Pudong');
 
/*
  创建存储过程
  in 参数例子
*/
DROP PROCEDURE IF EXISTS proc_user_demo_in; /*删除存储过程（如果存在）*/
CREATE PROCEDURE proc_user_demo_in(IN p_in INT)
 BEGIN
  SELECT p_in; 
  SET p_in=2; 
  SELECT p_in; 
 END
/*调用*/
/*定义客户端变量（与连接有关，当客户端退出时，变量会被释放）*/
SET @p_in=1; 
CALL proc_user_demo_in(@p_in); /*1*/ /*2*/
SELECT @p_in; /*1*/
SHOW GLOBAL VARIABLES like '%';
/*p_in虽然在存储过程中被修改，但并不影响@p_id的值*/
 
/*
 创建存储过程
 out 例子
*/
drop procedure if exists proc_user_demo_out;
create procedure proc_user_demo_out(out p_out int)
 BEGIN
  select p_out; 
  SET p_out=2; 
  select p_out; 
 END;
SET @p_out=1; 
call proc_user_demo_out(@p_out); /*null*/ /*2*/
SELECT @p_out; /*2*/
/*p_out在存储过程中被修改，影响@p_out的值*/
 
/*
 创建存储过程
 inout 例子
*/
drop procedure if exists proc_user_demo_inout;
create procedure proc_user_demo_inout(INOUT p_inout int)
 BEGIN
  select p_inout;
  set p_inout=2;
  select p_inout;
 END;
SET @p_inout=1;
CALL proc_user_demo_inout(@p_inout); /*1*/ /*2*/
SELECT @p_inout; /*2*/
 

/*在存储过程中使用用户变量*/
drop procedure if exists proc_user_demo_user_variable;
create procedure proc_user_demo_user_variable()
 BEGIN
  select CONCAT(@greeting, ' world!') as 'greeting';
 END;
set @greeting='hello';
call proc_user_demo_user_variable();
 
/*在存储间传递全局范围的用户变量*/
drop procedure if exists proc_user_demo_variable_transport1;
drop procedure if exists proc_user_demo_variable_transport2;
create procedure proc_user_demo_variable_transport1()
 BEGIN
  set @last_variable_transport='p1';
 END;
create procedure proc_user_demo_variable_transport2()
 BEGIN
  select CONCAT('the last_variable_transport is ', @last_variable_transport) as 'last_variable_transport';
 END;
CALL proc_user_demo_variable_transport1();
CALL proc_user_demo_variable_transport2();
/*
注意:
①用户变量名一般以@开头
②滥用用户变量会导致程序难以理解及管理
*/
 

/*在存数过程中使用内部变量*/
drop procedure if exists proc_user_demo_use_inner_variable;
create procedure proc_user_demo_use_inner_variable(IN param INTEGER)
 BEGIN
  DECLARE variable1 char(10) default 'nothing';
  IF param=1 THEN
   SET variable1='birds';
  ELSEIF param=2 THEN
   SET  variable1='beasts';
  END IF;
  INSERT INTO user(userName) values(variable1);
 END;
CALL proc_user_demo_use_inner_variable(3);
 
 
/*查看数据库中的表和存储过程*/
show tables; /*查看数据库的所有表*/
show create TABLE mybatis.user; /*查看某张表的创建详情*/
select * from mysql.proc where db='mybatis'; /*查看数据库存储过列表*/
show create procedure mybatis.proc_user_demo_in; /*查看某个存储过程的创建详情*/
 

/*
存储过程：变量作用域
内部的变量在其作用域范围内享有更高的优先权，当执行到end。变量时，内部变量消失，此时已经在其作用域外，变量不再可见了，应为在存储
过程外再也不能找到这个申明的变量，但是你可以通过out参数或者将其值指派
给会话变量来保存其值。
*/
drop procedure if exists proc_user_demo_variable_scope;
create procedure proc_user_demo_variable_scope()
 BEGIN
  declare x1 varchar(5) default 'outer';
  BEGIN
   declare x1 varchar(5) default 'inner';
   select x1; /*inner*/
  END;
  select x1; /*outer*/
 END;
call proc_user_demo_variable_scope();
 
/*存储过程：条件语句*/
drop procedure if exists proc_user_demo_if_then_else;
create procedure proc_user_demo_if_then_else(IN param INT)
 BEGIN
  declare var int default 0;
  declare maxid int;
  set var=param;
  select max(id) into maxid from user;
  IF var=0 THEN
   insert into user(userName, userAge) values('nothing', var);
  END IF;
  IF var=1 THEN
   update user set userAge=1 where id=maxid;
  ELSE
   update user set userAge=100 where id=maxid;
  END IF;
 END;
call proc_user_demo_if_then_else(0);
 
 
/*存储过程：case语句*/
drop procedure if exists proc_user_demo_case_when;
create procedure proc_user_demo_case_when(in param int)
 BEGIN
  declare var int;
  set var=param;
  CASE var
   WHEN 0 THEN
    insert into user(userName, userAge) values('nothing', 0);
   WHEN 1 THEN
    insert into user(userName, userAge) values('nothing', 1);
   ELSE
    insert into user(userName, userAge) values('nothing', 100);
  END CASE;
 END;
CALL proc_user_demo_case_when(15);
 

/*存储过程：循环语句 while*/
drop procedure if exists proc_user_demo_while;
create procedure proc_user_demo_while()
 BEGIN
  declare var int default 0;
  while var<6 DO
   insert into user(userName, userAge) values(CONCAT('name',var), var);
   set var=var+1;
  END WHILE;
 END;
CALL proc_user_demo_while();
 
/*存储过程：循环语句 repeat*/
/*它在执行操作后检查结果，而while则是执行前进行检查*/
drop procedure if exists proc_user_demo_repeat;
create procedure proc_user_demo_repeat(in repeat_times int)
 BEGIN
  DECLARE var int default 1;
  REPEAT
   insert into user(userName, userAge) values(CONCAT('name',var), var);
   set var=var+1;
  UNTIL var>repeat_times END REPEAT;
 END;
CALL proc_user_demo_repeat(3);
 
/*存储过程：循环语句 loop*/
drop procedure if exists proc_user_demo_loop;
create procedure proc_user_demo_loop(in loop_times int)
 BEGIN
  declare var int default 1;
  LOOP_LABLE:loop
   insert into user(userName, userAge) values(CONCAT('name',var), var);
   set var=var+1;
   IF var>loop_times THEN
    LEAVE LOOP_LABLE;
   END IF;
  END LOOP;
 END;
CALL proc_user_demo_loop(2);
```
<br>


### mysql存储过程和函数的区别

 存储过程是用户定义的一系列sql语句的集合，涉及特定表或其它对象的任务，用户可以调用存储过程，而函数通常是数据库已定义的方法，它接收参数并返回某种类型的值并且不涉及特定用户表。 <br>

 存储过程和函数存在以下几个区别：<br>

1. 一般来说，存储过程实现的功能要复杂一点，而函数的实现的功能针对性比较强。存储过程，功能强大，可以执行包括修改表等一系列数据库操作；用户定义函数不能用于执行一组修改全局数据库状态的操作。
2. 对于存储过程来说可以返回参数，如记录集，而函数只能返回值或者表对象。函数只能返回一个变量；而存储过程可以返回多个。存储过程的参数可以有IN,OUT,INOUT三种类型，而函数只能有IN类~~存储过程声明时不需要返回类型，而函数声明时需要描述返回类型，且函数体中必须包含一个有效的RETURN语句。
3. 存储过程，可以使用非确定函数，不允许在用户定义函数主体中内置非确定函数。 
4. 存储过程一般是作为一个独立的部分来执行（ EXECUTE 语句执行），而函数可以作为查询语句的一个部分来调用（SELECT调用），由于函数可以返回一个表对象，因此它可以在查询语句中位于FROM关键字的后面。 SQL语句中不可用存储过程，而可以使用函数。

示例：<br>

```sql
/*定义函数*/
DROP FUNCTION IF EXISTS fun_user_demo_sum;
CREATE FUNCTION fun_user_demo_sum(a INT(2),b INT(2)) RETURNS INT(4)
 BEGIN
  DECLARE sum_ INT(2) DEFAULT 0;
  SET sum_ = a + b;
  RETURN sum_;
 END;
/*使用函数*/
select fun_user_demo_sum(1,2) as 'sum_';
 
/*定义存储过程*/
drop procedure if exists proc_user_demo_sum;
CREATE PROCEDURE proc_user_demo_sum(IN a INT(2), IN b INT(2), OUT sum INT(4))
 BEGIN
  SET sum=a+b;
 END;
/*使用存储过程*/
call proc_user_demo_sum(1, 2, @c);
select @c; 
```
<br>
