---
layout: post
title:  "Mysql之sql开发技巧"
desc: "Mysql之sql开发技巧"
keywords: "Mysql之sql开发技巧,kinglyjn,张庆力"
date: 2012-10-17
categories: [DB]
tags: [DB]
icon: fa-database
---



### 如何进行行转列

> 场景：报表统计、汇总显示

**报表统计示例：**

```sql
create table t_amount(
	create_time date not null,
  	amount float(8, 2) not null default 0
);

insert into t_amount values('2012-10-01', 111422.12), ('2012-10-10', 111424.12), ('2012-10-21', 112321.13);
insert into t_amount values('2012-11-01', 131422.12), ('2012-11-10', 111524.12), ('2012-11-21', 112678.34);
insert into t_amount values('2012-12-02', 1122721.78), ('2012-12-10', 112761.78), ('2012-12-21', 122231.23);


----行转列
select '销售额' as '月份', 
	sum(case month(create_time) when 10 then amount end) as '10月',
    sum(case month(create_time) when 11 then amount end) as '11月',
    sum(case month(create_time) when 12 then amount end) as '12月'
from t_amount;
```

效果如下图所示：

<img src="http://img.blog.csdn.net/20170805152741592?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:70%"/>

<br>



**汇总统计示例：**

```sql
create table stu_01(
	name varchar(10) not null,
    course varchar(20) not null,
    score float(5,2) not null default 0
);

insert into stu_01 values('张三', '语文', 98);
insert into stu_01 values('张三', '数学', 100);
insert into stu_01 values('张三', '英语', 95);
insert into stu_01 values('李四', '语文', 76);
insert into stu_01 values('李四', '数学', 80);
insert into stu_01 values('李四', '英语', 79);
insert into stu_01 values('王五', '语文', 90);
insert into stu_01 values('王五', '数学', 67);
insert into stu_01 values('王五', '英语', 78);
select * from stu_01;

--行转列
select name, max(case course when '语文' then score end) as '语文',
	   max(case course  when '数学' then score end) as '数学',
	   max(case course  when '英语' then score end) as '英语'
from stu_01 group by name;


--行转列(带总分和平均分)
select name, max(case course when '语文' then score end) as '语文',
	   max(case course  when '数学' then score end) as '数学',
	   max(case course  when '英语' then score end) as '英语',
       sum(score) as '总分',
       avg(score) as '平均分'
from stu_01 group by name;
```

效果如下图所示：

<img src="http://img.blog.csdn.net/20170805143813665?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:60%"/>

<br>

<br>



### 如何进进行列转行

> 场景：表格的属性拆分、ETL转换

**使用序列表的方式进行列转行示例：**

```sql
drop table user_01;
create table user_01(
	username varchar(20) not null,
    mobile varchar(100) not null
);
insert into user_01 values('张三', '13889278278,17628278987,19087826678');
insert into user_01 values('李四', '13628798689,13783937637');
insert into user_01 values('王五', '13039889876');


--第一步：生成一个序列表(最大的序列值需要大于上面待拆分mobile字段里面含有元素的最大个数)
create table seq_01(
	seq_num int primary key auto_increment
);
insert into seq_01 values(),(),(),(),(),(),(),(),();
select * from seq_01;

--第二步，将序列表和稍微处理过的带拆分的表进行内连接
select * from seq_01 s inner join(
  	select username, concat(mobile, ',') as mobile, 
  		length(mobile)-length(replace(mobile, ',', '')) +1 as size 
  	from  user_01) u on s.seq_num<=u.size;

--第三步，将内连接后的表进行字符串截取操作得到最终的表
SELECT 
    username,
    REPLACE(SUBSTRING(SUBSTRING_INDEX('13889278278,17628278987,19087826678,',
                    ',',
                    s.seq_num),
            CHAR_LENGTH(SUBSTRING_INDEX('13889278278,17628278987,19087826678,',
                            ',',
                            s.seq_num - 1)) + 1),
        ',',
        '') AS mobile
FROM
    seq_01 s
        INNER JOIN
    (SELECT 
        username,
            CONCAT(mobile, ',') AS mobile,
            LENGTH(mobile) - LENGTH(REPLACE(mobile, ',', '')) + 1 AS size
    FROM
        user_01) u ON s.seq_num <= u.size;
```

效果如下：

<img src="http://img.blog.csdn.net/20170805170712026?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:100%"/>

**使用序列表+case-when+coalesce函数进行列转行示例：**

```sql
create table book_01(
	book_name varchar(255) not null,
    author varchar(255) not null,
    amount int unsigned
);

insert into book_01 values('java in action', 'zhangsan', 20), ('c++ programming', 'lisi', 30), ('python thinking', 'wangwu', 25);


SELECT 
    book_name,
    CASE s.seq_num
        WHEN 1 THEN 'author'
        WHEN 2 THEN 'amount'
    END AS intem,
    COALESCE(CASE s.seq_num
                WHEN 1 THEN author
            END,
            CASE s.seq_num
                WHEN 2 THEN amount
            END) AS item_value
FROM
    seq_01 s
        INNER JOIN
    book_01 b ON s.seq_num <= 2;
```

效果如下：

<img src="http://img.blog.csdn.net/20170805175128300?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:100%"/>

<br>

<br>



### 如何生成唯一序列号

```sql
--存储当天当前订单号表的表
create table order_seq(
  	date_str varchar(8) not null,
	order_sn int unsigned 
);


--订单号生成的存储过程
delimiter //
create procedure gener_order_seq()
begin
	declare v_cnt int unsigned;
    declare v_timestr varchar(8);
    declare rowcount bigint;
    set v_timestr=date_format(now(), '%Y%m%d');
    select round(rand()*100,0)+1 into v_cnt;
    
    start transaction;
		update order_seq set order_sn=order_sn+v_cnt where date_str=v_timestr;
        if row_count()=0 then
			insert into order_seq(date_str, order_sn) values(v_timestr, v_cnt);
		end if;
        select concat(v_timestr, lpad(order_sn, 7, 0)) as order_sn from order_seq where date_str=v_timestr;
    commit;
end //
delimiter ;

--调用存储过程生成订单号(大约1000条/s)
call gener_order_seq;
```

<br>

<br>



### 如何删除重复数据

* 关键点一：如何判断哪些数据重复？
* 关键点二：如何删除重复的数据只保留一条？

```sql
create table user_03(
	id int unsigned not null auto_increment,
	name varchar(10) not null,
    primary key(id)
) engine=InnoDB charset=utf8;
insert into user_03 value(default, '张三'), (default,'李四'),(default,'小娟'),(default,'李四'), (default,'李四'), (default,'王五'),(default,'王五');


--删除重复数据，只保留id最大的那条记录
delete u1 from user_03 u1 left join(
		select name, max(id)  as maxid from user_03 group by name having count(name)>1
) u2 on u1.name=u2.name where u1.id<u2.maxid;
```

<br>



### （备注1）mysql 的数据类型

**数字型**

| **类型**      | **大小**                          | **范围（有符号）**                              | **范围（无符号）**                              | **用途**  |
| ----------- | ------------------------------- | ---------------------------------------- | ---------------------------------------- | ------- |
| TINYINT     | 1 字节                            | (-128，127)                               | (0，255)                                  | 小整数值    |
| SMALLINT    | 2 字节                            | (-32 768，32 767)                         | (0，65 535)                               | 大整数值    |
| MEDIUMINT   | 3 字节                            | (-8 388 608，8 388 607)                   | (0，16 777 215)                           | 大整数值    |
| INT或INTEGER | 4 字节                            | (-2 147 483 648，2 147 483 647)           | (0，4 294 967 295)                        | 大整数值    |
| BIGINT      | 8 字节                            | (-9 233 372 036 854 775 808，9 223 372 036 854 775 807) | (0，18 446 744 073 709 551 615)           | 极大整数值   |
| FLOAT       | 4 字节                            | (-3.402 823 466 E+38，1.175 494 351 E-38)，0，(1.175 494 351 E-38，3.402 823 466 351 E+38) | 0，(1.175 494 351 E-38，3.402 823 466 E+38) | 单精度浮点数值 |
| DOUBLE      | 8 字节                            | (1.797 693 134 862 315 7 E+308，2.225 073 858 507 201 4 E-308)，0，(2.225 073 858 507 201 4 E-308，1.797 693 134 862 315 7 E+308) | 0，(2.225 073 858 507 201 4 E-308，1.797 693 134 862 315 7 E+308) | 双精度浮点数值 |
| DECIMAL     | 对DECIMAL(M,D) ，如果M>D，为M+2否则为D+2 | 依赖于M和D的值                                 | 依赖于M和D的值                                 | 小数值     |

<br>



**字符类型**

| CHAR(M)    | 0-255字节(0=<M=<255)      | 定长字符串，若没有达到定长则补空格  |
| ---------- | ----------------------- | ------------------ |
| VARCHAR(M) | L+1个字节，L<=M且0<=M<=65535 | 变长字符串              |
| TINYBLOB   | L+1个字节，L<2^8            | 不超过 255 个字符的二进制字符串 |
| TINYTEXT   | L+1个字节，L<2^8            | 短文本字符串             |
| BLOB       | L+2个字节，L<2^16           | 二进制形式的长文本数据        |
| TEXT       | L+2个字节，L<2^16           | 长文本数据              |
| MEDIUMBLOB | L+3个字节，L<2^24           | 二进制形式的中等长度文本数据     |
| MEDIUMTEXT | L+3个字节，L<2^24           | 中等长度文本数据           |
| LOGNGBLOB  | L+4个字节，L<2^32           | 二进制形式的极大文本数据       |
| LONGTEXT   | L+4个字节，L<2^32           | 极大文本数据             |

<br>



**枚举集合**

- ENUM （最多65535个成员）     64KB
- SET  （最多64个成员）               64KB

<br>



**时间类型**

| **类型**    | **大小(字节)** | **范围**                                  | **格式**              | **用途**       |
| --------- | ---------- | --------------------------------------- | ------------------- | ------------ |
| DATE      | 3          | 1000-01-01/9999-12-31                   | YYYY-MM-DD          | 日期值          |
| TIME      | 3          | '-838:59:59'/'838:59:59'                | HH:MM:SS            | 时间值或持续时间     |
| YEAR      | 1          | 1901/2155                               | YYYY                | 年份值          |
| DATETIME  | 8          | 1000-01-01 00:00:00/9999-12-31 23:59:59 | YYYY-MM-DD HH:MM:SS | 混合日期和时间值     |
| TIMESTAMP | 8          | 1970-01-01 00:00:00/2037 年某时            | YYYYMMDD HHMMSS     | 混合日期和时间值，时间戳 |

<br>



### （备注2）Mysql 约束

- 主键约束
- 唯一约束
- 非空约束
- 默认约束
- 外键约束

<br>



### （备注3）外键约束的要求

- 父表和子表使用相同的 InnoDB 存储引擎，而且禁止使用临时表
  show engines;

  show variables like '%storage_engine%';

  select table_schema,table_name,table_type,engine from information_schema.tables where table_name='xxx';

  ALTER TABLE xxx ENGINE=InnoDB;

- 外键列和参照列必须具有相似的数据类型（数字的长度或是否有符号位必须相同，而字符长度可以不同）

- 外键列和参照列必须创建索引。如果外键列不存在索引的话，mysql将自动创建索引

<br>

