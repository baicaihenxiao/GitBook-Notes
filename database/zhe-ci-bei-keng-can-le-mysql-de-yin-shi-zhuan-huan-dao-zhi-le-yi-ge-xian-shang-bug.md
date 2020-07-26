https://mp.weixin.qq.com/s/AB2ylqNc5sPFKvW9X_kdyA



某一天，开发问我，为什么针对一个查询会有两条记录，且其中一条记录并不符合条件`select * from tablea where xxno = 170325171202362928;``xxno`为 `170325171202362928` 和 `170325171202362930`的都出现在结果中。

一个等值查询为什么会有另外一个不同值的记录查询出来呢？

我们一起来看看究竟！

### 分析

我们查看该表结构，发现`xxno` 为`varchar` 类型，但是等号右边是一个数值类型，这种情况下MySQL会如何进行处理呢？官方文档如下：https://dev.mysql.com/doc/refman/5.6/en/type-conversion.html

> The following rules describe how conversion occurs for comparison operations: .... 省略一万字 .... In all other cases, the arguments are compared as floating-point (real) numbers.

也就是说，他会将等于号的两边转换成浮点数来做比较。

> Comparisons that use floating-point numbers (or values that are converted to floating-point numbers) are approximate because such numbers are inexact. This might lead to results that appear inconsistent:

如果比较使用了浮点型，那么比较会是近似的，将导致结果看起来不一致，也就是可能导致查询结果错误。

我们测试下刚刚生产的例子：

```
mysql > select '170325171202362928' = 170325171202362930;
+-------------------------------------------+
| '170325171202362928' = 170325171202362930 |
+-------------------------------------------+
|                                         1 |
+-------------------------------------------+
1 row in set (0.00 sec)
```

可以发现，字符串的`'170325171202362928'` 和 数值的`170325171202362930`比较竟然是相等的。我们再看下字符串`'170325171202362928'` 和字符串`'170325171202362930'` 转化为浮点型的结果

```
mysql  > select '170325171202362928'+0.0;
+--------------------------+
| '170325171202362928'+0.0 |
+--------------------------+
|    1.7032517120236294e17 |
+--------------------------+
1 row in set (0.00 sec)

mysql > select '170325171202362930'+0.0;
+--------------------------+
| '170325171202362930'+0.0 |
+--------------------------+
|    1.7032517120236294e17 |
+--------------------------+
1 row in set (0.00 sec)
```

我们发现，将两个不同的字符串转化为浮点数后，结果是一样的，

所以只要是转化为浮点数之后的值是相等的，那么，经过隐式转化后的比较也会相等，我们继续进行测试其他转化为浮点型相等的字符串的结果

```
mysql > select '170325171202362931'+0.0;
+--------------------------+
| '170325171202362931'+0.0 |
+--------------------------+
|    1.7032517120236294e17 |
+--------------------------+
1 row in set (0.00 sec)

mysql > select '170325171202362941'+0.0;
+--------------------------+
| '170325171202362941'+0.0 |
+--------------------------+
|    1.7032517120236294e17 |
+--------------------------+
1 row in set (0.00 sec)
```

字符串`'170325171202362931'`和`'170325171202362941'`转化为浮点型结果一样，我们看下他们和数值的比较结果

```
mysql > select '170325171202362931' = 170325171202362930;
+-------------------------------------------+
| '170325171202362931' = 170325171202362930 |
+-------------------------------------------+
|                                         1 |
+-------------------------------------------+
1 row in set (0.00 sec)

mysql > select '170325171202362941' = 170325171202362930;
+-------------------------------------------+
| '170325171202362941' = 170325171202362930 |
+-------------------------------------------+
|                                         1 |
+-------------------------------------------+
1 row in set (0.00 sec)
```

结果也是符合预期的。

因此，当MySQL遇到字段类型不匹配的时候，会进行各种隐式转化，一定要小心，有可能导致精度丢失。

> For comparisons of a string column with a number, MySQL cannot use an index on the column to look up the value quickly. If str_col is an indexed string column, the index cannot be used when performing the lookup in the following statement:

如果字段是字符型，且上面有索引的话，如果查询条件是用数值来过滤的，那么该SQL将无法利用字段上的索引

```
SELECT * FROM tbl_name WHERE str_col=1;
```

> The reason for this is that there are many different strings that may convert to the value 1, such as '1', ' 1', or '1a'.

我们进行测试

```
mysql > create table tbl_name(id int ,str_col varchar(10),c3 varchar(5),primary key(id),key idx_str(str_col));
Query OK, 0 rows affected (0.02 sec)

mysql  > insert into tbl_name(id,str_col) values(1,'a'),(2,'b');
Query OK, 2 rows affected (0.01 sec)
Records: 2  Duplicates: 0  Warnings: 0

mysql  > insert into tbl_name(id,str_col) values(3,'3c'),(4,'4d');
Query OK, 2 rows affected (0.00 sec)
Records: 2  Duplicates: 0  Warnings: 0

mysql  > desc select * from tbl_name where str_col='a';
+----+-------------+----------+------+---------------+---------+---------+-------+------+--------------------------+
| id | select_type | table    | type | possible_keys | key     | key_len | ref   | rows | Extra                    |
+----+-------------+----------+------+---------------+---------+---------+-------+------+--------------------------+
|  1 | SIMPLE      | tbl_name | ref  | idx_str       | idx_str | 13      | const |    1 | Using where; Using index |
+----+-------------+----------+------+---------------+---------+---------+-------+------+--------------------------+

mysql  > desc select * from tbl_name where str_col=3;
+----+-------------+----------+------+---------------+------+---------+------+------+-------------+
| id | select_type | table    | type | possible_keys | key  | key_len | ref  | rows | Extra       |
+----+-------------+----------+------+---------------+------+---------+------+------+-------------+
|  1 | SIMPLE      | tbl_name | ALL  | idx_str       | NULL | NULL    | NULL |    4 | Using where |
+----+-------------+----------+------+---------------+------+---------+------+------+-------------+
1 row in set (0.00 sec)

mysql [localhost] {msandbox} (test) > select * from tbl_name where str_col=3;
+----+---------+------+
| id | str_col | c1   |
+----+---------+------+
|  3 | 3c      | NULL |
+----+---------+------+
1 row in set, 2 warnings (0.00 sec)
```

同时我们可以看到，我们用数值型的`3`和`str_col`进行比较的时候，他无法利用索引，同时取出来的值也是错误的，

```
mysql  > show warnings;
+---------+------+----------------------------------------+
| Level   | Code | Message                                |
+---------+------+----------------------------------------+
| Warning | 1292 | Truncated incorrect DOUBLE value: '3c' |
| Warning | 1292 | Truncated incorrect DOUBLE value: '4d' |
+---------+------+----------------------------------------+
```

MySQL针对`3c` 和 `4d`这两个值进行了转化，变成了`3`和`4`

### 小结

在数据库中进行查询的时候，不管是Oracle还是MySQL，一定要注意字段类型，杜绝隐式转化，不仅会导致查询缓慢，还会导致结果错误。

来源 | https://urlify.cn/mMBFZz
