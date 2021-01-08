---
title: MySql事务处理
date: 2021-01-08 15:07:47
tags:
- MySQL
category: MySQL
---
# MySQL事务处理
事务(Transaction)是MySQL对SQL用户操作数据库的一种抽象。我们执行的每一条SQL语句都可以被看作是事务。一般来说，我们的MySQL在执行SQL语句时采用自动提交的模式，这就让我感觉不到事务的存在。其实，事务是为了保障多个SQL操作的一致性。MySQL的事务主要用于处理操作量大，复杂度高的数据。

同时，在MySQL中只有InnoDB引擎是支持事务处理的。通过SHOW ENGINES;可以查询到，我们数据库的存储引擎相关的信息。

```markdown
如果想要把一个表的存储引擎切换为InnoDB，我们可以使用SQL语句进行切换。

ALTER TABLE [table_name] ENGINE=InnoDB；
```

## 1. 事务的特性
事务的特性我们一般简称为ACID。已经在上一篇中提到过

## 2. 事务的开启
MySQL默认为自动提交模式，表示事务为未开启状态，开启事务有多种方式。

+ 全局事务开启
``` mysql
SET autocommit = 0;
```
+ SQL语句开启
``` mysql
START TRANSACTION;
BEGIN;
```
例：

全局
``` mysql
SET autocommit = 0;
# Some SQL will execute
COMMIT/ROLLBACK;
SET autocommit = 1;
```

SQL：
``` mysql
BEGIN/START TRANSACTION;
# Some SQL will execute
COMMIT/ROLLBACK;
```

## 3. 事务的分类 

事务一般分为五类：

1. 扁平事务
2. 保存点事务
3. 链式事务
4. 嵌套事务
5. 分布式事务

#### 3.1 扁平事务
扁平事务是在开发过程中最常见的一种事务形式，在这种事务中，所有的操作都在同一层次。
```
BEGIN WORK;
//TODO
COMMIT WORK;
```
扁平事务的特点，就是被事务包裹起来的SQL代码段只有一个状态即要么成功要么失败。
#### 3.2 保存点事务
保存点事务和扁平事务是形成鲜明对比的。保存点事务允许事务在执行过程中回滚到本事务先前的一个状态。
这样就降低了因事务中某个环节出错而导致整个事务回滚的开销。
```
BEGIN WORK;
//TODO 1
SAVEPOINT t1
//TODO 2
SAVEPOINT t2
COMMIT WORK/ROLLBACK WORK/ROLLBACK TO SAVEPOINT t1;
```
#### 3.3链式事务
链式事务是在提交一个事务时，释放不必要的数据对象，同时将必要的上下文隐式传递给下一个事务。
提交事务和下一个事务操作将被视为原子操作，就是下一个事务可以看到上一个事务的执行结果。
不过链式事务在回滚时，只能回滚到就近的上一个保存点。
![链式事务](http://m.qpic.cn/psc?/V51lIF8R3HS9sa4GVlzK1thDGf39F65U/ruAMsa53pVQWN7FLK88i5rj9TxzNzoNEMKztzgZEflWZpJGf4N5TFhFMKdTm4Pa76pHvSNGqLyzKjWq17ksWGpyG4PIqAVQL8mibbYuPtBo!/b&bo=CQN4AQAAAAADB1E!&rf=viewer_4 "链式事务")


#### 3.4嵌套事务

嵌套事务顾名思义便是多个事务嵌套在一起，如下图所示。这种情况下，我们一般把最外层事务称之为顶层事务，一个事务的前驱事务为父事务、后继事务为子事务。

嵌套事务的特点便是，任何一个节点的子事务都可以提交或回滚。但是只有在其父事务提交或回滚后，其事务才能算的上是真正意义上的提交或回滚。由此便可得出结论，只有在顶层事务提交或回滚后，所有的事务才生效。
![嵌套事务](http://a1.qpic.cn/psc?/V51lIF8R3HS9sa4GVlzK1thDGf39F65U/ruAMsa53pVQWN7FLK88i5krnoQNs1ND7YDnJI*fA0BCozaq56OD7rCLTeo9P*AJARCNJNOsXIAwCFhY*SPv6qAAEwt9X2BhuLf.4us91CAc!/c&ek=1&kp=1&pt=0&bo=twL4AQAAAAADF34!&tl=1&vuin=2581170206&tm=1610089200&sce=60-2-2&rf=newphoto&t=5 "嵌套事务") 

#### 3.5分布式事务

分布式事务通常是指在一个分布式环境下运行的扁平事务，因此需要根据数据所在位置访问网络中的不同节点。
在不同的物理地址，通过网络访问，执行不同的事务，这就是分布式事务。


<font color="#dd0000">关于分布式事务的回滚，还需要继续研究</font>

https://www.cnblogs.com/jing99/p/11769093.html

https://www.cnblogs.com/barrywxx/p/8506512.html

## 4.事务的隔离
之所以使用事务，是因为事务具有一定的隔离性。因为我们的MySQL是并发执行的。同一时刻可以有多条SQL语句要被我们的数据库管理系统执行。那么为了让他们之间保持一种相互独立的状态，这就需要事务去完成。

<font color="#dd0000">关于事务的隔离等级，见上一篇文章</font>


[<font color="0000dd">Reference</font>](https://blog.csdn.net/qq_39530754/article/details/82701753)




