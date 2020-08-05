# 有个定时任务突然不执行了，别急！原因可能在这

[https://mp.weixin.qq.com/s/4Y4UlpETEtmdr0peF5Ndew](https://mp.weixin.qq.com/s/4Y4UlpETEtmdr0peF5Ndew)

来源 \| [https://ricstudio.top/archives/dev-note-scheduled-thread-pool-executor](https://ricstudio.top/archives/dev-note-scheduled-thread-pool-executor)

程序发版之后一个定时任务突然挂了！

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/05/640-20200805110814188-110814.jpg)

“幸亏是用灰度跑的，不然完蛋了。😭”

之前因为在线程池踩过坑，阅读过`ThreadPoolExecutor`的源码，自以为不会再踩坑，没想到又一不小心踩坑了，只不过这次的坑踩在了`ScheduledThreadPoolExecutor`上面。写代码真的是要注意细节上的东西。

`ScheduledThreadPoolExecutor`是`ThreadPoolExecutor`功能的延伸（继承关系），按照以前的经验，很快就知道的问题所在，特此记录一下。希望小伙伴们别重蹈覆辙。

## 问题重现

代码模拟：

```text
public class ScheduledExecutorTest {

    private static LongAdder longAdder = new LongAdder();

    public static void main(String[] args) {

        ScheduledExecutorService scheduledExecutor = Executors.newSingleThreadScheduledExecutor();

        scheduledExecutor.scheduleAtFixedRate(ThreadExecutorExample::doTask,
                1, 1, TimeUnit.SECONDS);
    }

    private static void doTask() {

        int count = longAdder.intValue();
        longAdder.increment();

        System.out.println("定时任务开始执行 === " + count);

        // ① 下面这一段注释前和注释后的区别
        if (count == 3) {
            throw new RuntimeException("some runtime exception");
        }
    }
}
```

代码块①注释的情况下，执行结果：

```text
定时任务开始执行 === 0
定时任务开始执行 === 1
定时任务开始执行 === 2
定时任务开始执行 === 3
定时任务开始执行 === 4
定时任务开始执行 === 5
定时任务开始执行 === 6
定时任务开始执行 === 7
定时任务开始执行 === 8
.... 会一直执行下去
```

代码块①不注释的情况下，执行结果：

```text
定时任务开始执行 === 0
定时任务开始执行 === 1
定时任务开始执行 === 2
定时任务开始执行 === 3
// 停止输出，任务不再被执行
```

## 初步结论

因为任务最外面没有用`try-catch` 捕捉，或者说任务执行时，遇到了 Uncaught Exception，所以导致这个定时任务停止执行了。

## 走进源码看问题

有了初步的结论，我们需要知道的就是，`ScheduledExecutorService`这个定时线程调度器（定时任务线程池）在碰到 Uncaught Exception 的时候，是怎么处理的，是在哪一块导致任务停止的？

之前是看过`ThreadPoolExecutor`的源码，当线程池的线程工作时抛出 Uncaught Exception 时，会这个线程抛弃掉，然后再新启一个worker，来执行任务。在这里显然不一样，因为这个问题的主体是定时任务，定时任务的后续执行停止了，而不是worker线程。

带着问题，我们走进源码去看更深层次的答案。

> 这里说一句，本文不会成为`ScheduledThreadPoolExecutor`的完整源码解析，只是在具体问题场景下，讨论源码的运行。

```text
ScheduledExecutorService scheduledExecutor = Executors.newSingleThreadScheduledExecutor();
```

先看生成的`ScheduledExecutorService`实例，

```text
public static ScheduledExecutorService newSingleThreadScheduledExecutor() {
    return new DelegatedScheduledExecutorService
        (new ScheduledThreadPoolExecutor(1));
}
```

返回了一个`DelegatedScheduledExecutorService`对象，

```text
static class DelegatedScheduledExecutorService
        extends DelegatedExecutorService
        implements ScheduledExecutorService {
    private final ScheduledExecutorService e;
    DelegatedScheduledExecutorService(ScheduledExecutorService executor) {
        super(executor);
        e = executor;
    }
    public ScheduledFuture<?> schedule(Runnable command, long delay, TimeUnit unit) {
        return e.schedule(command, delay, unit);
    }
    public <V> ScheduledFuture<V> schedule(Callable<V> callable, long delay, TimeUnit unit) {
        return e.schedule(callable, delay, unit);
    }
    public ScheduledFuture<?> scheduleAtFixedRate(Runnable command, long initialDelay, long period, TimeUnit unit) {
        return e.scheduleAtFixedRate(command, initialDelay, period, unit);
    }
    public ScheduledFuture<?> scheduleWithFixedDelay(Runnable command, long initialDelay, long delay, TimeUnit unit) {
        return e.scheduleWithFixedDelay(command, initialDelay, delay, unit);
    }
}
```

发现这个类实际上就是把`ScheduledExecutorService` 包装了一层，实际上的动作是由`ScheduledThreadPoolExecutor`类执行的。

所以我们再进去看，这里我们关注的`scheduleAtFixedRate(...)`方法，也就是计划执行定时任务的方法。

我们先不急着看方法的实现，先看下它的接口层`ScheduledExecutorService`，这个方法的 JavaDoc 上面写了这么一段话：

> If any execution of the task encounters an exception, subsequent executions are suppressed. Otherwise, the task will only terminate via cancellation or termination of the executor. If any execution of this task takes longer than its period, then subsequent executions may start late, but will not concurrently execute.

_如果任务的任何一次执行遇到异常，则将禁止后续执行。其他情况下，任务将仅通过取消操作或终止线程池来停止。_

_如果某一次的执行时间超过了任务的间隔时间，后续任务会等当前这次执行结束才执行。_

这个方法的注释，已经告诉我们了在使用这个方法的时候，要注意的事项了。

1. 要注意发生异常时，任务终止的情况。
2. 要注意定时任务调度会等待正在执行的任务结束，才会发起下一轮调度，即使超过了间隔时间。

> 这里说一句，线程池的使用中，注释真的十分关键，把坑说的很清楚。（mdzz，说了那么多你自己还不是没看😓😓）

这个注释已经解释了一大半，但是我们这个是源码解析，当然看看里面是怎么做的，

```text
public ScheduledFuture<?> scheduleAtFixedRate(Runnable command,
                                              long initialDelay,
                                              long period,
                                              TimeUnit unit) {
    if (command == null || unit == null)
        throw new NullPointerException();
    if (period <= 0)
        throw new IllegalArgumentException();
    // ①
    ScheduledFutureTask<Void> sft =
        new ScheduledFutureTask<Void>(command,
                                      null,
                                      triggerTime(initialDelay, unit),
                                      unit.toNanos(period));
    RunnableScheduledFuture<Void> t = decorateTask(command, sft);
    sft.outerTask = t;
    delayedExecute(t);
    return t;
}

protected <V> RunnableScheduledFuture<V> decorateTask(
    Runnable runnable, RunnableScheduledFuture<V> task) {
    return task;
}

private void delayedExecute(RunnableScheduledFuture<?> task) {
    if (isShutdown())
        reject(task);
    else {
        super.getQueue().add(task);
        if (isShutdown() &&
            !canRunInCurrentRunState(task.isPeriodic()) &&
            remove(task))
            task.cancel(false);
        else
            ensurePrestart();
    }
}
```

这里的核心逻辑就是将 `Runnable` 包装成了一个`ScheduledFutureTask`对象，这个包装是在`FutureTask`基础上增加了定时调度需要的一些数据。（`FutureTask`是线程池的核心类之一）

`decorateTask`是一个钩子方法，用来给扩展用的，在这里的默认实现就是返回`ScheduledFutureTask`本身。

然后主逻辑就是通过`delayedExecute`放入队列中。（这里省略对源码中线程池shutdown情况处理的解释）

这里我们放一张图，简单描述一下`ScheduledThreadPoolExecutor`工作的过程：

image

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/08/05/640-20200805110814386-110814.png)

我们很容易都推断出来，我们想要找的对于 Uncaught Exception 逻辑的处理肯定是在任务执行的时候，从哪里可以看出来呢，就是`ScheduledFutureTask`的`run`方法。

```text
public void run() {
    // 是否是周期性任务
    boolean periodic = isPeriodic();
    // 如果不可以在当前状态下运行，就取消任务（将这个任务的状态设置为CANCELLED）。
    if (!canRunInCurrentRunState(periodic))
        cancel(false);
    else if (!periodic)
        // 如果不是周期性的任务，调用 FutureTask # run 方法
        ScheduledFutureTask.super.run();
    else if (ScheduledFutureTask.super.runAndReset()) {
        // 如果是周期性的。
  // 执行任务，但不设置返回值，成功后返回 true。

        // 设置下次执行时间
        setNextRunTime();
        // 再次将任务添加到队列中
        reExecutePeriodic(outerTask);
    }
}
```

这里我们关注的是`ScheduledFutureTask.super.runAndReset()`，实际上调用的是其父类`FutureTask`的

`runAndReset()`方法，这个方法会在执行成功之后重置线程状态，reset就是这个语义。

可以看到，当上述方法执行返回false的时候，就不会再次将任务添加的队列中，这和我们最开始看到的异常情况是一致的，看来答案就在这个方法里面。那我们接下去看看。

```text
protected boolean runAndReset() {
    if (state != NEW ||
        !UNSAFE.compareAndSwapObject(this, runnerOffset,
                                     null, Thread.currentThread()))
        return false;
    boolean ran = false;
    int s = state;
    try {
        Callable<V> c = callable;
        if (c != null && s == NEW) {
            try {
                // ① 任务执行
                c.call(); // don't set result
                ran = true;
            } catch (Throwable ex) {
                setException(ex);
            }
        }
    } finally {
        // runner must be non-null until state is settled to
        // prevent concurrent calls to run()
        runner = null;
        // state must be re-read after nulling runner to prevent
        // leaked interrupts
        s = state;
        if (s >= INTERRUPTING)
            handlePossibleCancellationInterrupt(s);
    }
    // 
    return ran && s == NEW;
}
```

代码块①是执行任务的地方，这里有一个默认为false的`ran`变量，当任务执行成功时，`ran`会被设成 true，即任务已执行。可以看到当代码块①抛出异常的时候，`ran` 等于false，`runAndReset()`返回给调用方的最终结果是false，也就应验了我们上面说的逻辑走向。

## 总结

整篇文章到这里结束啦，本篇主要介绍了当`ScheduledThreadPoolExecutor`碰到 Uncaught Exception 时的源码处理逻辑。我们自己在使用这个线程池时，需要注意对任务运行时异常的处理（最简单的方式就是在最外层加个`try-catch` ，然后捕捉打印日志）。

