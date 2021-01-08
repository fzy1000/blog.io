---
title: MySql小记
date: 2021-01-08 16:00:50
tags:
- MySQL 
category: MySQL
---
# MySql 小记

### 冷知识

1. unique key（唯一键） 和 primary key（主键）的区别：
主键必须是唯一且非空， 唯一键必须是唯一但是可能是空
   
2. 主键（UK）和外键（FK）： 外键是建立在一张二维表上可以引用另一张二维表上对应的主键
3. select count(*) 和 select (1) 是相等的。
是求符合条件的总数，其中包含null。
4. 但是如果是select count（col）就会不计算null的记录。
如果有主键的情况下， select count（主键）最快。
5. 如果使用@Transcation（rollback = Exception.class） 这个注解，记得标注的方法必须是public。否则将不会回滚

### 优化

1. 尽量不要select *
2. 建议使用limit， 防止全表扫描（如果 where 后面跟唯一索引就不再需要limit来提高性能，因为这样本身就是非全表扫描）
3. 尽量避免使用or
   
   例如： select * from user where name = 'allen' or age = '18'

   这种情况下name是唯一索引，但是age不是，这个语句就相当于走了全表扫描+索引扫描+合并，还不如一开始就全表扫描。

   所以可以写成：
   
   select * from user where name = ‘allen’
   
   union all
   
   select * from user where age = '18'

   或者直接分开两条写成两条

4. 优化分页
  
   当偏移量非常大的时候需要优化，例如：

   反例：select * from user limit 10000，10 
  
   正例
   1. 返回上次查询的最大记录（偏移量）
    
      select * from user where id > 10000 limit 10
    
   2. order by + 索引
    
      select * from user order by id limit 10000, 10 
    
   解释：

   mysql并非是跳过偏移量，而是先把偏移量+要取的数查询到，然后再把前面偏移量这一段数据扔掉再返回的。

   方案1可以跳过偏移量，提高效率

   方案2 使用order by 加索引也可以提高效率

5. 优化like语句

   反例： 

   select * from user where name like ‘%123’；

   正例：

   select * from user where name like '123%';

   解释：

   把%放在前面滨步走索引、、放后面还是会走索引

6. 避免在索引列上使用mysql内置函数

   反例：

   select * from user where Date_ADD(create_at, Interval 7 DAY) >= now();

   正例：

   select * from user where create_at >= Date_ADD(NOW(), Interval - 7 DAY)

   在索引列上使用内置函数会导致索引失效，放在后面索引还会走

7. 同理尽量避免在where里对索引进行计算

   反例：

   select * from user where create_at-1 >= now();

   正例：

   select * from user where create_at >= NOW()+1；


8. Inner left right三种优先使用inner， 如果是left，左边表尽量小，如果有条件尽量放在左边处理。

   反例：

   select * from user t1 left join tab2 t2 on t1.name = t2.name where t1.id > 2;

   正例：

   select * from (select * from user where id > 2) t1 left join tab2 t2 on t1.name = t2.name;

   解释：

   条件放左边使左边的表尽量小
9. 尽量避免在where里面使用！= 或者<>。否则会放弃索引进行全表扫描。可以考虑直接分开写成两条。。。

   age！=19 写成

   age>19
 
   age<19

10. 如果有联合索引的时候注意索引列的顺序，一般遵循最左匹配原则。

    假如有一个联合索引idx_userid_age,userid在前，age在后，应该写成：

    select * from user where userid = 1 and age = 10;
    
    select * from user where userid = 1;

    解释：

    当我们创建联合索引的时候（a,b,c）,想到与创建了（a）,(a,b),(a,b,c)三个索引。这就是最左匹配原则

11. 对查询优化，在where和order by涉及的列上建立索引避免全表扫描

    select * from user where userid = 1 order by age;

    alter table user add index idx_userid_age(userid,age)

   
















