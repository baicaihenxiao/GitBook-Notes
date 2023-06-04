# 如何设计一个本地缓存

{% embed url="https://juejin.im/post/5dd942e15188257324096fe7" %}

{% embed url="https://mp.weixin.qq.com/s/8hI6WXp0OL4CHNA_09ldEQ" %}



## 如何设计一个本地缓存

### 前言

最近在看Mybatis的源码，刚好看到缓存这一块，Mybatis提供了一级缓存和二级缓存；一级缓存相对来说比较简单，功能比较齐全的是二级缓存，基本上满足了一个缓存该有的功能；当然如果拿来和专门的缓存框架如ehcache来对比可能稍有差距；本文我们将来整理一下实现一个本地缓存都应该需要考虑哪些东西。

### 考虑点

考虑点主要在数据用何种方式存储，能存储多少数据，多余的数据如何处理等几个点，下面我们来详细的介绍每个考虑点，以及该如何去实现；

#### 1.数据结构

首要考虑的就是数据该如何存储，用什么数据结构存储，最简单的就直接用Map来存储数据；或者复杂的如redis一样提供了多种数据类型哈希，列表，集合，有序集合等，底层使用了双端链表，压缩列表，集合，跳跃表等数据结构；

#### 2.对象上限

因为是本地缓存，内存有上限，所以一般都会指定缓存对象的数量比如1024，当达到某个上限后需要有某种策略去删除多余的数据；

#### 3.清除策略

上面说到当达到对象上限之后需要有清除策略，常见的比如有LRU(最近最少使用)、FIFO(先进先出)、LFU(最近最不常用)、SOFT(软引用)、WEAK(弱引用)等策略；

#### 4.过期时间

除了使用清除策略，一般本地缓存也会有一个过期时间设置，比如redis可以给每个key设置一个过期时间，这样当达到过期时间之后直接删除，采用清除策略+过期时间双重保证；

#### 5.线程安全

像redis是直接使用单线程处理，所以就不存在线程安全问题；而我们现在提供的本地缓存往往是可以多个线程同时访问的，所以线程安全是不容忽视的问题；并且线程安全问题是不应该抛给使用者去保证；

#### 6.简明的接口

提供一个傻瓜式的对外接口是很有必要的，对使用者来说使用此缓存不是一种负担而是一种享受；提供常用的get，put，remove，clear，getSize方法即可；

#### 7.是否持久化

这个其实不是必须的，是否需要将缓存数据持久化看需求；本地缓存如ehcache是支持持久化的，而guava是没有持久化功能的；分布式缓存如redis是有持久化功能的，memcached是没有持久化功能的；

#### 8.阻塞机制

在看Mybatis源码的时候，二级缓存提供了一个blocking标识，表示当在缓存中找不到元素时，它设置对缓存键的锁定；这样其他线程将等待此元素被填充，而不是命中数据库；其实我们使用缓存的目的就是因为被缓存的数据生成比较费时，比如调用对外的接口，查询数据库，计算量很大的结果等等；这时候如果多个线程同时调用get方法获取的结果都为null，每个线程都去执行一遍费时的计算，其实也是对资源的浪费；最好的办法是只有一个线程去执行，其他线程等待，计算一次就够了；但是此功能基本上都交给使用者来处理，很少有本地缓存有这种功能；

### 如何实现

以上大致介绍了实现一个本地缓存我们都有哪些需要考虑的地方，当然可能还有其他没有考虑到的点；下面继续看看关于每个点都应该如何去实现，重点介绍一下思路；

#### 1.数据结构

本地缓存最常见的是直接使用Map来存储，比如guava使用ConcurrentHashMap，ehcache也是用了ConcurrentHashMap，Mybatis二级缓存使用HashMap来存储：

```
Map<Object, Object> cache = new ConcurrentHashMap<Object, Object>()
复制代码
```

Mybatis使用HashMap本身是非线程安全的，所以可以看到起内部使用了一个SynchronizedCache用来包装，保证线程的安全性； 当然除了使用Map来存储，可能还使用其他数据结构来存储，比如redis使用了双端链表，压缩列表，整数集合，跳跃表和字典；当然这主要是因为redis对外提供的接口很丰富除了哈希还有列表，集合，有序集合等功能；

#### 2.对象上限

本地缓存常见的一个属性，一般缓存都会有一个默认值比如1024，在用户没有指定的情况下默认指定；当缓存的数据达到指定最大值时，需要有相关策略从缓存中清除多余的数据这就涉及到下面要介绍的清除策略；

#### 3.清除策略

配合对象上限之后使用，场景的清除策略如：LRU(最近最少使用)、FIFO(先进先出)、LFU(最近最不常用)、SOFT(软引用)、WEAK(弱引用)； **LRU**：Least Recently Used的缩写最近最少使用，移除最长时间不被使用的对象；常见的使用LinkedHashMap来实现，也是很多本地缓存默认使用的策略； **FIFO**：先进先出，按对象进入缓存的顺序来移除它们；常见使用队列Queue来实现； **LFU**：Least Frequently Used的缩写大概也是最近最少使用的意思，和LRU有点像；区别点在LRU的淘汰规则是基于访问时间，而LFU是基于访问次数的；可以通过HashMap并且记录访问次数来实现； **SOFT**：软引用基于垃圾回收器状态和软引用规则移除对象；常见使用SoftReference来实现； **WEAK**：弱引用更积极地基于垃圾收集器状态和弱引用规则移除对象；常见使用WeakReference来实现；

#### 4.过期时间

设置过期时间，让缓存数据在指定时间过后自动删除；常见的过期数据删除策略有两种方式：被动删除和主动删除； **被动删除**：每次进行get/put操作的时候都会检查一下当前key是否已经过期，如果过期则删除，类似如下代码：

```
if (System.currentTimeMillis() - lastClear > clearInterval) {
      clear();
}
复制代码
```

**主动删除**：专门有一个job在后台定期去检查数据是否过期，如果过期则删除，这其实可以有效的处理冷数据；

#### 5.线程安全

尽量用线程安全的类去存储数据，比如使用ConcurrentHashMap代替HashMap；或者提供相应的同步处理类，比如Mybatis提供了SynchronizedCache：

```
 public synchronized void putObject(Object key, Object object) {
    ...省略...
  }

  @Override
  public synchronized Object getObject(Object key) {
    ...省略...
  }
复制代码
```

#### 6.简明的接口

提供常用的get，put，remove，clear，getSize方法即可，比如Mybatis的Cache接口：

```
public interface Cache {
  String getId();
  void putObject(Object key, Object value);
  Object getObject(Object key);
  Object removeObject(Object key);
  void clear();
  int getSize();
  ReadWriteLock getReadWriteLock();
}
复制代码
```

再来看看guava提供的Cache接口，相对来说也是比较简洁的：

```
public interface Cache<K, V> {
  V getIfPresent(@CompatibleWith("K") Object key);
  V get(K key, Callable<? extends V> loader) throws ExecutionException;
  ImmutableMap<K, V> getAllPresent(Iterable<?> keys);
  void put(K key, V value);
  void putAll(Map<? extends K, ? extends V> m);
  void invalidate(@CompatibleWith("K") Object key);
  void invalidateAll(Iterable<?> keys);
  void invalidateAll();
  long size();
  CacheStats stats();
  ConcurrentMap<K, V> asMap();
  void cleanUp();
}
复制代码
```

#### 7.是否持久化

持久化的好处是重启之后可以再次加载文件中的数据，这样就起到类似热加载的功效；比如ehcache提供了是否持久化磁盘缓存的功能，将缓存数据存放在一个.data文件中；

```
diskPersistent="false" //是否持久化磁盘缓存
复制代码
```

redis更是将持久化功能发挥到极致，慢慢的有点像数据库了；提供了AOF和RDB两种持久化方式；当然很多情况下可以配合使用两种方式；

#### 8.阻塞机制

除了在Mybatis中看到了BlockingCache来实现此功能，之前在看**<>**的时候其中有实现一个很完美的缓存，大致代码如下：

```
public class Memoizerl<A, V> implements Computable<A, V> {
    private final Map<A, Future<V>> cache = new ConcurrentHashMap<A, Future<V>>();
    private final Computable<A, V> c;

    public Memoizerl(Computable<A, V> c) {
        this.c = c;
    }

    @Override
    public V compute(A arg) throws InterruptedException, ExecutionException {
        while (true) {
            Future<V> f = cache.get(arg);
            if (f == null) {
                Callable<V> eval = new Callable<V>() {
                    @Override
                    public V call() throws Exception {
                        return c.compute(arg);
                    }
                };
                FutureTask<V> ft = new FutureTask<V>(eval);
                f = cache.putIfAbsent(arg, ft);
                if (f == null) {
                    f = ft;
                    ft.run();
                }
                try {
                    return f.get();
                } catch (CancellationException e) {
                    cache.remove(arg, f);
                }
            }
        }
    }
}
复制代码
```

compute是一个计算很费时的方法，所以这里把计算的结果缓存起来，但是有个问题就是如果两个线程同时进入此方法中怎么保证只计算一次，这里最核心的地方在于使用了ConcurrentHashMap的putIfAbsent方法，同时只会写入一个FutureTask；

### 总结

本文大致介绍了要设计一个本地缓存都需要考虑哪些点：数据结构，对象上限，清除策略，过期时间，线程安全，阻塞机制，实用的接口，是否持久化；当然肯定有其他考虑点，欢迎补充。
