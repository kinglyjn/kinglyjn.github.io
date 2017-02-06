---
layout: post
title:  "MySQL创建表时添加表和列的注释以及查看表和列的注释"
desc: "MySQL创建表时添加表和列的注释以及查看表和列的注释"
keywords: "MySQL创建表时添加表和列的注释以及查看表和列的注释,command,mysql,kinglyjn"
date: 2012-10-17
categories: [DB]
tags: [mysql,sql]
icon: fa-database
---

```sql
create table UserLogOn  (
   UserId               INTEGER                         not null,
   UserSource           VARCHAR2(50),
   OpenId               VARCHAR2(50),
   constraint PK_BASE_USERLOGON primary key (UserId)
);

comment on table Base_UserLogOn is
'系统用户登录信息表';
 
comment on column Base_UserLogOn.UserId is
'主键';
 
comment on column Base_UserLogOn.UserSource is
'用户来源';
 
comment on column Base_UserLogOn.OpenId is
'单点登录标识';
```
<br>

创建表时候添加默认注释：<br>

```sql
CREATE TABLE groups(
  gid INT PRIMARY KEY AUTO_INCREMENT COMMENT '设置主键自增',
  gname VARCHAR(200) COMMENT '列注释',
) COMMENT='表注释'
```
<br>

修改已创建了的表注释：<br>

```sql
ALTER TABLE groups COMMENT '修改表注释'
ALTER TABLE groups MODIFY COLUMN gname VARCHAR(100) COMMENT '修改列注释'
```
<br>

查看表注释<br>

```sql
SHOW  CREATE TABLE communication_group;

或

SELECT table_name,table_comment FROM information_schema.tables  WHERE table_schema = '所在的db' AND table_name ='表名groups';
```
<br>


查看列注释<br>

```sql
SHOW FULL COLUMNS FROM groups;

SHOW FULL COLUMNS FROM tableName WHERE FIELD = 'add_time' OR TYPE LIKE '%date%' -- 查看tableName表中列名是add_time的或类型是date的列

SELECT  column_name, column_comment FROM information_schema.columns WHERE table_schema ='db'  AND table_name = 'groups' 
```
<br>



