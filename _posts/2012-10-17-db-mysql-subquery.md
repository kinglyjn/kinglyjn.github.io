---
layout: post
title:  "mysql子查询、多表更新、多表连接"
desc: "mysql子查询、多表更新、多表连接"
keywords: "mysql子查询、多表更新、多表连接,kinglyjn,张庆力"
date: 2012-10-17
categories: [DB]
tags: [DB]
icon: fa-database
---



### 常见的三类子查询

* 使用比较运算符的子查询 >  < =  >=  <=  <>  !=  <=>  和 any(some)、all关键字
* 使用 in 和 not in 的子查询
* 使用 exists 和 not exists 的子查询

使用比较运算符的子查询示例：

 ```sql
select * from t_goods where price > any(select goods_price from t_goods where goods_cate='超极本');
 ```

使用 in 和 not in 的子查询示例：

```sql
select * from t_stu2 where clazz_id not in (select clazz_id from t_stu2 group by clazz_id having count(clazz_id)<3);
```

使用 exists 和 not exists 的子查询示例：

```sql
SELECT * FROM article WHERE EXISTS (SELECT * FROM user WHERE article.uid = user.uid)
```

<br>



### 多表更新

```sql
--1
create t_goods_cate(
	id int unsigned not null auto_increment,
  	cate_name varchar(40) not null,
  	primary key(id)
);
--2
insert into t_goods_cate 
	select goods_cate from t_goods group by goods_cate;
--3
update t_goods as g inner join t_goods_cate as c on g.goods_cate=c.cate_name 
	set g.goods_cate=c.id;


--缩减成两步
--1
create table if not exists t_goods_cate(
	id int unsigned not null auto_increment,
  	cate_name varchar(40) not null,
  	primary key(id)
) select goods_cate from t_goods group by goods_cate;
--2
update t_goods as g inner join t_goods_cate as c on g.goods_cate=c.cate_name 
	set g.goods_cate=c.id;
```

<br>



### 多表连接

```sql
select s.*, c.* from t_stu s left join t_clazz c on s.clazz_id=c.id;
select s.*, c.* from t_stu s right join t_clazz c on s.clazz_id=c.id;
select s.*, c.* from t_stu s {inner|cross} join t_clazz c on s.clazz_id=c.id;
```

<br>



### 多表删除

```sql
--需求：删除重复记录只保留一条
delete t1 
	from t_goods as t1 left join (
      select goods_id, goods_name
      	from t_goods 
      	group by goods_name 
      	having count(goods_name)>1
   	) as t2 on t1.goods_name=t2.goods_name
	where t1.goods_id>t2.goods_id;
```



