---
title: SQL-optimization
date: 2016-09-27 11:33:10
categories: "DB"
tags: 
     - SQL
---

这几条经验是在网上收集并加上笔者的理解，总结出来的。希望能对大家有所帮助。

<!-- more -->


### 多where，少having
where用来过滤行，having用来过滤组
### 多union all，少union
union删除了重复的行，因此花费了一些时间
### 多Exists，少in
Exists只检查存在性，性能比in强很多，有些朋友不会用Exists，就举个例子。
例：想要得到有电话号码的人的基本信息，table2有冗余信息。
```
select * from table1;--(id,name,age)
select * from table2;--(id,phone)
in：
select * from table1 t1 where t1.id in (select t2.id from table2 t2 where t1.id=t2.id);
Exists：
select * from table1 t1 where Exists (select 1 from table2 t2 where t1.id=t2.id);
```
### 使用绑定变量
Oracle数据库软件会缓存已经执行的sql语句，复用该语句可以减少执行时间。
复用是有条件的，sql语句必须相同
问：怎样算不同？
答：随便什么不同都算不同，不管什么空格啊，大小写什么的，都是不同的
想要复用语句，建议使用PreparedStatement
将语句写成如下形式：
```
insert into XXX(pk_id,column1) values(?,?);
update XXX set column1=? where pk_id=?;
delete from XXX where pk_id=?;
select pk_id,column1 from XXX where pk_id=?;
```
### 少用*
很多朋友很喜欢用*，比如：select * from XXX;
一般来说，并不需要所有的数据，只需要一些，有的仅仅需要1个2个，
拿5W的数据量，10个属性来测试:
(这里的时间指的是PL/SQL Developer显示所有数据的时间)
使用select * from XXX;平均需要20秒，
使用select column1,column2 from XXX;平均需要12秒
### 分页sql
一般的分页sql如下所示：
```
sql1:select * from (select t.*,rownum rn from XXX t)where rn>0 and rn <10;
sql2:select * from (select t.*,rownum rn from XXX t where rownum <10)where rn>0;
```
乍看一下没什么区别，实际上区别很大...125万条数据测试，
sql1平均需要1.25秒(咋这么准呢？ )
sql2平均需要... 0.07秒
原因在于，子查询中，sql2排除了10以外的所有数据
当然了，如果查询最后10条，那效率是一样的
### 能用一句sql，千万别用2句sql

这个我就不多解释了。