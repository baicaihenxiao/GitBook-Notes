# Java之戳中痛点之 synchronized

[https://mp.weixin.qq.com/s/TkLFuV9uwql3xzW3ePpqqg](https://mp.weixin.qq.com/s/TkLFuV9uwql3xzW3ePpqqg)



作者：Json\_wangqiang

cnblogs.com/JsonShare/p/11433302.html

## 概览：

* 简介：作用、地位、不控制并发的影响
* 用法：对象锁和类锁
* 多线程访问同步方法的7种情况
* 性质：可重入、不可中断
* 原理：加解锁原理、可重入原理、可见性原理
* 缺陷：效率低、不够灵活、无法预判是否成功获取到锁
* 如何选择Lock或Synchronized
* 如何提高性能、JVM如何决定哪个线程获取锁
* 总结

后续会有代码演示，测试环境 JDK8、IDEA

## 一、简介

### 1、作用

能够保证在同一时刻最多只有一个线程执行该代码，以保证并发安全的效果。

### 2、地位

* Synchronized是Java关键字，Java原生支持
* 最基本的互斥同步手段
* 并发编程的元老级别

### 3、不控制并发的影响

测试：两个线程同时a++，猜一下结果

```text
package cn.jsonshare.java.base.synchronizedtest;

/**
 * 不使用synchronized,两个线程同时a++
 *
 * @author JSON
 */
public class SynchronizedTest1 implements Runnable{
    static SynchronizedTest1 st = new SynchronizedTest1();

    static int a = 0;

    /**
     * 不使用synchronized,两个线程同时a++
     */
    public static void main(String[] args) throws Exception{
        Thread t1 = new Thread(st);
        Thread t2 = new Thread(st);
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        System.out.println(a);
    }

    @Override
    public void run(){
        for(int i=0; i<10000; i++){
            a++;
        }
    }
}
```

预期是20000，但多次执行的结果都小于20000

```text
10108
11526
10736
...
```

## 二、用法：对象锁和类锁

### 1、对象锁

* 代码块形式：手动指定锁对象
* 方法锁形式：synchronized修饰方法，锁对象默认为this

```text
package cn.jsonshare.java.base.synchronizedtest;

/**
 * 对象锁实例: 代码块形式
 *
 * @author JSON
 */
public class SynchronizedTest2 implements Runnable{
    static SynchronizedTest2 st = new SynchronizedTest2();

    public static void main(String[] args) {
        Thread t1 = new Thread(st);
        Thread t2 = new Thread(st);
        t1.start();
        t2.start();
        while(t1.isAlive() || t2.isAlive()){

        }
        System.out.println("run over");

    }

    @Override
    public void run(){
        synchronized (this){
            System.out.println("开始执行:" + Thread.currentThread().getName());
            try {
                // 模拟执行内容
                Thread.sleep(3000);
            }catch (Exception e){
                e.printStackTrace();
            }
            System.out.println("执行结束:" + Thread.currentThread().getName());
        }
    }
}
```

```text
package cn.jsonshare.java.base.synchronizedtest;

/**
 * 对象锁实例：synchronized方法
 * @author JSON
 */
public class SynchronizedTest3 implements Runnable{
    static SynchronizedTest3 st = new SynchronizedTest3();

    public static void main(String[] args) throws Exception{
        Thread t1 = new Thread(st);
        Thread t2 = new Thread(st);
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        System.out.println("run over");
    }

    @Override
    public void run(){
        method();
    }

    public synchronized void method(){
        System.out.println("开始执行:" + Thread.currentThread().getName());
        try {
            // 模拟执行内容
            Thread.sleep(3000);
        }catch (Exception e){
            e.printStackTrace();
        }
        System.out.println("执行结束:" + Thread.currentThread().getName());
    }
}
```

结果：

```text
开始执行:Thread-0
执行结束:Thread-0
开始执行:Thread-1
执行结束:Thread-1
run over
```

### 2、类锁

**概念：**Java类可能有多个对象，但只有一个Class对象

**本质：**所谓的类锁，不过是Class对象的锁而已

**用法和效果：**类锁只能在同一时刻被一个对象拥有

形式1：synchronized加载static方法上

形式2：synchronized\(\*.class\)代码块

```text
package cn.jsonshare.java.base.synchronizedtest;

/**
 * 类锁：synchronized加载static方法上
 *
 * @author JSON
 */
public class SynchronizedTest4 implements Runnable{

    static SynchronizedTest4 st1 = new SynchronizedTest4();
    static SynchronizedTest4 st2 = new SynchronizedTest4();

    public static void main(String[] args) throws Exception{
        Thread t1 = new Thread(st1);
        Thread t2 = new Thread(st2);
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        System.out.println("run over");
    }

    @Override
    public void run(){
        method();
    }

    public static synchronized void method(){
        System.out.println("开始执行:" + Thread.currentThread().getName());
        try {
            // 模拟执行内容
            Thread.sleep(3000);
        }catch (Exception e){
            e.printStackTrace();
        }
        System.out.println("执行结束:" + Thread.currentThread().getName());
    }
}
```

```text
package cn.jsonshare.java.base.synchronizedtest;

/**
 * 类锁：synchronized(*.class)代码块
 *
 * @author JSON
 */
public class SynchronizedTest5 implements Runnable{
    static SynchronizedTest4 st1 = new SynchronizedTest4();
    static SynchronizedTest4 st2 = new SynchronizedTest4();

    public static void main(String[] args) throws Exception{
        Thread t1 = new Thread(st1);
        Thread t2 = new Thread(st2);
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        System.out.println("run over");
    }

    @Override
    public void run(){
        method();
    }

    public void method(){
        synchronized(SynchronizedTest5.class){
            System.out.println("开始执行:" + Thread.currentThread().getName());
            try {
                // 模拟执行内容
                Thread.sleep(3000);
            }catch (Exception e){
                e.printStackTrace();
            }
            System.out.println("执行结束:" + Thread.currentThread().getName());
        }
    }
}
```

结果：

```text
开始执行:Thread-0
执行结束:Thread-0
开始执行:Thread-1
执行结束:Thread-1
run over
```

Java知音公众号内回复“面试题聚合”，送你一份面试题宝典

## 三、多线程访问同步方法的7种情况

1. 两个线程同时访问一个对象的相同的synchronized方法
2. 两个线程同时访问两个对象的相同的synchronized方法
3. 两个线程同时访问两个对象的相同的static的synchronized方法
4. 两个线程同时访问同一对象的synchronized方法与非synchronized方法
5. 两个线程访问同一对象的不同的synchronized方法
6. 两个线程同时访问同一对象的static的synchronized方法与非static的synchronized方法
7. 方法抛出异常后，会释放锁吗

仔细看下面示例代码结果输出的结果，注意输出时间间隔，来预测结论

场景1：

```text
package cn.jsonshare.java.base.synchronizedtest;

/**
 * 两个线程同时访问一个对象的相同的synchronized方法
 *
 * @author JSON
 */
public class SynchronizedScene1 implements Runnable{
    static SynchronizedScene1 ss = new SynchronizedScene1();

    public static void main(String[] args) throws Exception{
        Thread t1 = new Thread(ss);
        Thread t2 = new Thread(ss);
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        System.out.println("run over");
    }

    @Override
    public void run(){
        method();
    }

    public synchronized void method(){
        System.out.println("开始执行:" + Thread.currentThread().getName());
        try {
            // 模拟执行内容
            Thread.sleep(3000);
        }catch (Exception e){
            e.printStackTrace();
        }
        System.out.println("执行结束:" + Thread.currentThread().getName());
    }
}
```

场景2：

```text
package cn.jsonshare.java.base.synchronizedtest;

/**
 * 两个线程同时访问两个对象的相同的synchronized方法
 *
 * @author JSON
 */
public class SynchronizedScene2 implements Runnable{
    static SynchronizedScene2 ss1 = new SynchronizedScene2();
    static SynchronizedScene2 ss2 = new SynchronizedScene2();

    public static void main(String[] args) throws Exception{
        Thread t1 = new Thread(ss1);
        Thread t2 = new Thread(ss2);
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        System.out.println("run over");
    }

    @Override
    public void run(){
        method();
    }

    public synchronized void method(){
        System.out.println("开始执行:" + Thread.currentThread().getName());
        try {
            // 模拟执行内容
            Thread.sleep(3000);
        }catch (Exception e){
            e.printStackTrace();
        }
        System.out.println("执行结束:" + Thread.currentThread().getName());
    }
}
```

场景3：

```text
package cn.jsonshare.java.base.synchronizedtest;

/**
 * 两个线程同时访问两个对象的相同的static的synchronized方法
 *
 * @author JSON
 */
public class SynchronizedScene3 implements Runnable{
    static SynchronizedScene3 ss1 = new SynchronizedScene3();
    static SynchronizedScene3 ss2 = new SynchronizedScene3();

    public static void main(String[] args) throws Exception{
        Thread t1 = new Thread(ss1);
        Thread t2 = new Thread(ss2);
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        System.out.println("run over");
    }

    @Override
    public void run(){
        method();
    }

    public synchronized static void method(){
        System.out.println("开始执行:" + Thread.currentThread().getName());
        try {
            // 模拟执行内容
            Thread.sleep(3000);
        }catch (Exception e){
            e.printStackTrace();
        }
        System.out.println("执行结束:" + Thread.currentThread().getName());
    }
}
```

场景4：

```text
package cn.jsonshare.java.base.synchronizedtest;

/**
 * 两个线程同时访问同一对象的synchronized方法与非synchronized方法
 *
 * @author JSON
 */
public class SynchronizedScene4 implements Runnable{
    static SynchronizedScene4 ss1 = new SynchronizedScene4();

    public static void main(String[] args) throws Exception{
        Thread t1 = new Thread(ss1);
        Thread t2 = new Thread(ss1);
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        System.out.println("run over");
    }

    @Override
    public void run(){
        // 模拟两个线程同时访问 synchronized方法与非synchronized方法
        if(Thread.currentThread().getName().equals("Thread-0")){
            method1();
        }else{
            method2();
        }
    }

    public void method1(){
        System.out.println("method1开始执行:" + Thread.currentThread().getName());
        try {
            // 模拟执行内容
            Thread.sleep(3000);
        }catch (Exception e){
            e.printStackTrace();
        }
        System.out.println("method1执行结束:" + Thread.currentThread().getName());
    }

    public synchronized void method2(){
        System.out.println("method2开始执行:" + Thread.currentThread().getName());
        try {
            // 模拟执行内容
            Thread.sleep(3000);
        }catch (Exception e){
            e.printStackTrace();
        }
        System.out.println("method2执行结束:" + Thread.currentThread().getName());
    }
}
```

场景5：

```text
package cn.jsonshare.java.base.synchronizedtest;

/**
 * 两个线程访问同一对象的不同的synchronized方法
 *
 * @author JSON
 */
public class SynchronizedScene5 implements Runnable{
    static SynchronizedScene5 ss1 = new SynchronizedScene5();

    public static void main(String[] args) throws Exception{
        Thread t1 = new Thread(ss1);
        Thread t2 = new Thread(ss1);
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        System.out.println("run over");
    }

    @Override
    public void run(){
        // 模拟两个线程同时访问不同的synchronized方法
        if(Thread.currentThread().getName().equals("Thread-0")){
            method1();
        }else{
            method2();
        }
    }

    public synchronized void method1(){
        System.out.println("method1开始执行:" + Thread.currentThread().getName());
        try {
            // 模拟执行内容
            Thread.sleep(3000);
        }catch (Exception e){
            e.printStackTrace();
        }
        System.out.println("method1执行结束:" + Thread.currentThread().getName());
    }

    public synchronized void method2(){
        System.out.println("method2开始执行:" + Thread.currentThread().getName());
        try {
            // 模拟执行内容
            Thread.sleep(3000);
        }catch (Exception e){
            e.printStackTrace();
        }
        System.out.println("method2执行结束:" + Thread.currentThread().getName());
    }
}
```

场景6：

```text
package cn.jsonshare.java.base.synchronizedtest;

/**
 * 两个线程同时访问同一对象的static的synchronized方法与非static的synchronized方法
 *
 * @author JSON
 */
public class SynchronizedScene6 implements Runnable{
    static SynchronizedScene6 ss1 = new SynchronizedScene6();

    public static void main(String[] args) throws Exception{
        Thread t1 = new Thread(ss1);
        Thread t2 = new Thread(ss1);
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        System.out.println("run over");
    }

    @Override
    public void run(){
        // 模拟两个线程同时访问static的synchronized方法与非static的synchronized方法
        if(Thread.currentThread().getName().equals("Thread-0")){
            method1();
        }else{
            method2();
        }
    }

    public static synchronized void method1(){
        System.out.println("method1开始执行:" + Thread.currentThread().getName());
        try {
            // 模拟执行内容
            Thread.sleep(3000);
        }catch (Exception e){
            e.printStackTrace();
        }
        System.out.println("method1执行结束:" + Thread.currentThread().getName());
    }

    public synchronized void method2(){
        System.out.println("method2开始执行:" + Thread.currentThread().getName());
        try {
            // 模拟执行内容
            Thread.sleep(3000);
        }catch (Exception e){
            e.printStackTrace();
        }
        System.out.println("method2执行结束:" + Thread.currentThread().getName());
    }
}
```

场景7：

```text
package cn.jsonshare.java.base.synchronizedtest;

/**
 * 方法抛出异常后，会释放锁吗
 *
 * @author JSON
 */
public class SynchronizedScene7 implements Runnable{
    static SynchronizedScene7 ss1 = new SynchronizedScene7();

    public static void main(String[] args) throws Exception{
        Thread t1 = new Thread(ss1);
        Thread t2 = new Thread(ss1);
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        System.out.println("run over");
    }

    @Override
    public void run(){
        method1();
    }

    public synchronized void method1(){
        System.out.println("method1开始执行:" + Thread.currentThread().getName());
        try {
            // 模拟执行内容
            Thread.sleep(3000);
        }catch (Exception e){
            e.printStackTrace();
        }
        // 模拟异常
        throw new RuntimeException();
        //System.out.println("method1执行结束:" + Thread.currentThread().getName());
    }
}
```

## 

Java知音公众号内回复“面试题聚合”，送你一份面试题宝典

## 总结：

**1、两个线程同时访问一个对象的相同的synchronized方法**

同一实例拥有同一把锁，其他线程必然等待，顺序执行

**2、两个线程同时访问两个对象的相同的synchronized方法**

不同的实例拥有的锁是不同的，所以不影响，并行执行

**3、两个线程同时访问两个对象的相同的static的synchronized方法**

静态同步方法，是类锁，所有实例是同一把锁，其他线程必然等待，顺序执行

**4、两个线程同时访问同一对象的synchronized方法与非synchronized方法**

非synchronized方法不受影响，并行执行

**5、两个线程访问同一对象的不同的synchronized方法**

同一实例拥有同一把锁，所以顺序执行（说明：锁的是this对象==同一把锁）

**6、两个线程同时访问同一对象的static的synchronized方法与非static的synchronized方法**

static同步方法是类锁，非static是对象锁，原理上是不同的锁，所以不受影响，并行执行

**7、方法抛出异常后，会释放锁吗**

会自动释放锁，这里区别Lock，Lock需要显示的释放锁

**3个核心思想：**

* 一把锁只能同时被一个线程获取，没有拿到锁的线程必须等待（对应1、5的情景）
* 每个实例都对应有自己的一把锁，不同的实例之间互不影响；例外：锁对象是\*.class以及synchronized被static修饰的时候，所有对象共用同一把锁（对应2、3、4、6情景）
* 无论是方法正常执行完毕还是方法抛出异常，都会释放锁（对应7情景）

## 补充：

问题：目前进入到被synchronized修饰的方法，这个方法里边调用了非synchronized方法，是线程安全的吗？

```text
package cn.jsonshare.java.base.synchronizedtest;

/**
 * 目前进入到被synchronized修饰的方法，这个方法里边调用了非synchronized方法，是线程安全的吗？
 *
 * @author JSON
 */
public class SynchronizedScene8 {
    public static void main(String[] args) {
        new Thread(() -> {
            method1();
        }).start();

        new Thread(() -> {
            method1();
        }).start();
    }

    public static synchronized void method1() {
        method2();
    }

    private static void method2() {
        System.out.println(Thread.currentThread().getName() + "进入非Synchronized方法");
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println(Thread.currentThread().getName() + "结束非Synchronized方法");
    }
}
```

结论：这样是线程安全的

## 四、性质

### 1、可重入

指的是同一线程的外层函数获取锁之后，内层函数可以直接再次获取该锁

**Java典型的可重入锁：**synchronized、ReentrantLock

**好处：**避免死锁，提升封装性

**粒度：**线程而非调用

* 情况1：证明同一方法是可重入的
* 情况2：证明可重入不要求是同一方法
* 情况3：证明可重入不要求是同一类中的

### 2、不可中断

一旦这个锁被别的线程获取了，如果我现在想获得，我只能选择等待或者阻塞，直到别的线程释放这个锁，如果别的线程永远不释放锁，那么我只能永远的等待下去。

相比之下，Lock类可以拥有中断的能力，第一点：如果我觉得我等待的时间太长了，有权中断现在已经获取到锁的线程执行；第二点：如果我觉得我等待的时间太长了不想再等了，也可以退出。

## 五、原理

### 1、加解锁原理（现象、时机、深入JVM看字节码）

**现象：**每一个类的实例对应一把锁，每一个synchronized方法都必须首先获得调用该方法的类的实例的锁，方能执行，否则就会阻塞，方法执行完成或者抛出异常，锁被释放，被阻塞线程才能获取到该锁，执行。

**获取和释放锁的时机：**内置锁或监视器锁

```text
package cn.jsonshare.java.base.synchronizedtest;

import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 * method1 等价于 method2
 *
 * @author JSON
 * @date 2019-08-29
 */
public class SynchronizedToLock1 {
    Lock lock = new ReentrantLock();

    public synchronized void method1(){
        System.out.println("执行method1");
    }

    public void method2(){
        lock.lock();
        try {
            System.out.println("执行method2");
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            lock.unlock();
        }
    }

    public static void main(String[] args) {
        SynchronizedToLock1 sl = new SynchronizedToLock1();

        // method1 等价于 method2
        sl.method1();
        sl.method2();
    }
}
```

深入JVM看字节码：

```text
...
monitorenter指令
...
monitorexit指令
...
```

### 2、可重入原理（加锁次数计数器）

JVM负责跟踪对象被加锁的次数

线程第一次给对象加锁的时候，计数变为1，每当这个相同的线程在此对象上再次获得锁时，计数会递增

每当任务离开时，计数递减，当计数为0的时候，锁被完全释放

### 3、可见性原理（内存模型）

Java内存模型

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/19/640-20200819232354172-232354.jpg)

线程A向线程B发送数据的过程（JMM控制）

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/19/640-20200819232354601-232354.jpg)

synchronized关键字实现可见性：

> 被synchronized修饰，那么执行完成后，对对象所做的任何修改都要在释放锁之前，都要从线程内存写入到主内存，所以主内存中的数据是最新的。

## 六、缺陷

### 1、效率低

1\)、锁的释放情况少（线程执行完成或者异常情况释放）

2\)、试图获得锁时不能设定超时（只能等待）

3\)、不能中断一个正在试图获得锁的线程（不能中断）

### 2、不够灵活

加锁和释放的时机比较单一，每个锁仅有单一的条件（某个对象），可能是不够的

比如：读写锁更灵活

### 3、无法预判是否成功获取到锁

## 七、常见问题

### 1、synchronized关键字注意点：

* 锁对象不能为空
* 作用域不宜过大
* 避免死锁

### 2、如何选择Lock和synchronized关键字？

总结建议（优先避免出错的原则）：

* 如果可以的话，尽量优先使用java.util.concurrent各种类（不需要考虑同步工作，不容易出错）
* 优先使用synchronized，这样可以减少编写代码的量，从而可以减少出错率
* 若用到Lock或Condition独有的特性，才使用Lock或Condition

## 八、总结

一句话总结synchronized：

> JVM会自动通过使用monitor来加锁和解锁，保证了同一时刻只有一个线程可以执行指定的代码，从而保证线程安全，同时具有可重入和不可中断的特性。

