# explain都不懂，还说会SQL调优?

{% embed url="https://mp.weixin.qq.com/s/CHQriFUsG24iP6V3XZt8Lg" %}



来源 \| [https://urlify.cn/rUVBJv](https://urlify.cn/rUVBJv)

mysql中的explain命令可以用来查看sql语句是否使用了索引，用了什么索引，有没有做全表扫描。可以帮助我们优化查询语句。

explain出来的信息有10列，文章主要介绍type、key、Extra这几个字段。

演示中涉及到的表结构如下：

```text
 CREATE TABLE `dept_desc` (
  `dept_no` char(4) NOT NULL,
  `dept_name` varchar(40) NOT NULL,
  `desc` varchar(255) NOT NULL,
  PRIMARY KEY (`dept_no`)
) ENGINE=InnoDB

CREATE TABLE `dept_emp` (
  `emp_no` int(11) NOT NULL,
  `dept_no` char(4) NOT NULL,
  `from_date` date NOT NULL,
  `to_date` date NOT NULL,
  PRIMARY KEY (`emp_no`,`dept_no`),
  KEY `dept_no` (`dept_no`),
  CONSTRAINT `dept_emp_ibfk_1` FOREIGN KEY (`emp_no`) REFERENCES `employees` (`emp_no`) ON DELETE CASCADE,
  CONSTRAINT `dept_emp_ibfk_2` FOREIGN KEY (`dept_no`) REFERENCES `departments` (`dept_no`) ON DELETE CASCADE
) ENGINE=InnoDB

CREATE TABLE `employees` (
  `emp_no` int(11) NOT NULL,
  `birth_date` date NOT NULL,
  `first_name` varchar(14) NOT NULL,
  `last_name` varchar(16) NOT NULL,
  `gender` enum('M','F') NOT NULL,
  `hire_date` date NOT NULL,
  PRIMARY KEY (`emp_no`)
) ENGINE=InnoDB
```

上面的表都是mysql中测试库的表，需要的同学可以自行去下载。官方文档：[https://dev.mysql.com/doc/employee/en/employees-installation.html；github下载地址：https://github.com/datacharmer/test\_db.git](https://dev.mysql.com/doc/employee/en/employees-installation.html；github下载地址：https://github.com/datacharmer/test_db.git)

## key

sql语句实际执行时使用的索引列，有时候mysql可能会选择优化效果不是最好的索引，这时，我们可以在select语句中使用force index\(INDEXNAME\)来强制mysql使用指定索引或使用ignore index\(INDEXNAME\)强制mysql忽略指定索引

## type

访问类型，表示数据库引擎查找表的方式，常见的type类型有：

all,index,range,ref,eq\_ref，const。

### all

全表扫描，表示sql语句会把表中所有表数据全部读取读取扫描一遍。效率最低，我们应尽量避免。

```text
mysql> explain select * from dept_emp;
+----+-------------+----------+------+---------------+------+---------+------+--------+-------+
| id | select_type | table    | type | possible_keys | key  | key_len | ref  | rows   | Extra |
+----+-------------+----------+------+---------------+------+---------+------+--------+-------+
|  1 | SIMPLE      | dept_emp | ALL  | NULL          | NULL | NULL    | NULL | 331570 | NULL  |
+----+-------------+----------+------+---------------+------+---------+------+--------+-------+
```

### index

全索引扫描,表示sql语句将会把整颗二级索引树全部读取扫描一遍，因为二级索引

树的数据量比全表数据量小，所以效率比all高一些。一般查询语句中查询字段为索

引字段，且无where子句时，type会为index。如下，mysql确定使用dept\_no这

个索引，然后扫描整个dept\_no索引树得到结果。

```text
mysql> explain select dept_no  from dept_emp;
+----+-------------+----------+-------+---------------+---------+---------+------+--------+-------------+
| id | select_type | table    | type  | possible_keys | key     | key_len | ref  | rows   | Extra       |
+----+-------------+----------+-------+---------------+---------+---------+------+--------+-------------+
|  1 | SIMPLE      | dept_emp | index | NULL          | dept_no | 4       | NULL | 331570 | Using index |
+----+-------------+----------+-------+---------------+---------+---------+------+--------+-------------+
```

### range

部分索引扫描，当查询为区间查询，且查询字段为索引字段时，这时会根据where条件对索引进行部分扫描。

```text
mysql> explain select * from dept_emp  where emp_no > '7';
+----+-------------+----------+-------+---------------+---------+---------+------+--------+-------------+
| id | select_type | table    | type  | possible_keys | key     | key_len | ref  | rows   | Extra       |
+----+-------------+----------+-------+---------------+---------+---------+------+--------+-------------+
|  1 | SIMPLE      | dept_emp | range | PRIMARY       | PRIMARY | 4       | NULL | 165785 | Using where |
+----+-------------+----------+-------+---------------+---------+---------+------+--------+-------------+
```

### ref

出现于where操作符为‘=’，且where字段为非唯一索引的单表查询或联表查询。

// 单表

```text
mysql> explain select * from dept_emp where dept_no = 'd005';
+----+-------------+----------+------+---------------+---------+---------+-------+--------+-----------------------+
| id | select_type | table    | type | possible_keys | key     | key_len | ref   | rows   | Extra                 |
+----+-------------+----------+------+---------------+---------+---------+-------+--------+-----------------------+
|  1 | SIMPLE      | dept_emp | ref  | dept_no       | dept_no | 4       | const | 145708 | Using index condition |
+----+-------------+----------+------+---------------+---------+---------+-------+--------+-----------------------+
```

// 联表

```text
mysql> explain select * from dept_emp,departments where dept_emp.dept_no = departments.dept_no;
+----+-------------+-------------+-------+---------------+-----------+---------+-------------------------------+------+-------------+
| id | select_type | table       | type  | possible_keys | key       | key_len | ref                           | rows | Extra       |
+----+-------------+-------------+-------+---------------+-----------+---------+-------------------------------+------+-------------+
|  1 | SIMPLE      | departments | index | PRIMARY       | dept_name | 42      | NULL                          |    9 | Using index |
|  1 | SIMPLE      | dept_emp    | ref   | dept_no       | dept_no   | 4       | employees.departments.dept_no |    1 | NULL        |
+----+-------------+-------------+-------+---------------+-----------+---------+-------------------------------+------+-------------+
```

### eq\_ref

出现于where操作符为‘=’，且where字段为唯一索引的联表查询。

```text
mysql> explain select * from departments,dept_desc where departments.dept_name=dept_desc.dept_name;
+----+-------------+-------------+--------+---------------+-----------+---------+-------------------------------+------+-------------+
| id | select_type | table       | type   | possible_keys | key       | key_len | ref                           | rows | Extra       |
+----+-------------+-------------+--------+---------------+-----------+---------+-------------------------------+------+-------------+
|  1 | SIMPLE      | dept_desc   | ALL    | NULL          | NULL      | NULL    | NULL                          |    1 | NULL        |
|  1 | SIMPLE      | departments | eq_ref | dept_name     | dept_name | 42      | employees.dept_desc.dept_name |    1 | Using index |
+----+-------------+-------------+--------+---------------+-----------+---------+-------------------------------+------+-------------+
```

### const

出现于where操作符为‘=’，且where字段为唯一索引的单表查询,此时最多只会匹配到一行。

```text
mysql> explain select * from departments where dept_no = 'd005';
+----+-------------+-------------+-------+---------------+---------+---------+-------+------+-------+
| id | select_type | table       | type  | possible_keys | key     | key_len | ref   | rows | Extra |
+----+-------------+-------------+-------+---------------+---------+---------+-------+------+-------+
|  1 | SIMPLE      | departments | const | PRIMARY       | PRIMARY | 4       | const |    1 | NULL  |
+----+-------------+-------------+-------+---------------+---------+---------+-------+------+-------+
```

综上，单从type字段考虑效率，const &gt; eq\_ref &gt; ref &gt; range &gt; index &gt; all. 注意：我们不能仅仅根据type去判断两条sql的执行速度。例如type为range的查询不一定比type为index的全表查询速度要快，还要看具体的sql。因为type为index时，查询是不需要回表操作的，而type为range时，有可能需要回表操作。如sqlA\("select dept\_no from dept\_emp;"\)和sqlB\("select from\_date from dept\_emp where dept\_no &gt; 'd005';"\),这个时候sqlB根据where条件扫描索引树后，需要回表查询相应的行数据，以获取from\_date的值，而sqlA虽然扫描了整颗索引树，但并不需要回表，所以速度可能会比sqlB更快。

## Extra

extra列会包含一些十分重要的信息，我们可以根据这些信息进行sql优化

using index: sql语句没有where查询条件，使用覆盖索引，不需要回表查询即可拿到结果

using where: 没有使用索引/使用了索引但需要回表查询且没有使用到下推索引

using index && useing where: sql语句有where查询条件，且使用覆盖索引，不需要回表查询即可拿到结果。

Using index condition:使用索引查询，sql语句的where子句查询条件字段均为同一索引字段，且开启索引下推功能，需要回表查询即可拿到结果。

Using index condition && using where:使用索引查询，sql语句的where子句查询条件字段存在非同一索引字段，且开启索引下推功能，需要回表查询即可拿到结果。

using filesort: 当语句中存在order by时，且orderby字段不是索引，这个时候mysql无法利用索引进行排序，只能用排序算法重新进行排序，会额外消耗资源。

Using temporary：建立了临时表来保存中间结果，查询完成之后又要把临时表删除。会很影响性能，需尽快优化。

有时在extra字段中会出现"Impossible WHERE noticed after reading const tables"这种描述。翻看网上资料后，个人发现这是mysql一种很怪的处理方式。

当sql语句满足：

1、根据主键查询或者唯一性索引查询；

2、where操作符为"="时。

在sql语句优化阶段，mysql会先根据查询条件找到相关记录，这样，如果这条数据不存在，实际上就进行了一次全扫描，然后得出一个结论，该数据不在表中。这样对于并发较高的数据库，会加大负载。所以，如果数据不用唯一的话，普通的索引比唯一索引更好用。

