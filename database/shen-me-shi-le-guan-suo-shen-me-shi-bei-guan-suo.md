# ✔️什么是乐观锁，什么是悲观锁

{% embed url="https://www.jianshu.com/p/d2ac26ca6525" %}

#### **概念**

*   **悲观锁  访问或者提交前加锁** 是“先取锁再访问”的保守策略，分为 共享锁【Shared lock】 和 排他锁【Exclusive lock】。

    _使用select…for update会把数据给锁住，不过需要注意一些锁的级别，MySQL InnoDB默认行级锁。行级锁都是基于索引的，如果一条SQL语句用不到索引是不会使用行级锁的，会使用表级锁把整张表锁住，这点需要注意。_


* **乐观锁 提交时验证** 主要两个步骤：冲突检测和数据更新。其实现方式有一种比较典型的就是**CAS(Compare and Swap)**。
  1. 版本控制，数据库加version字段，提交时比较version。~~（或者使用时间戳）~~
  2.  先查询出数据库数量为3，`update set quantity=2 where id=1 and quantity=3`存在**ABA问题**

      或者直接不查数量直接`update set quantity=quantity-1 where id=1 and quantity-1>=0`



#### 比较

1. 乐观锁并**未真正加锁，效率高**。一旦**锁的粒度掌握不好，更新失败的概率就会比较高**，容易发生业务失败。
2. 悲观锁**依赖数据库锁，效率低。更新失败的概率比较低**。



## 一、并发控制

当程序中可能出现并发的情况时，就需要通过一定的手段来保证在并发情况下数据的准确性，通过这种手段保证了当前用户和其他用户一起操作时，所得到的结果和他单独操作时的结果是一样的。这种手段就叫做并发控制。并发控制的目的是保证一个用户的工作不会对另一个用户的工作产生不合理的影响。

**没有做好并发控制，就可能导致**[**脏读、幻读和不可重复读**](https://www.jianshu.com/p/7e76ce65e3ad)**等问题。**

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/21/webp-093443.jpg)

常说的并发控制，一般都和数据库管理系统（DBMS）有关。在DBMS中的并发控制的任务，是确保在多个事务同时存取数据库中同一数据时，不破坏事务的隔离性和统一性以及数据库的统一性。

`实现并发控制的主要手段大致可以分为乐观并发控制和悲观并发控制两种。` 首先要明确：无论是悲观锁还是乐观锁，都是人们定义出来的概念，可以认为是一种思想。其实不仅仅是关系型数据库系统中有乐观锁和悲观锁的概念，像hibernate、tair、memcache等都有类似的概念。所以，不应该拿乐观锁、悲观锁和其他的数据库锁等进行对比。

## 二、悲观锁(Pessimistic Lock)

当要对数据库中的一条数据进行修改的时候，为了避免同时被其他人修改，最好的办法就是直接对该数据进行加锁以防止并发。这种借助数据库锁机制，在修改数据之前先锁定，再修改的方式被称之为悲观并发控制【又名“悲观锁”，Pessimistic Concurrency Control，缩写“PCC”】。

**百度百科：** 悲观锁，正如其名，具有强烈的独占和排他特性。它指的是对数据被外界(包括本系统当前的其他事务，以及来自外部系统的事务处理)修改持保守态度。因此，在整个数据处理过程中，将数据处于锁定状态。悲观锁的实现，往往依靠数据库提供的锁机制(也只有数据库层提供的锁机制才能真正保证数据访问的排他性，否则，即使在本系统中实现了加锁机制，也无法保证外部系统不会修改数据)。

之所以叫做悲观锁，是因为这是一种对数据的修改抱有悲观态度的并发控制方式。我们一般认为数据被并发修改的概率比较大，所以需要在修改之前先加锁。

**悲观锁主要分为共享锁或排他锁**

* 共享锁【Shared lock】又称为读锁，简称S锁。顾名思义，共享锁就是多个事务对于同一数据可以共享一把锁，都能访问到数据，但是只能读不能修改。
* 排他锁【Exclusive lock】又称为写锁，简称X锁。顾名思义，排他锁就是不能与其他锁并存，如果一个事务获取了一个数据行的排他锁，其他事务就不能再获取该行的其他锁，包括共享锁和排他锁，但是获取排他锁的事务是可以对数据行读取和修改。

悲观并发控制实际上是“先取锁再访问”的保守策略，为数据处理的安全提供了保证。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/21/webp-20200721093444481-093444.jpg)

悲观锁

但是在效率方面，处理加锁的机制会让数据库产生额外的开销，还有增加产生死锁的机会。另外还会降低并行性，一个事务如果锁定了某行数据，其他事务就必须等待该事务处理完才可以处理那行数据。

## 三、乐观锁(Optimistic Locking)

乐观锁是相对悲观锁而言的，乐观锁假设数据一般情况下不会造成冲突，所以在数据进行提交更新的时候，才会正式对数据的冲突与否进行检测，如果发现冲突了，则返回给用户错误的信息，让用户决定如何去做。

**百度百科：** 乐观锁机制采取了更加宽松的加锁机制。乐观锁是相对悲观锁而言，也是为了避免数据库幻读、业务处理时间过长等原因引起数据处理错误的一种机制，但乐观锁不会刻意使用数据库本身的锁机制，而是依据数据本身来保证数据的正确性。

相对于悲观锁，在对数据库进行处理的时候，乐观锁并不会使用数据库提供的锁机制。一般的实现乐观锁的方式就是记录数据版本。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/21/webp-20200721093446279-093446.jpg)

乐观锁

乐观并发控制相信事务之间的数据竞争(data race)的概率是比较小的，因此尽可能直接做下去，直到提交的时候才去锁定，所以不会产生任何锁和死锁。

## 四、实现方式

### 1️⃣悲观锁实现方式

悲观锁的实现，往往依靠数据库提供的锁机制。在数据库中，悲观锁的流程如下：

1. 在对记录进行修改前，先尝试为该记录加上排他锁(exclusive locking)。
2. 如果加锁失败，说明该记录正在被修改，那么当前查询可能要等待或者抛出异常。具体响应方式由开发者根据实际需要决定。
3. 如果成功加锁，那么就可以对记录做修改，事务完成后就会解锁了。
4. 期间如果有其他对该记录做修改或加排他锁的操作，都会等待解锁或直接抛出异常。

**拿比较常用的MySql Innodb引擎举例，来说明一下在SQL中如何使用悲观锁。**

要使用悲观锁，必须关闭MySQL数据库的自动提交属性。因为MySQL默认使用autocommit模式，也就是说，当执行一个更新操作后，MySQL会立刻将结果进行提交。(sql语句：set autocommit=0)

以电商下单扣减库存的过程说明一下悲观锁的使用：

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/21/webp-20200721093446329-093446.jpg)

悲观锁使用

以上，在对id = 1的记录修改前，先通过[for update](https://www.jianshu.com/p/6ecc0d21dc50)的方式进行加锁，然后再进行修改。这就是比较典型的悲观锁策略。

如果以上修改库存的代码发生并发，同一时间只有一个线程可以开启事务并获得id=1的锁，其它的事务必须等本次事务提交之后才能执行。这样可以保证当前的数据不会被其它事务修改。

上面提到，**使用select…for update会把数据给锁住，不过需要注意一些锁的级别，MySQL InnoDB默认行级锁。行级锁都是基于索引的，如果一条SQL语句用不到索引是不会使用行级锁的，会使用表级锁把整张表锁住，这点需要注意。**

### 2️⃣乐观锁实现方式

使用乐观锁就不需要借助数据库的锁机制了。

乐观锁的概念中其实已经阐述了它的具体实现细节。主要就是两个步骤：冲突检测和数据更新。其实现方式有一种比较典型的就是**CAS(Compare and Swap)**。

CAS是项乐观锁技术，当多个线程尝试使用CAS同时更新同一个变量时，只有其中一个线程能更新变量的值，而其它线程都失败，失败的线程并不会被挂起，而是被告知这次竞争中失败，并可以再次尝试。

比如前面的扣减库存问题，通过乐观锁可以实现如下：

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/21/webp-20200721093446383-093446.jpg)

乐观锁使用

以上，在更新之前，先查询一下库存表中当前库存数(quantity)，然后在做update的时候，以库存数作为一个修改条件。当提交更新的时候，判断数据库表对应记录的当前库存数与第一次取出来的库存数进行比对，如果数据库表当前库存数与第一次取出来的库存数相等，则予以更新，否则认为是过期数据。

以上更新语句存在一个比较重要的问题，即传说中的**ABA问题**。

比如说一个线程one从数据库中取出库存数3，这时候另一个线程two也从数据库中取出库存数3，并且two进行了一些操作变成了2，然后two又将库存数变成3，这时候线程one进行CAS操作发现数据库中仍然是3，然后one操作成功。尽管线程one的CAS操作成功，但是不代表这个过程就是没有问题的。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/21/webp-20200721093448848-093448.jpg)

ABA

有一个比较好的办法可以解决ABA问题，那就是通过一个单独的可以顺序递增的version字段。改为以下方式即可：

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/21/webp-20200721093451613-093451.jpg)

ABA的解决

乐观锁每次在执行数据的修改操作时，都会带上一个版本号，一旦版本号和数据的版本号一致就可以执行修改操作并对版本号执行+1操作，否则就执行失败。因为每次操作的版本号都会随之增加，所以不会出现ABA问题，因为版本号只会增加不会减少。

除了version以外，还可以使用时间戳，因为时间戳天然具有顺序递增性。

以上SQL其实还是有一定的问题的，就是一旦遇上高并发的时候，就只有一个线程可以修改成功，那么就会存在大量的失败。对于像淘宝这样的电商网站，高并发是常有的事，总让用户感知到失败显然是不合理的。所以，还是要想办法减少乐观锁的粒度的。有一条比较好的建议，可以减小乐观锁力度，最大程度的提升吞吐率，提高并发能力！如下：

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/21/webp-20200721093451664-093451.jpg)

优化

以上SQL语句中，**如果用户下单数为1，则通过quantity - 1 > 0的方式进行乐观锁控制。**

以上update语句，在执行过程中，会在一次原子操作中自己查询一遍quantity的值，并将其扣减掉1。

高并发环境下锁粒度把控是一门重要的学问，选择一个好的锁，在保证数据安全的情况下，可以大大提升吞吐率，进而提升性能。

## 五、如何选择

在乐观锁与悲观锁的选择上面，主要看下两者的区别以及适用场景就可以了。

1. 乐观锁并**未真正加锁，效率高**。一旦**锁的粒度掌握不好，更新失败的概率就会比较高**，容易发生业务失败。
2. 悲观锁**依赖数据库锁，效率低。更新失败的概率比较低**。

随着互联网三高架构（高并发、高性能、高可用）的提出，悲观锁已经越来越少的被应用到生产环境中了，尤其是并发量比较大的业务场景。

## [MySQL乐观锁电商库存并发问题应用](https://www.jianshu.com/p/efe85c5b4d62)
