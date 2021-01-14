---
title: MySql 优化（一）
date: 2021-01-11 16:09:30
tags:
- MySQL 
category: MySQL
---

1. 尽量不要select *
2. 建议使用limit， 防止全表扫描（如果 where 后面跟唯一索引就不再需要limit来提高性能，因为这样本身就是非全表扫描）
3. 尽量避免使用or

   例如：
   ```
   select * from user where name = 'allen' or age = '18'
   ```
   这种情况下name是唯一索引，但是age不是，这个语句就相当于走了全表扫描+索引扫描+合并，还不如一开始就全表扫描。

   所以可以写成：
   ```
   select * from user where name = ‘allen’
   
   union all
   
   select * from user where age = '18'
   ```
   或者直接分开两条写成两条

4. 优化分页

   当偏移量非常大的时候需要优化，例如：
   ```
   反例：select * from user limit 10000，10 
  
   正例
   1. 返回上次查询的最大记录（偏移量）
    
      select * from user where id > 10000 limit 10
    
   2. order by + 索引
    
      select * from user order by id limit 10000, 10 
   ``` 
   解释：

   mysql并非是跳过偏移量，而是先把偏移量+要取的数查询到，然后再把前面偏移量这一段数据扔掉再返回的。

   方案1可以跳过偏移量，提高效率

   方案2 使用order by 加索引也可以提高效率

5. 优化like语句
   ```
   反例： 

   select * from user where name like ‘%123’；

   正例：

   select * from user where name like '123%';
   ```
   解释：

   把%放在前面滨步走索引、、放后面还是会走索引

6. 避免在索引列上使用mysql内置函数
   ```
   反例：

   select * from user where Date_ADD(create_at, Interval 7 DAY) >= now();

   正例：
   
   select * from user where create_at >= Date_ADD(NOW(), Interval - 7 DAY)
   ```
   解释：

   在索引列上使用内置函数会导致索引失效，放在后面索引还会走

7. 同理尽量避免在where里对索引进行计算
   ```
   反例：

   select * from user where create_at-1 >= now();

   正例：

   select * from user where create_at >= NOW()+1；
   ```

8. Inner left right三种优先使用inner， 如果是left，左边表尽量小，如果有条件尽量放在左边处理。
   ```
   反例：

   select * from user t1 left join tab2 t2 on t1.name = t2.name where t1.id > 2;

   正例：

   select * from (select * from user where id > 2) t1 left join tab2 t2 on t1.name = t2.name;

   ```
   解释：
   条件放左边使左边的表尽量小
9. 尽量避免在where里面使用！= 或者<>。否则会放弃索引进行全表扫描。可以考虑直接分开写成两条。。。

   age！=19 写成

   age>19

   age<19

10. 如果有联合索引的时候注意索引列的顺序，一般遵循最左匹配原则。

    假如有一个联合索引idx_userid_age,userid在前，age在后，应该写成：
    ```
    select * from user where userid = 1 and age = 10;
    
    select * from user where userid = 1;
    ```
    解释：

    当我们创建联合索引的时候（a,b,c）,想到与创建了（a）,(a,b),(a,b,c)三个索引。这就是最左匹配原则

11. 对查询优化，在where和order by涉及的列上建立索引避免全表扫描
    ```
    select * from user where userid = 1 order by age;
    ```
    ```
    alter table user add index idx_userid_age(userid,age)
    ```
12. 不要用forloop插入数据，选择用foreach批量插入数据
13. 适当使用覆盖索引

    覆盖索引是select的数据列只用从索引中就能够取得，不必读取数据行，换句话说查询列要被所建的索引覆盖。

    比如说在文章系统里分页显示的时候，一般的查询是这样的：
    ```
    SELECT id, title, content FROM article ORDER BY created DESC LIMIT 10000, 10;
    ```
    通常这样的查询会把索引建在created字段（其中id是主键），不过当LIMIT偏移很大时，查询效率仍然很低，改变一下查询：
    ```
    SELECT id, title, content FROM article
    INNER JOIN (
    SELECT id FROM article ORDER BY created DESC LIMIT 10000, 10
    ) AS page USING(id)
    ```
    此时，建立复合索引"created, id"（只要建立created索引就可以吧，Innodb是会在辅助索引里面存储主键值的），就可以在子查询里利用上Covering Index，快速定位id，查询效率嗷嗷的。

    解释：
    
    覆盖索引能够使你的sql语句不需要回表，仅仅访问索引就能拿到需要的数据

14. 慎用distinct
   ```
   反例：

   select DISTINCT * from user  

   正例：

   select DISTINCT name from user

   ```

    解释：

    distinct 在查询的字段过多时，数据库引擎就会对数据进行比较过滤，这个比较过滤会占用系统资源和cpu时间
15. 删除冗余和重复索引
    ```
    反例：

    KEY 'idx_userId'('userId')
    KEY 'idx_userId_age'('userId','age')

    正例：

    KEY 'idx_userId_age'('userId','age')

    ```

    解释：

    优化器在优化查询的时候也要逐个考虑索引，多个重复索引会影响性能

16. 如果数据量较大，优化你的修改、删除语句

    避免同时修改或删除过多数据，因为会造成cpu利用率过高，从而影响他人访问数据库
    ```
    反例：
    
    //一次删除10w或者100w数据
    delete from user where id < 100000;

    //采用单一循环
    for(User user : list){
        delete from iser;
    }
    
    正例：

    分批次删除，比如每次500
    delete user where id < 500;
    delete product where id >=500 and id < 1000;

    ```
    
    解释：

    一次删除太多会出现lock wait timeout exceed错误，建议分批操作
17. where 字句中考虑使用默认值代替null
    ```
    反例：

    select * from user where age is not null;

    正例：

    //设置0为默认值
    select * from user where age > 0;

    ```
    
    解释：

    并不是说用了is null 或者 is not null 就会不走索引了，如果mysql优化器发现走索引比不走索引的成本
还要高，那么就会放弃索引。这些条件： 
    ！=， is null, is not null, <> 
    经常被认为让索引失效，其实是因为一般情况下，查询成本高，优化器自动放弃索引的
    如果把null值换成默认值，很多时候让走索引成为可能，同时，表达意思会相对清晰。
    
18. 不要超过5个以上的表链接 
    + 连表越多，编译的时间和开销也就越大。
    + 把表拆开写，可读性高。
    + 如果一定要连好多表，说明设计有问题
    
19. 索引不宜太多，一般5个以内
    + 索引并不是越多越好，索引虽然提高了查询的效率，但是降低了插入和更新的效率。
    + insert 或者 update 时有可能会重建索引， 所以建索引需要慎重考虑， 视具体情况来定。
    + 一个表的索引最好不要超过5个，若太多就要考虑一些索引是否没有存在的必要。
    
20. 索引不适合建在有大量重复数据的字段上，如性别等
    + sql优化器是根据表中数据量来进行查询优化的，如果优化器发现有大量重复数据，Mysql查询优化器推算
    发现不走索引成本更低，那么就会放弃索引
      
21. 尽量是同数字型字段， 若只含数值信息的字段尽量不要设计为字符型

    ```
    反例：

    ‘king_id’ varchar (20) not null comment 'id'

    正例：

    ‘king_id’ int (11) not null comment 'id'

    ```
    
    解释：
    
    相对于数字型字段，字符型会降低查询和连接的性能，并且会增加存储开销

22. 为了提高group by 语句的效率，可以在执行到该语句前，把不需要的记录过滤掉

    ```
    反例：

    select job, avg from employee group by job having job = 'president' or job = 'managent';

    正例：

    select job, avg from employee where job = 'president' or job = 'managent' group by job;


    ```
23. 如果字段类型是字符串， where时一定用括号括起来，否则索引失效
    ```
    反例：

    select * from user where userid = 123;

    正例：

    select * from user where userid = '123';

    ```

    解释：

    反例中不走索引，因为是字符串跟数字的比较，mysql会做隐式类型转换，把他们转换为浮点数再做比较。