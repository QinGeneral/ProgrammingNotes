# Java 线程池

[TOC]

Java 线程是一个重量级的资源对象，应该避免频繁的创建和销毁，线程池可以重复利用线程，减少线程创建和销毁的开销。

## 线程池状态

![](https://p0.meituan.net/travelcube/62853fa44bfa47d63143babe3b5a4c6e82532.png)
![](https://p0.meituan.net/travelcube/582d1606d57ff99aa0e5f8fc59c7819329028.png)

## 线程池参数

```java
ThreadPoolExecutor(
    int corePoolSize,
    int maximumPoolSize,
    long keepAliveTime,
    TimeUnit unit,
    BlockingQueue<Runnable> workQueue,
    ThreadFactory threadFactory,
    RejectedExecutionHandler handler)
```

线程池有几个关键参数：核心线程数（最小线程数）、最大线程数、任务队列、拒绝策略。

核心线程是一直运行的线程，即使没有任务，也不会被回收。除非调用 `allowCoreThreadTimeOut` 方法，将根据 keepAliveTime 参数回收核心线程。非核心线程执行完会根据 keepAliveTime 回收。

线程池执行任务时，如果在任务运行时抛出异常，会导致执行任务的线程终止，而且收不到任务通知。最稳妥的方法是捕获异常按需处理。

任务队列建议使用有界队列，无界队列可能会导致 OOM。Executors 的很多方法使用了无界队列，所以不建议使用 Executors 创建线程池。

ThreadPoolExecutor 提供了四种拒绝策略：
1. AbortPolicy：默认策略，抛出 RejectedExecutionException 异常；
2. CallerRunsPolicy：调用者线程自己去执行任务；
3. DiscardPolicy：直接丢弃任务，不会抛出异常；
4. DiscardOldestPolicy：丢弃最老的任务，执行当前任务。

## 线程池执行任务流程

线程池执行一个任务的流程如下：

1. 如果正在执行的线程数小于最大核心线程数，创建新核心线程执行任务；
2. 如果任务队列没满，将任务加入队列；
3. 如果当前运行的线程数小于最大线程数，创建非核心线程执行任务；
4. 如果任务队列满了，且当前运行的线程数等于最大线程数，执行拒绝策略。

![](https://p0.meituan.net/travelcube/31bad766983e212431077ca8da92762050214.png)

```java
    public void execute(Runnable command) {
        if (command == null)
            throw new NullPointerException();
        /*
         * Proceed in 3 steps:
         *
         * 1. If fewer than corePoolSize threads are running, try to
         * start a new thread with the given command as its first
         * task.  The call to addWorker atomically checks runState and
         * workerCount, and so prevents false alarms that would add
         * threads when it shouldn't, by returning false.
         *
         * 2. If a task can be successfully queued, then we still need
         * to double-check whether we should have added a thread
         * (because existing ones died since last checking) or that
         * the pool shut down since entry into this method. So we
         * recheck state and if necessary roll back the enqueuing if
         * stopped, or start a new thread if there are none.
         *
         * 3. If we cannot queue task, then we try to add a new
         * thread.  If it fails, we know we are shut down or saturated
         * and so reject the task.
         */
        int c = ctl.get();
        // 如果当前运行的线程数小于核心线程数，创建新核心线程执行任务
        if (workerCountOf(c) < corePoolSize) {
            if (addWorker(command, true))
                return;
            c = ctl.get();
        }
        // 如果任务队列没满，将任务加入队列
        if (isRunning(c) && workQueue.offer(command)) {
            int recheck = ctl.get();
            if (! isRunning(recheck) && remove(command))
                reject(command);
            else if (workerCountOf(recheck) == 0)
                addWorker(null, false);
        }
        // 如果当前运行的线程数小于最大线程数，创建非核心线程执行任务
        else if (!addWorker(command, false))
            // 如果任务队列满了，且当前运行的线程数等于最大线程数，执行拒绝策略
            reject(command);
    }
```

调用 `ThreadPoolExecutor.shutdown()` 方法后，不可以新增任务，已有任务会继续执行，执行完后关闭线程池。