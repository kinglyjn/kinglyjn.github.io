---
layout: post
title:  "mysql外键约束的参照操作"
desc: "mysql外键约束的参照操作"
keywords: "mysql外键约束的参照操作,kinglyjn,张庆力"
date: 2012-10-17
categories: [DB]
tags: [DB]
icon: fa-database
---



> cascade：父表删除或者更新，自动删除或更新子表中匹配的行
>
> set null：父表删除或更新行，设置子表中的外键列为null。使用该选项必须保证子表中没有指定not null
>
> restrict：拒绝对父表的删除或更新操作（默认，可以不写）
>
> no action：标准sql的关键字，在mysql中与restrict相同



### 实验1:

```sql
--创建表
create table if not exists t_clazz2 (
	id int unsigned not null auto_increment,
    name varchar(10) not null,
	primary key(id)
) engine=InnoDB default charset=utf8;

create table t_stu2 (
	id int unsigned not null auto_increment,
    name varchar(10) not null,
    clazz_id int unsigned default null,
    primary key(id),
    constraint fk_stu2_clazz2 foreign key(clazz_id) references t_clazz2(id) on delete cascade
) engine=InnoDB default charset=utf8;


--插入数据
insert into t_clazz2(name) values('23班');
insert into t_clazz2(name) values('24班');
insert into t_clazz2(name) values('25班');

insert into t_stu2(name, clazz_id) values('张三', 1); 
insert into t_stu2(name, clazz_id) values('李四', 1); 
insert into t_stu2(name, clazz_id) values('小娟', 1);
insert into t_stu2(name, clazz_id) values('王五', 2);
insert into t_stu2(name, clazz_id) values('赵六', 2); 
insert into t_stu2(name, clazz_id) values('田七', 3); 


--开始实验
--删除班级25班
delete from t_clazz2 where id=3;
--实验结果：删除t_clazz2表中的25班数据之后，子表中的名字为 田七 的数据也被删除了！
```

<br>



### 实验二

```sql
--创建表
create table if not exists t_clazz3 (
	id int unsigned not null auto_increment,
    name varchar(10) not null,
	primary key(id)
) engine=InnoDB default charset=utf8;

create table t_stu3 (
	id int unsigned not null auto_increment,
    name varchar(10) not null,
    clazz_id int unsigned default null,
    primary key(id)
) engine=InnoDB default charset=utf8;

alter table t_stu3 add constraint fk_stu3_clazz3 foreign key(clazz_id) references t_clazz3(id) on delete set null;


--插入数据
insert into t_clazz3(name) values('23班');
insert into t_clazz3(name) values('24班');
insert into t_clazz3(name) values('25班');

insert into t_stu3(name, clazz_id) values('张三', 1); 
insert into t_stu3(name, clazz_id) values('李四', 1); 
insert into t_stu3(name, clazz_id) values('小娟', 1);
insert into t_stu3(name, clazz_id) values('王五', 2);
insert into t_stu3(name, clazz_id) values('赵六', 2); 
insert into t_stu3(name, clazz_id) values('田七', 3); 


--开始实验
--删除班级25班
delete from t_clazz3 where id=3;
--实验结果：删除t_clazz3表中的25班数据之后，子表中的名字为 田七 的数据外键被置为null！
```

<br>



### 实验三

```sql
--创建表
create table if not exists t_clazz4 (
	id int unsigned not null auto_increment,
    name varchar(10) not null,
	primary key(id)
) engine=InnoDB default charset=utf8;

create table t_stu4 (
	id int unsigned not null auto_increment,
    name varchar(10) not null,
    clazz_id int unsigned default null,
    primary key(id)
) engine=InnoDB default charset=utf8;

--alter table t_stu4 drop foreign key fk_stu4_clazz4;
alter table t_stu4 add constraint fk_stu4_clazz4 foreign key(clazz_id) references t_clazz4(id) on delete restrict;


--插入数据
insert into t_clazz4 values(default, '23班');
insert into t_clazz4 values(default, '24班');
insert into t_clazz4 values(default, '25班');

insert into t_stu4 values(default, '张三', 1); 
insert into t_stu4 values(default, '李四', 1); 
insert into t_stu4 values(default, '小娟', 1);
insert into t_stu4 values(default, '王五', 2);
insert into t_stu4 values(default, '赵六', 2); 
insert into t_stu4 values(default, '田七', 3); 


--开始实验
--删除班级25班
delete from t_clazz4 where id=3;
--实验结果：删除t_clazz4表中的25班数据之后，报错信息如下：
18:33:12	delete from t_clazz4 where id=3	Error Code: 1451. Cannot delete or update a parent row: a foreign key constraint fails (`test`.`t_stu4`, CONSTRAINT `fk_stu4_clazz4` FOREIGN KEY (`clazz_id`) REFERENCES `t_clazz4` (`id`))	0.0014 sec
```

<br>



### 总结

实际上，我们在开发过程当中，我们很少来使用物理的外键约束，很多都是使用逻辑的外键约束，因为物理的外键约束只有InnoDB这种引擎才支持，而且维护外键也需要空间和时间。对于小型项目，数据量不是很大，物理外键给我们带来开发上的方便，所以这是后我们使用外键。对于大型项目，数据量非常大，通常需要架设mysql集群，这时候只定义逻辑上的外键而不使用物理上的外键给我们开发带来更多的灵活性。