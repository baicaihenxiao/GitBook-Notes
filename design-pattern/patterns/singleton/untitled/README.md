# （设计模式）单例模式的双重检查

## [https://www.jianshu.com/p/a9ecac4bd753](https://www.jianshu.com/p/a9ecac4bd753)

**1.双重检查锁定的由来**

下面是非线程安全的延迟初始化对象的示例代码。

```text
public class UnsafeLazyInitialization {
    private static Instance instance;
    public static Instance getInstance(){
        if(instance ==null)                  //1:A线程执行
            instance = new Instance();       //2:B线程执行
        return instance;
    }
}
```

在UnsafeLazyInitialization类中，假设A线程执行代码1的同时，B线程执行代码2。此时，线程A可能会看到instance引用的对象还没有完成初始化（原因之后分析）

对于UnsafeLazyInitialization类，我们可以对getInstance\(\)方法做同步处理来实现线程安全的延迟初始化。示例代码如下。

```text
public class safeLazyInitialization {
    private static Instance instance;
    public synchronized static Instance getInstance(){
        if(instance ==null)
            instance = new Instance();       
        return instance;
    }
}
```

由于对getInstance\(\)方法做了同步处理，synchronized将导致性能开销。如果getInstance（）被多个线程调用，将导致程序性能下降。反之，那么这个延迟初始化方案能提供令人满意的性能。

在早期的JVM中，synchronized（甚至是无竞争的synchronized）存在着巨大性能开销。因此，出现双重检查锁定。以下是示例代码。

```text
public class DoubleCheckedLocking {                     //1
    private static Instance instance;                   //2
    public  static Instance getInstance(){              //3
        if(instance ==null) {                           //4:第一次检查
            synchronized (DoubleCheckedLocking.class) { //5：加锁
                if (instance == null)                   //6：第二次检查
                    instance = new Instance();          //7：问题的根源处在这里
            }                                           //8
        }                                               //9
        return instance;                                //10
    }                                                   //11
}
```

如上面的代码所示，如果第一次检查instance不为null，那就不需要执行下面的加锁和初始化操作。因此，可以大幅降低synchronized带来的性能开销。

这样似乎很完美，但这是一个错误的优化！在线程执行到第4行，代码读取到instance不为null时，instance引用的对象可能还没有完成初始化。

**2.问题的根源**

前面的双重检查示例代码第7行创建了一个对象。这一行代码可以分解为如下的3行伪代码。

```text
memory=allocate();        //1:分配对象的内存空间
ctorInstance(memory);     //2:初始化对象
instance = memory;          //3:设置instance指向刚分配的内存地址
```

上面3行伪代码中的2和3之间，可能会被重排序。2和3重排序之后的执行时序如下。

```text
memory=allocate();        //1:分配对象的内存空间
instance = memory;          //3:设置instance指向刚分配的内存地址
                           //注意，此时对象还没有被初始化！
ctorInstance(memory);     //2:初始化对象
```

![](//upload-images.jianshu.io/upload_images/15720722-6dfbffce7e4e9697.png?imageMogr2/auto-orient/strip|imageView2/2/w/802/format/webp)多线程执行时序图.png

由于单线程内要遵守intra-thread semantics,从而能保证A线程的执行结果不会被改变。但是，当线程A和B按上图时序执行时，B线程将看到一个还没有被初始化的对象。

回到主题，DoubleCheckedLocking代码第7行（instance=new Instance\(\)；）如果发生重排序，拎一个并发执行的线程B就有可能在第4行判断instance不为null。线程B接下来访问instance所引用的对象，但此时这个对象可能还没有被A线程初始化！

在知晓了问题发生的根源之后，我们可以想出两个办法来实现线程安全的延迟初始化。

1.不允许2和3重排序

2.允许2和3重排序，但不允许其他线程“看到”这个重排序。

基于上面这两点，提出两个解决方案。

**3.1基于volatile的解决方案**

对于前面的基于双重检查锁定来实现延迟初始化的方案，只需要做一点小的修改（把instance声明为volatile型），就可以实现线程安全的延迟初始化。请看下面的示例代码。

```text
public class SafeDoubleCheckedLocking {
        private volatile static Instance instance;
        public  static Instance getInstance(){
           if(instance ==null) {
              synchronized (SafeDoubleCheckedLocking.class) {
                  if (instance == null)
                     instance = new Instance(); //instance为volatile，现在没问题了
                  }
            }
            return instance;
        }
}
```

当声明对象的引用为volatile后，之前的3行伪代码中的2和3之间的重排序，在多线程环境中将会被禁止。上面的示例代码江安如下的时序执行。多线程执行时序图.jpg

![](//upload-images.jianshu.io/upload_images/15720722-24b22a5a9bea33a3.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/1000/format/webp)

这个方案是通过禁止上图2和3之间的重排序，来保证线程安全的延迟初始化。

**3.2基于类初始化的解决方案**

JVM在类的初始化阶段（即在Class被加载后，且被线程使用之前），会执行类的初始化。在执行类的初始化期间，JVM会去获取一个锁。这个锁可以同步多个线程对同一个类的初始化。

基于这个特性可以实现另一种线程安全的延迟初始化方案。

```text
public class InstanceFactory{
        private static class InstanceHolder{
            public static Instance instance = new Instance();
        }
        public static Instance getInstance(){
            return InstanceHolder.instance;//这里将导致InstanceHolder类被初始化
        }
}
```

假设两个线程并发执行getInstance（）方法，下面是执行示意图。两个线程并发执行的示意图.jpg

![](//upload-images.jianshu.io/upload_images/15720722-a624fd5afdec1745.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/1000/format/webp)

这个方案的实质是：允许之前的3行伪代码中的2和3重排序，但不允许非构造线程（这里指线程B）“看到”这个重排序。

**4.总结**

通过对比基于volatile的双重检查锁定的方案和基于类初始化的方案，我们会发现基于类初始化的方案的实现代码更简洁。但基于volatile的双重检查锁定的方案有一个额外的优势：除了可以对静态字段实现延迟初始化外，还可以对实例字段实现延迟初始化。

字段延迟初始化降低了初始化类或创建实例的开销，但增加了访问被延迟初始化的字段的开销。在大多数时候，正常的初始化要优于延迟初始化。如果确实需要对实例字段使用线程安全的延迟初始化，请使用上面介绍的基于volatile的延迟初始化方案；如果确实需要对静态字段使用线程安全的延迟初始化，请使用基于类初始化的方案。  
  
作者：王大千\_xMol\_西摩尔  
链接：https://www.jianshu.com/p/a9ecac4bd753  
来源：简书  
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

