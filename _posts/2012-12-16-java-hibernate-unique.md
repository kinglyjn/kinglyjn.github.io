---
layout: post
title:  "Hibernate唯一约束"
desc: "Hibernate唯一约束"
keywords: "Hibernate唯一约束,unique,hibernate,java,kinglyjn"
date: 2012-12-16
categories: [Java]
tags: [java]
icon: fa-coffee
---

> UNIQUE约束可以防止两个记录在一个特定的列具有相同的值。 Customers表中，例如，你可能要防止两个或两个以上的人具有相同的年龄。

例子:<br>
例如，下面的SQL语句创建一个新的表名为CUSTOMERS，并增加了5列。这里年龄列设置为独一无二的，所以不能有两个记录具有相同的年龄：<br>

```sql
CREATE TABLE CUSTOMERS( 
    ID INT NOT NULL, 
    NAME VARCHAR (20) NOT NULL, 
    ADDRESS CHAR (25), 
    PRIMARY KEY (ID)
);
```

如果已经创建了CUSTOMERS表，然后添加一个UNIQUE约束年龄的列，那么可能会写一个类似下面的语句：<br>

```sql
ALTER TABLE CUSTOMERS MODIFY AGE INT NOT NULL UNIQUE;
```

还可以使用下面的语法，支持命名的约束和多列：<br>

```sql
ALTER TABLE CUSTOMERS ADD CONSTRAINT myUniqueConstraint UNIQUE(AGE, SALARY);
```

删除一个唯一约束：<br>
要删除一个UNIQUE约束，请使用下面的SQL语句：<br>

```sql
ALTER TABLE CUSTOMERS DROP CONSTRAINT myUniqueConstraint;
```

如果使用MySQL，那么可以使用下面的语法：<br>

```sql
ALTER TABLE CUSTOMERS DROP INDEX myUniqueConstraint;
```

hibernate唯一约束示例：<br>

<img src="http://img.blog.csdn.net/20170122173157736?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:80%"/><br>


