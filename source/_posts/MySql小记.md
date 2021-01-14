---
title: MySql小记
date: 2021-01-08 16:00:50
tags:
- MySQL 
category: MySQL
---

### 冷知识

1. unique key（唯一键） 和 primary key（主键）的区别：
主键必须是唯一且非空， 唯一键必须是唯一但是可能是空
   
2. 主键（UK）和外键（FK）： 外键是建立在一张二维表上可以引用另一张二维表上对应的主键
3. select count(*) 和 select (1) 是相等的。
是求符合条件的总数，其中包含null。
4. 但是如果是select count（col）就会不计算null的记录。
如果有主键的情况下， select count（主键）最快。
5. 如果使用@Transcation（rollback = Exception.class） 这个注解，记得标注的方法必须是public。否则将不会回滚















