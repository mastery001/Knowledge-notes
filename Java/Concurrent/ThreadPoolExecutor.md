[TOC]

# 核心参数

```java
// Integer.SIZE = 32
private static final int COUNT_BITS = Integer.SIZE - 3;
    
// 2^31-1 
private static final int CAPACITY   = (1 << COUNT_BITS) - 1;
    
// runState is stored in the high-order bits
// 允许接入新的task或处理workQueue中的task
private static final int RUNNING    = -1 << COUNT_BITS;
// 不接受新的task，但是能处理workQueue中的task
private static final int SHUTDOWN   =  0 << COUNT_BITS;
// 不接受新task，也不处理workQueue中的task，同时还会interrupt当前运行的task
private static final int STOP       =  1 << COUNT_BITS;
// 当状态变为TIDYING将会调起terminated()方法：所有task都被中断，workerCount=0；
private static final int TIDYING    =  2 << COUNT_BITS;
// terminated()方法执行完成
private static final int TERMINATED =  3 << COUNT_BITS;

/**
1. runState：高三位来代表线程运行状态，runState
2. workCount:低28位来表示工作者个数
*/
private static int runStateOf(int c)     { return c & ~CAPACITY; }
private static int workerCountOf(int c)  { return c & CAPACITY; }
private static int ctlOf(int rs, int wc) { return rs | wc; }

// 阻塞工作队列，当线程池运行的线程数满了，则加入到等待队列
private final BlockingQueue<Runnable> workQueue;
```

## 线程池状态

- `RUNNING = -1 << COUNT_BITS`：允许接入新的task或处理workQueue中的task
- `SHUTDOWN = 0 << COUNT_BITS`：不接受新的task，但是能处理workQueue中的task
- `STOP = 1 << COUNT_BITS`：不接受新task，也不处理workQueue中的task，同时还会interrupt当前运行的task
- `TIDYING = 2 << COUNT_BITS`：当状态变为TIDYING将会调起terminated()方法：所有task都被中断，workerCount=0；
- `TERMINATED = 3 << COUNT_BITS`：terminated()方法执行完成

TERMINATED > TIDYING > STOP > SHUTDOWN > RUNNING

状态转换过程：
- `RUNNING -> SHUTDOWN`：shutdown()方法被调用时
- `(RUNNING or SHUTDOWN) -> STOP`：shutdownNow()方法被调用时
- `SHUTDOWN -> TIDYING`：当workQueue为空且线程池的运行线程为0时
- `STOP -> TIDYING`：当线程池的运行线程为0时
- `TIDYING -> TERMINATED`：当terminated()方法执行完成




# execute

```java
public void execute() {
    // 获取当前状态
    int c = ctl.get();

    // 当前工作线程数小于线程池大小时，创建一个新的线程运行
    if (workerCountOf(c) < corePoolSize) {
        if (addWorker(command, true))
            return;
        c = ctl.get();
    }
    // 当前状态处于RUNNING，且队列还没满
    if (isRunning(c) && workQueue.offer(command)) {
        // 这里再次校验状态，是为了保证如果此时是SHUTDOWN状态时，不允许新任务提交，要从workQueue中移除
        int recheck = ctl.get();
        // remove一般不可能失败，除非是该command被队列拉取出来执行去了
        if (! isRunning(recheck) && remove(command))
            reject(command);
        else if (workerCountOf(recheck) == 0)
            // 如果当前没有worker，则启动一个worker去拉取workerQueue中的task执行
            addWorker(null, false);
        }
    	// 如果任务队列加不进去，尝试再生成一个worker；如果还是失败，则代表线程池已满或已经关闭。
        else if (!addWorker(command, false))
            reject(command);
}

/**
* true：当线程池状态且小于线程池最大限制时，创建一个新的线程并提交并start
* false：当线程池在stop或shutdown状态时；或ThreadFactory创建线程失败
*/
private boolean addWorker(Runnable firstTask, boolean core) {
    retry:
    for (;;) {
        // 获取当前状态
        int c = ctl.get();
        // 当前运行状态
        int rs = runStateOf(c);

        // Check if queue empty only if necessary.
        /**
        * 两个条件做交集：
        * 1. 状态大于SHUTDOWN，即状态等于SHUTDOWN、STOP、TIDYING、TERMINATED
        * 2. !(状态=SHUTDOWN&提交的task为空&等待队列不为空)：为什么要判断这个？
        * 	我们知道，当状态处于SHUTDOWN时，线程池还能处理workQueue中的task，括号里的判断是确认是否还有线程需要执行，如果还需要处理，则可以添加worker
        */
        if (rs >= SHUTDOWN &&
            ! (rs == SHUTDOWN &&
               firstTask == null &&
               ! workQueue.isEmpty()))
            return false;

        
        for (;;) {
            // 判断workCount是否超过最大值
            int wc = workerCountOf(c);
            if (wc >= CAPACITY ||
                wc >= (core ? corePoolSize : maximumPoolSize))
                return false;
            // cas对workCount+1
            if (compareAndIncrementWorkerCount(c))
                break retry;
            // 判断状态是否改变，如果改变意味着此时不处于RUNNING状态，则走外层循环
            c = ctl.get();  // Re-read ctl
            if (runStateOf(c) != rs)
                continue retry;
            // else CAS failed due to workerCount change; retry inner loop
        }
    }

    boolean workerStarted = false;
    boolean workerAdded = false;
    Worker w = null;
    try {
        w = new Worker(firstTask);
        final Thread t = w.thread;
        if (t != null) {
            // 使用可重入锁保证workers的线程安全
            final ReentrantLock mainLock = this.mainLock;
            mainLock.lock();
            try {
                // Recheck while holding lock.
                // Back out on ThreadFactory failure or if
                // shut down before lock acquired.
                int rs = runStateOf(ctl.get());

                if (rs < SHUTDOWN ||
                    (rs == SHUTDOWN && firstTask == null)) {
                     // precheck that t is startable
                    ....
                    workers.add(w);
                    ...
                    workerAdded = true;
                }
            } finally {
                mainLock.unlock();
            }
            if (workerAdded) {
                t.start();
                workerStarted = true;
            }
        }
    } finally {
        if (! workerStarted)
            // 移除新建的worker，并尝试根据校验状态执行tryTerminate();方法 
            addWorkerFailed(w);
        	
    }
    return workerStarted;
}
```



# Worker#run

实际是调用`ThreadPoolExecutor#runWorker`方法

```java
final void runWokrer(Worker w) {
    ...
    // allow interrupts
    w.unlock();   
    boolean completedAbruptly = true;
    try {
        // getTask方法是从workQueue中take出task
        while(w.task != null || (task = getTask()) != null) {
            w.lock();
            /** 状态检测是否需要执行interrupt()方法：
            *   条件：是否处于STOP || 当处于STOP状态时线程是否被interrupted && 线程未被interrupted 
            * 	原因：处于STOP时，线程必须被中断；第二个再次判断STOP的原因是因为当执行shutdownNow方法时，会立刻转换为STOP；第三个条件是为了确认此时线程shutdownNow()尚未开始中断此线程，由自己来中断
            */
            ....
            // 执行run方法
            task.run()
        }
        completedAbruptly = false;
    }finally {
       processWorkerExit(w, completedAbruptly); 
    }
}
```

