---
title: 线程池源码剖析
date: 2018-06-04 19:43:59
tags: Thread
---

### ThreadPoolExecutor 白话概念
**线程池里面有一个集合里面放的`Worker`对象 集合大小对应着核心线程数**  
1. 如果工作线程数量小于核心线程数  
2. 会创建任务放进集合,之后启动`Worker`线程  
3. 工作线程数不小于核心线程数,往队列里面放任务  
4. `Worker`线程启动之后,`worker`线程会调`runWorker()`方法  
5. 它会一直循环从队列里面取任务。然后调用它的`Run`方法取执行它。  
<!-- more -->


#### 线程池ThreadPoolExecutor分析   
##### 一、首先看`ThreadPoolExecutor#execute(Runnable)`提交任务方法
``` java
 int c = ctl.get();
        if (workerCountOf(c) < corePoolSize) {
            if (addWorker(command, true))
                return;
            c = ctl.get();
        }
        if (isRunning(c) && workQueue.offer(command)) {
            int recheck = ctl.get();
            if (! isRunning(recheck) && remove(command))
                reject(command);
            else if (workerCountOf(recheck) == 0)
                addWorker(null, false);
        }
        else if (!addWorker(command, false))
            reject(command);
```

1. 判断超过核心线程数没 没超过addWorker (Worker对象也很关键)
2. 创建任务或者workQuery.offer()队列里存放任务
3. 判断超出限制拒绝任务

##### 二、`ThreadPoolExecutor#addWorker(Runnable firstTask, boolean core)`添加工作线程方法
第1段 判断
``` java
retry:
        for (;;) {
            int c = ctl.get();
            int rs = runStateOf(c);
            for (;;) {
                int wc = workerCountOf(c);
                if (wc >= CAPACITY ||
                    wc >= (core ? corePoolSize : maximumPoolSize))
                    return false;
                if (compareAndIncrementWorkerCount(c))
                    break retry;
                c = ctl.get();  // Re-read ctl
                if (runStateOf(c) != rs)
                    continue retry;      
            }
        }
```
这段代码主要是为了判断当前线程池状态和限制数量，不正确进行重试，略过compareAndIncrementWorkerCount成功跳出最外层循环往下走。  
第2段 才是这个方法的核心部分
``` java
        boolean workerStarted = false;
        boolean workerAdded = false;
        Worker w = null;
        try {
            final ReentrantLock mainLock = this.mainLock;
            w = new Worker(firstTask);
            final Thread t = w.thread;
            if (t != null) {
                mainLock.lock();
                try {
                        workers.add(w);
                        int s = workers.size();
                        if (s > largestPoolSize)
                            largestPoolSize = s;
                        workerAdded = true;
                    }
                } finally {
                    mainLock.unlock();
                }
                if (workerAdded) {
                    t.start();      
                }
        return workerStarted;
```
ThreadPoolExecutor里面有一个Set<Worker> 集合。将要提交的任务Runnable放到Worker对象里 ，Worker对象本身是一个线程，然后启动它
##### 三、`Worker对象，本身也是一个对象`
``` java
private final class Worker extends AbstractQueuedSynchronizer implements Runnable
        final Thread thread;
        Runnable firstTask;
        volatile long completedTasks;
        Worker(Runnable firstTask) {
            setState(-1); // inhibit interrupts until runWorker
            this.firstTask = firstTask;
            this.thread = getThreadFactory().newThread(this);
        }
        public void run() {
            runWorker(this);
        }
        final void runWorker(Worker w) {
            Thread wt = Thread.currentThread();
            Runnable task = w.firstTask;
            w.firstTask = null;
            w.unlock(); // allow interrupts
            boolean completedAbruptly = true;
            while (task != null || (task = getTask()) != null) {
                w.lock();
                    beforeExecute(wt, task);
                        task.run();
                    w.completedTasks++;
                    w.unlock();
            }
    }
```
这段代码运行一个线程 死循环 `getTask()`取`Runnable`调用他的`Run()`方法 其中`getTask()`是关键;

##### 四、ThreadPoolExecutor#getTask() 从队列取任务方法
``` java
 boolean timedOut = false; // Did the last poll() time out?
        retry:
        for (;;) {
            int c = ctl.get();
            int rs = runStateOf(c);  
                Runnable r = timed ?
                    workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) :
                    workQueue.take();
                if (r != null)
              return r;          
        }
```
死循环从workQueue取Runnable任务返回



#### ThreadPoolExecutor ctl字段分析
``` java
    static final int COUNT_BITS = Integer.SIZE - 3;
    static final int CAPACITY   = (1 << COUNT_BITS) - 1;

    // runState is stored in the high-order bits
    static final int RUNNING    = -1 << COUNT_BITS;
    static final int SHUTDOWN   =  0 << COUNT_BITS;
    static final int STOP       =  1 << COUNT_BITS;
    static final int TIDYING    =  2 << COUNT_BITS;
    static final int TERMINATED =  3 << COUNT_BITS;

    static int runStateOf(int c)     { return c & ~CAPACITY; }
    static int workerCountOf(int c)  { return c & CAPACITY; }
    static int ctlOf(int rs, int wc) { return rs | wc; }
```

|线程状态|----|----|----|----|----|----|----|----|十进制数值|
|------|---|---|---|---|---|---|---|---|---|
|**RUNNING**   |1110|0000|0000|0000|0000|0000|0000|0000|-536870912|  
|**SHUTDOWN**  |0000|0000|0000|0000|0000|0000|0000|0000|0|
|**STOP**      |0010|0000|0000|0000|0000|0000|0000|0000|536870912|
|**TIDYING**   |0100|0000|0000|0000|0000|0000|0000|0000|1073741824|
|**TERMINATED**|0110|0000|0000|0000|0000|0000|0000|0000|1610612736|


|字段|----|----|----|----|----|----|----|----|十进制数值|
|------|---|---|---|---|---|---|---|---|---|
|**CAPACITY**  |0001|1111|1111|1111|1111|1111|1111|1111|536870911|
|**~CAPACITY** |1110|0000|0000|0000|0000|0000|0000|0000|-536870912|

由此二进制结果 分析
> runStateOf() c & ~CAPACITY      

相当于就是取前4位的值


> workerCountOf() c & CAPACITY   

相当就是取后面的值

ctl一个int型4个字节 4*8=32 位 里面存了两部分的值
前面4位存的是线程的5个状态,后面28位存的工作线程池的数量

``` java
int c = ctl.get();
int rs = runStateOf(c);
if (rs >= SHUTDOWN &&! (rs == SHUTDOWN &&firstTask == null &&! workQueue.isEmpty()));
```
这种`rs >= SHUTDOWN` 相当*SHUTDOWN*,*STOP*,*TIDYING*,*TERMINATED*这些字段是符合状态的。

<br>