# 这简直就是一个“定时炸弹”！记一次 MySQL 的奇葩故障

[https://mp.weixin.qq.com/s/xwQpxikQ2aM8UsacORYoKQ](https://mp.weixin.qq.com/s/xwQpxikQ2aM8UsacORYoKQ)

来源 \| [https://urlify.cn/FJFbuy](https://urlify.cn/FJFbuy)

开发突然紧急的过来说，他们记录无法插入了，有报重复键错误

```text
ERROR 1062 (23000): Duplicate entry '2147483647' for key 'PRIMARY'
```

表名和数据都是采用测试数据，结果和生产的现象是一致的

#### 分析

测试环境为percona server 5.7.20。

首先查看表数据和表结构，结果如下

```text
mysql> select  * from t2 order by id desc limit 3;
+------------+------+------+
| id         | c1   | c2   |
+------------+------+------+
| 2147483647 |  101 |  101 |
|        100 |  100 |  100 |
|          4 |    4 |    4 |
+------------+------+------+
3 rows in set (0.00 sec)

mysql> show create table t2\G
*************************** 1. row ***************************
       Table: t2
Create Table: CREATE TABLE `t2` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `c1` int(11) DEFAULT NULL,
  `c2` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `uniq_c1` (`c1`)
) ENGINE=InnoDB AUTO_INCREMENT=2147483647 DEFAULT CHARSET=utf8mb4
1 row in set (0.00 sec)
```

可以发现，其中id 是有符号的int，已经达到了int的最大值，但是这个表没有相应的时间字段来记录这个id=2147483647 的记录是何时插入或者更新的。

和开发联系问有没有手动执行插入指定ID字段的，开发回复没有。

拿到发送报错的时间点，把时间点之前的binlog 捞了下，都没找到有id=2147483647的插入记录。

查找DDL变更记录，只找到了该表其他字段的变更，刚好在这个报错的时间点之前，但是没有修改auto\_increment的值，一时间陷入了懵逼状态。

为了找到原因，豁出去了。

用了九牛二虎之力，使用二分查找恢复了好多份备份，同时结合binlog ，终于确认这条记录的具体插入时间。

意外的是，我发现这条记录插入的binlog是这样的

```text
insert into t2(id,c1,c2) values(101,101,101)
```

也就是说，插入的时候的id 是101，并不是 2147483647，那又是为啥呢？

继续解析binlog，我发现了新大陆

```text
update t2 set id=4147483647,c1=101,c2=101 where id=101;
```

通过dml平台的日志审计功能，我们找到了对应的开发，发现是开发误操作把主键更新了，然后溢出，id变成了2147483647

此时，表的ddl 的 auto\_increment 还是等于101，并没有变成2147483647。后面的正常业务SQL进行insert产生的id正常产生，因此可以执行成功，直到我们做了一次DDL，加了一个字段，MySQL重新计算了auto\_incremnt的值，变成了2147483647，新插入的SQL的自增值无法继续分配，主键冲突，业务开始报错，才发现了这个定时炸弹。

#### 小结

MySQL如果在指定id 进行插入的时候，如果这个id大于表的自增值，那么MySQL会把表的自增值修改为这个id，并加1，但是如果我们把主键更新成更大的值，MySQL并不会把表的自增值修改为更新后的值，会埋下一颗定时炸弹，在某些情况下，如DDL，重启等之后，业务开始报错，会误认为DDL或者重启导致业务表的插入故障。

该问题在percona 5.6.24 和 percona 5.7.20均有出现，在MySQL 8.0.11 中表现正常。

找到BUG发现2005年就有被提出，因为性能原因以及场景很少没有被修复。

