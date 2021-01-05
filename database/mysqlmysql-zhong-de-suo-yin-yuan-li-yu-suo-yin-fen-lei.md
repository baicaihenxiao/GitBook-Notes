# 【MySQL】MySQL中的索引原理与索引分类

## [【MySQL】MySQL中的索引原理与索引分类](https://www.cnblogs.com/jojop/p/13951979.html)

## 一、MySQL索引起步[\#](https://www.cnblogs.com/jojop/p/13951979.html#3917359533)

### 1. 索引的概述[\#](https://www.cnblogs.com/jojop/p/13951979.html#1029480609)

MySQL官方对索引的定义为：索引（index）是帮助MySQL高效获取数据的数据结构（有序）。在数据之外，数据库系统还维护着满足特定查找算法的数据结构，这些数据结构以某种方式引用（指向）数据，这样就可以在这些数据结构上实现高级查找算法，这种数据结构就是索引。如下图所示：

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2021/01/05/1614350-20201110080030201-1160829661-225952.png)

左边是数据表，一共有两列七行记录，最左边的`0x07`格式的数据是物理地址（注意逻辑上相邻的记录在磁盘上也并不是一定物理相邻的）。为了加快 Col 2 的查找，可以维护一个右边所示的二叉查找树，每个节点分别包含索引键值和一个指向对应数据的物理地址的指针，这样就可以运用二叉查找快速获取到对应的数据了。

一般来说索引本身也很大，不可能全部存储在内存中，因此索引往往以索引文件的形式存储在磁盘上。建立索引是数据库中用来提高性能的最常用的方式。

### 2. 建立索引的优劣[\#](https://www.cnblogs.com/jojop/p/13951979.html#4065290382)

优势

* 类似于书籍的目录索引，提高数据检索的效率，降低数据库的IO成本；
* 通过索引对数据进行排序，降低数据排序的成本，降低CPU的消耗；

劣势：

* 实际上索引也是一张表，该表中保存了主键与索引字段，并指向实体类的记录，所以索引列也是要占用空间的；
* 虽然索引提高了查询效率，同时也降低了表更新的速度，如对表进行INSERT、UPDATE、DELETE。因为更新表时，MySQL不仅要保存数据，还要保存一下索引列的字段，都会调整因为更新所带来的键值变化后的索引信息。

### 3. 什么时候建立索引[\#](https://www.cnblogs.com/jojop/p/13951979.html#2098727709)

> 索引是应用程序设计和开发的一个重要方面。若索引太多，应用程序的性能可能会受到影响。而索引太少，对查询性能又会产生影响，要找到一个平衡点，这对应用程序的性能至关重要。一些开发人员总是在事后才想起添加索引----我一直认为，这源于一种错误的开发模式。如果知道数据的使用，那么从一开始就应该在需要处添加索引。开发人员往往对数据库的使用停留在应用的层面，比如编写SQL语句、存储过程之类，甚至可能不知道索引的存在，或认为事后让相关DBA加上即可。DBA往往不够了解业务的数据流，而添加索引需要通过监控大量的SQL语句进而从中找到问题，这个步骤所需的时间肯定是远大于初始添加索引所需的时间，并且可能会遗漏一部分的索引。当然索引也并不是越多越好，我曾经遇到过这样一个问题：某台MySQL服务器 iostat 显示磁盘使用率一直处于100%，经过分析后发现是由于开发人员添加了太多的索引，在删除一些不必要的索引之后，磁盘使用率马上下降为20%。可见索引的添加也是非常有技术含量的。

## 二、索引的数据结构[\#](https://www.cnblogs.com/jojop/p/13951979.html#1487200160)

### 1. 索引分类与引擎对索引的支持[\#](https://www.cnblogs.com/jojop/p/13951979.html#1266960912)

索引是在MySQL的存储引擎层中实现的，而不是在服务器层实现的。所以每种存储引擎的索引都不一定完全相同，也不是所有的存储引擎都支持所有的索引类型。MySQL目前提供了以下4中索引：

* B TREE索引：最常见的索引类型，大部分索引都支持B树索引。
* HASH索引：只有Memory引擎支持，使用场景简单。
* R-tree索引（空间索引）：空间索引是MyISAM引擎的一个特殊索引类型，主要用于地理空间数据类型，通常使用较少；
* Full-text（全文索引）：全文索引也是MyISAM的一个特殊索引类型，主要用于全文索引，InnoDB从MySQL 5.6版本开始支持全文索引；

MyISAM、InnoDB、Memory三种存储引擎对各种索引类型的支持：

| 索引 | InnoDB引擎 | MyISAM引擎 | Memory引擎 |
| :--- | :--- | :--- | :--- |
| B TREE索引 | 支持 | 支持 | 支持 |
| HASH索引 | 不支持 | 不支持 | 支持 |
| R-tree索引 | 不支持 | 支持 | 不支持 |
| Full-text | 5.6版本后支持 | 支持 | 不支持 |

我们常说的索引，如果没有特别的索引，都是指B+树（多路搜索树，并不一定是二叉的）结构组织的索引。其中聚集索引、符合索引、前缀索引、唯一索引默认都是使用 B+ tree，统称为索引。

### 2. B Tree结构[\#](https://www.cnblogs.com/jojop/p/13951979.html#3203847386)

B树又叫多路平衡搜索树，一颗m叉的B Tree特性如下：

* 树中每个节点最多包含m个孩子节点；
* 除根节点与叶子节点外，每个节点至少有\[ceil \(m / 2\) \]个孩子节点；
* 若根节点不是叶子节点，则至少有两个孩子；
* 所有的叶子结点都在同一层；
* 每个非叶子结点由n个key与n+1个指针组成，其中 \[ceil \( m / 2 \) -1\] &lt;= n &lt;= m-1；

接下来以5叉B TREE为例，key的数量：公式推导 \[ ceil \( m / 2 \) -1 \] &lt;= n &lt;= m-1。所以 2 &lt;= n &lt;= 4。当 n &gt; 4时，中间节点分裂到父节点，两边节点分裂。

以插入 C N G A H E K Q M 数据为例，演变过程如一下步骤：

（1）插入前四个字母 C N G A，此时根节点刚好满足 n &lt; m-1；

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2021/01/05/1614350-20201110080056693-769125805-225952.png)

（2）继续插入H，n &gt; 4，中间元素G字母向上分裂到新的节点；

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2021/01/05/1614350-20201110080115770-823729244-225953.png)

（3）继续插入E K Q，此时并不需要分裂；

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2021/01/05/1614350-20201110080129132-1328672699-225954.png)

（4）插入M，中间元素M字母向上分裂到父节点中，排在G后面；

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2021/01/05/1614350-20201110080144693-1148354458-225955.png)

到此就构建了一颗简单的B树，B树与二叉搜索树相比，查询效率更高，因为对于相同的数据量来说，B树的层次结构比二叉树小，因此搜索速度快，当然这与磁盘IO有很大联系，等会介绍。

### 3. B+ Tree结构[\#](https://www.cnblogs.com/jojop/p/13951979.html#3740942351)

B+树 是 B树的变种，B+树 与 B树 的区别为：

* n叉B+树最多含有n个key，而 B树最多只能含有n - 1个key；
* B+树的叶子结点保存所有的key信息，依照key的大小顺序排列；
* 所有的非叶子节点都可以看作key的索引部分。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2021/01/05/1614350-20201110080203100-440490606-225956.png)

如上图，就是一颗典型的B+ Tree，一个节点拥有n个子节点，那么就拥有n个key。同时非叶子节点仅具有充当索引的作用，不存放跟记录有关的信息。树的所有叶子节点构成一个有序链表，可以按照key排序依次遍历全部记录，接下来介绍下MySQL中的B+树。

## 三、为什么MySQL使用B+树？[\#](https://www.cnblogs.com/jojop/p/13951979.html#3139646090)

### 1. 磁盘IO与预读[\#](https://www.cnblogs.com/jojop/p/13951979.html#3626003457)

前面提到了磁盘访问，这里简单介绍一下磁盘IO和预读，磁盘读取数据靠的是机械运动，每次读取数据花费的时间可以分为寻道时间、旋转延迟、传输时间三个部分：

寻道时间指的是磁臂移动到指定磁道所需要的时间，主流磁盘一般在5 ms以下；旋转延迟就是我们经常说的磁盘转速磁盘转速，比如一个磁盘7200转/min，表示每分钟能转7200次，也就是说一秒能转120次，旋转延迟就是1/120/2 = 4.17 ms，也就是半圈的时间（这里有两个时间：平均寻道时间，受限于目前的物理水平，大概是5 ms的时间，找到磁道了，还需要找到你数据存在的那个点，寻点时间，这寻点时间的一个平均值就是半圈的时间，这个半圈时间叫做平均延迟时间，那么平均延迟时间加上平均寻道时间就是你找到一个数据所消耗的平均时间，大概 9 ms，其实机械硬盘慢主要是慢在这两个时间上了，当找到数据然后把数据拷贝到内存的时间是非常短暂的，和光速差不多了）；传输时间指的是从磁盘读出或将数据写入磁盘的时间，一般在零点几毫秒，相对于前两个时间可以忽略不计。

那么访问一次磁盘的时间，即一次磁盘IO的时间约等于5+4.17 = 9 ms左右，听起来还挺不错的，但要知道一台500 -MIPS（Million Instructions Per Second）的机器每秒可以执行5亿条指令，因为指令依靠的是电的性质，换句话说执行一次IO的消耗的时间段下cpu可以执行约450万条指令，数据库动辄十万百万乃至千万级数据，每次9毫秒的时间，显然是个灾难，所以我们要想办法降低IO次数。下图是计算机硬件延迟的对比图，供大家参考：

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2021/01/05/1614350-20201110080218818-1146633478-225957.png)

考虑到磁盘IO是非常高昂的操作，计算机操作系统做了一些优化，**当一次IO时，不光把当前磁盘地址的数据，而是把相邻的数据也都读取到内存缓冲区内**，因为局部预读性原理告诉我们，当计算机访问一个地址的数据的时候，与其相邻的数据也会很快被访问到。每一次IO读取的数据我们称之为一页\(page\)。具体一页有多大数据跟操作系统有关，一般为 4k 或 8k，也就是我们读取一页内的数据时候，实际上才发生了一次IO，这个理论对于索引的数据结构设计非常有帮助。

### 2. MySQL中的B+ 树[\#](https://www.cnblogs.com/jojop/p/13951979.html#3453347412)

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2021/01/05/1614350-20201110080241007-546903802-225959.png)

一般来说，索引本身也很大，所以不会存储在内存中，往往是以索引文件的形式存储在磁盘上的，因此索引在查找的过程中也是需要到IO消耗的。

好了，那么我们为了减少磁盘IO的次数，是不是一次读取的内容越多，就越能减少IO的消耗呢？假如我们现在按上图的索引图要查找3这个元素，需要到几次磁盘IO呢？首先将磁盘块1加载到内存中，判断出3小于28，那么就会去锁定p1指针（这个比较过程以及指针的切换是非常快的，这个过程是发生在内存中，相较于IO操作可以忽略不计），将磁盘块2加载到内存中，继续判断，从而将磁盘块4也加载到内存中，此时为叶节点，叶节点存储数据，那么就只需要在当前的块中将3查找出来即可，整个过程只发生了3次磁盘IO，大大地提高了效率。

#### 2.1 为什么是B+树而不是B树？[\#](https://www.cnblogs.com/jojop/p/13951979.html#2935800847)

上面我们提到的B+树所完成的工作，B树也能完成？为什么MySQL中的索引大多使用B+树而不是B树呢？有以下几个原因：

* 首先B+树的空间利用率更高（非叶节点没有data域），可减少IO次数，磁盘读写所耗费的代价更低；
* B+树的查询效率更加地稳定，B树搜索在非叶子节点还是叶子节点结束都有可能，约靠近根节点，查找效率越快；而B+树无论查找的是什么数据，最终都需要从根节点一直走向叶节点，所有查找所经过的次数都是一样的；
* B+树能同时支持随机检索和顺序检索，而B树只适合随机检索，顺序检索的效率比B+树低；
* 增删文件时，B+树的效率更高，因为所有的data都在叶子节点中，而B树删减节点时还需要分裂，中间节点向上等操作；

#### 2.2 那Hash索引呢？[\#](https://www.cnblogs.com/jojop/p/13951979.html#582399486)

Hash索引更容易理解，底层就是Hash表，调用一次hash函数就可以直接确定相应键值，之后进行回表查询实际数据，按理说Hash索引比B+树还高效？为什么不使用Hash索引呢？原因有以下几点：

* Hash索引不支持区间查找，类似`select * form table where age > 10`这种查找，对于Hash来说，XX；
* Hash索引不支持模糊查询，像`JoJoX`和`JoJoA`之间没有关联性，原因在于Hash函数的不可预测；
* Hash索引在等值查询上很快，但是却不稳定，hash索引还有一个重要的问题，hash碰撞，当发生hash碰撞时，某个键值大量重复时，效率变得极差；

## 四、索引分类[\#](https://www.cnblogs.com/jojop/p/13951979.html#3466208762)

> 按表列属性分类，有以下几种索引类型：

### 1. 单值索引（单列索引）[\#](https://www.cnblogs.com/jojop/p/13951979.html#1831094684)

单值索引即一个索引只包含单个列，一个表中可以有多个单列索引；语法如下：

```sql
Copy# 随表一起建立的索引 索引名同 列名(customer_name)
CREATE TABLE customer (
    id INT(10) UNSIGNED  AUTO_INCREMENT,
    customer_no VARCHAR(200),
    customer_name VARCHAR(200),
    PRIMARY KEY(id),
    KEY (customer_name)  
);

# 单独建单值索引：
CREATE INDEX idx_customer_name ON customer(customer_name); 
# 删除索引：
DROP INDEX idx_customer_name ;
```

### 2. 主键索引[\#](https://www.cnblogs.com/jojop/p/13951979.html#2558491513)

```sql
Copy# 随表一起建索引：
# 使用AUTO_INCREMENT关键字的列必须有索引(只要有索引就行)。
CREATE TABLE customer (
    id INT(10) UNSIGNED  AUTO_INCREMENT,
    customer_no VARCHAR(200),
    customer_name VARCHAR(200),
    PRIMARY KEY(id) 
);

CREATE TABLE customer2 (
    id INT(10) UNSIGNED,
    customer_no VARCHAR(200),
    customer_name VARCHAR(200),
    PRIMARY KEY(id) 
);

# 单独建主键索引：
ALTER TABLE customer add PRIMARY KEY customer(customer_no);  
# 删除建主键索引：
ALTER TABLE customer drop PRIMARY KEY ;  
# 修改建主键索引：
#必须先删除掉(drop)原索引，再新建(add)索引
```

### 3. 唯一索引[\#](https://www.cnblogs.com/jojop/p/13951979.html#3663898392)

索引列的值必须唯一，但允许有空值，语法如下：

```sql
Copy# 随表一起建索引：
#建立 唯一索引时必须保证所有的值是唯一的（除了null），若有重复数据，会报错：
CREATE TABLE customer (
    id INT(10) UNSIGNED  AUTO_INCREMENT,
    customer_no VARCHAR(200),
    customer_name VARCHAR(200),
    PRIMARY KEY(id),
    KEY (customer_name),
    UNIQUE (customer_no)
);

# 单独建唯一索引：
CREATE UNIQUE INDEX idx_customer_no ON customer(customer_no);

# 删除索引：
DROP INDEX idx_customer_no on customer ;
```

### 4. 复合索引（联合索引）[\#](https://www.cnblogs.com/jojop/p/13951979.html#3475149205)

复合索引即一个索引中包含多个列，在数据库操作期间，复合索引所需要的开销更小（相对于相同的多个列建立单值索引）；

```sql
Copy# 随表一起建索引：
CREATE TABLE customer (
    id INT(10) UNSIGNED  AUTO_INCREMENT,
    customer_no VARCHAR(200),
    customer_name VARCHAR(200),
    PRIMARY KEY(id),
    KEY (customer_name),
    UNIQUE (customer_name),
    KEY (customer_no,customer_name)
);

# 单独建索引：
CREATE INDEX idx_no_name ON customer(customer_no,customer_name); 
# 删除索引：
DROP INDEX idx_no_name on customer ;
```

> 接下来介绍以下最左前缀匹配域覆盖索引：

### 5. 最左前缀匹配[\#](https://www.cnblogs.com/jojop/p/13951979.html#3672402414)

认识了单值索引和复合索引，接下来了解最左前缀匹配（leftmost prefix）就容易得多了。什么是最左前缀匹配呢？

假设现在有三个字段建立了复合索引`CREATE INDEX idx_a_b_c ON demo_table(a, b, c);`那么但你的where条件是a、a、b或者啊a、b、c时，都可以命中索引，除此之外都不能命中索引，例如：a、c或者b、c等；

当然有一个例外，当你 select 的字段里有复合索引里的字段，那么where语句不需要满足最左前缀匹配，MySQL 也会走索引。 比如：`select a from demo_table where b = "xxx";` 不过这时走索引不是为了加速查询（这时候索引对查询效率提升效果几乎没有），而是为了利用下面要讲的，覆盖索引，来减少对数据的检索。

### 6. 覆盖索引[\#](https://www.cnblogs.com/jojop/p/13951979.html#1548298485)

覆盖索引（covering index）的原理很简单，就像你拿到了一本书的目录，里头有标题和对应的页码，当你想知道第267页的标题是什么的时候，完全没有必要翻到267页去看，而是直接看目录。 同理，当你要select的字段，已经在索引树里面存储，那就不需要再去检索数据库，直接拿来用就行了。

还是上面的例子，你给a、b、c三个字段建了复合索引，那么对于下面这条sql，就可以走覆盖索引:`select b, c from demo_table where a = "xxx";`

explain一下，你就会发现extra字段是“Using index”，或者使用explain FORMAT=JSON … ，输出一个json结果的结果，看“using\_index”属性，你会发现是“true”，这都意味着使用到了覆盖索引。

```sql
CopyUsing index (JSON property: using_index)：
The column information is retrieved from the table using only information in the index tree without having to do an additional seek to read the actual row.
```

> 按存储结构分类，有以及几种索引：

### 7. 聚簇索引（聚集索引）[\#](https://www.cnblogs.com/jojop/p/13951979.html#3072429284)

聚簇索引就是将数据存储与索引放到了一块，找到索引也就找到了数据。

InnoDB的聚簇索引实际上是在同一个BTree结构中同时存储了索引和整行数据，通过该索引查询可以直接获取查询数据行。

聚簇索引不是一种单独的索引类型，而是一种数据的存储方式，聚簇索引的顺序，就是数据在硬盘上的物理顺序。

在 MySQL 通常聚簇索引是主键的同义词，每张表只包含一个聚簇索引\(其他数据库不一定\)。

### 8. 辅助索引（二级索引）[\#](https://www.cnblogs.com/jojop/p/13951979.html#1988033759)

非聚簇索引就是将数据存储于索引分开结构，索引结构的叶子节点指向了数据的对应行，MyISAM通过key\_buffer把索引先缓存到内存中，当需要访问数据时（通过索引访问数据），在内存中直接搜索索引，然后通过索引找到磁盘相应数据（这一步称为回表查询），这也就是为什么索引不在key buffer命中时，速度慢的原因。

## 五、索引设计原则[\#](https://www.cnblogs.com/jojop/p/13951979.html#3166155250)

索引的设计可以遵循一些已有的原则，创建索引的时候尽量考虑是否符合这些原则，便于提升索引的使用效率，更高效地使用索引：

* 对查询次数频次较高，且数据量较大的表建立索引；
* 索引字段的选择，最佳候选列应当从where子句的条件中提取，如果where子句中的组合比较多，那么应当挑选最常用、过滤效果最好的列的组合；
* 使用唯一索引，区分度高、使用效率越高；
* 索引可以有效的提升查询数据的效率 , 但索引数量并不是多多益善 , 索引越多 , 维护索引的代价自然也就水涨船高 . 对于插入, 更新, 删除 等 DML 操作比较繁琐的表来说 , 索引过多 , 会引入相当高的维护代价 , 降低 DML 操作的效率 , 增加相应操作的时间消耗 , 另外索引过多的话 , MySQL也会犯 选择困难症 , 虽然最终仍然会找到一个可用的索引 , 但无疑提高了索引的代价 .
* 使用段索引 , 索引创建之后也是使用硬盘来存储的 , 因此提高索引访问的 I/O 效率 , 也可以跳高总体的访问效率 . 假如构成索引的字段 总长度比较短 , 那么在给定大小的存储块内 , 可以存储更多的索引值 , 相应的可以有效地提升MySQL访问索引的 I/O 效率.
* 利用最左前缀的原则 , N个列组合而成的组合索引 , 那么相当于是创建了N 个索引 。 如果查询时where 子句使用了组成该索引的前几个字段 , 那么这条查询SQL可以利用组合索引来提升查询效率

参考文章：

* [MySQL索引原理](https://www.cnblogs.com/exceptioneye/p/5452068.html)
* [MySQL索引分类](https://blog.csdn.net/qq_17707713/article/details/90059408)
* [MySQL优化之索引篇](https://www.cnblogs.com/cryin107/p/13166865.html)

作者：[ 周二鸭](https://www.cnblogs.com/jojop/)

出处：[https://www.cnblogs.com/jojop/p/13951979.html](https://www.cnblogs.com/jojop/p/13951979.html)

版权：本站使用「[CC BY 4.0](https://creativecommons.org/licenses/by/4.0)」创作共享协议，转载请在文章明显位置注明作者及出处。

