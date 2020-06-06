# AQS模型

## 概念

- 资源state：acquire和release来控制state的增减，state > 0时代表资源被占用；0被释放
- 锁状态：独占锁和共享锁

## CLH队列

队列构成：非逻辑队列，由CAS来控制内存状态

- head：空结点 `new Node()` ;对应headOffset的内存地址
- tail：初始等于head，后续为新结点；对应tailOffset的内存地址
- 双指针：
  - prev：前置结点
  - next：后继结点
- waitStatus：对应Node的结点状态

#### Node

结点模式：

- SHARED：共享
- EXCLUSIVE：排他

结点状态：

- 0：初始化状态
- CANCELLED =  1：线程被cancelled
- SIGNAL = -1：结点的继任节点被阻塞了，到时需要通知
- CONDITION = -2：线程正在等待condition
- PROPAGATE = -3：仅在SHARED模式生效，锁的获取会传播给后续的节点

## acquire

![输入图片说明](https://static.oschina.net/uploads/img/201511/19151122_tpXi.jpg)

```Java
/**
* 1. tryAcquire:尝试获取exclusive mode下的对象
* 2. addWaiter：添加新结点至队尾
* 3. acquireQueued：尝试获取锁
*/ 
if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
```

【addWaiter】

```java
Node node = new Node(Thread.currentThread(), mode);
// Try the fast path of enq; backup to full enq on failure
Node pred = tail;
// 如果当前队尾存在结点，则直接入队并返回
if (pred != null) {
	node.prev = pred;
	if (compareAndSetTail(pred, node)) {
		pred.next = node;
		return node;
  }
}
// 执行入队操作
enq(node);
return node;
```

【acquireQueued】

```Java
boolean interrupted = false;
for(;;) {
  // 获取前置结点
  final Node p = node.predecessor();
  // 如果前面没有人等待获取锁，则尝试获取
  if (p == head && tryAcquire(arg)) {
    // 将当前线程设置为头结点
  	setHead(node);
    // help gc
    p.next = null;
    return interrupted;
  }
  // 如果允许被挂起，则执行挂起操作：parkAndCheckInterrupt
  if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt()) {
    interrupted = true;
  }
}
```

【shouldParkAfterFailedAcquire】

```Java
int ws = pred.waitStatus;
if (ws == Node.SIGNAL)
  return true;
// 跳过被cancelled的结点
if (ws > 0) {
	do {
		node.prev = pred = pred.prev;
  } while (pred.waitStatus > 0);
  pred.next = node;	
}else {
  // 重置状态为SIGNAL
  compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
}
return false;
```



## release

```java
// 尝试释放锁
if (tryRelease(arg)) {
  Node h = head;
  // 头结点不为空，且不是初始化状态
  if(h != null && h.waitStatus != 0) {
    // 唤醒后继结点
    unparkSuccessor(h);
  }
  return true;
}
return false;
```

【unparkSuccessor】

```java
int ws = node.waitStatus;
// 将状态置为0
if (ws < 0)
  compareAndSetWaitStatus(node, ws, 0);

// 找到后继结点：需要跳过cancelled的结点
Node s = findNextNodeAndSkipCancelled(node)
if(s != null) 
  LockSupport.unpark(s.thread);
```



## 问题

【公平与非公平】

- 公平：必须判断是头结点才允许获取锁
- 非公平：只要能获取到资源就获取锁

【为什么是双向队列】

1. 插入效率
2. 唤醒后继者时需要判断当前节点的前置节点是否是头结点
3. 前驱节点遍历是线程安全的



# 使用

## ReentrantLock

可重入锁：同一个线程可重入多次，通过传入的arg来计数，当且仅当state=0该线程才会释放锁

- 主要实现tryAcquire和tryRelease方法，判断资源state是否可以被获取

## ReentrantReadWriteLock

读写锁：写锁-排它锁，读锁-共享锁

资源：高16位--读状态，低16位--写状态

- 重入性：同一个线程在写或读时可重入
- 共享：多个线程可共享读锁
- 锁降级：已获取写锁，获取读锁再释放写锁次序；写锁能降级为读锁
- 读时不可写：当存在读锁时，无法获取写锁。【为了保障写锁能让读锁可见。这一步也导致了当存在大量读时，会导致写饥饿，即获取不到写锁】

## StampedLock

增强读写锁：ReentrantReadWriteLock的升级版，为了解决读时不可写的问题。

- 非AQS实现，使用64位state来控制
- 基于版本号来进行乐观读



### state状态实现

- 0-6位：读锁计数，当超出RFULL（126）时，用readerOverflow作为读锁计数。当获取到读锁时，state加RUINT(值为1)；当释放读锁时，state减去RUINT。
- 第7位：写锁标识，1=写锁，0=读锁。当线程获取写锁或释放写锁时，都会将state加WBIT
- 8-64位：初始第8位=1，写锁变更时会+1



### 乐观读锁

示例：依赖方法--tryOptimisticRead

```java
double x , y;
void optimisticReadDemo() {
  // 获取当前锁的版本号
  long stampVersion = lock.tryOptimisticRead();
  // 将当前变量值copy到工作线程
  double currentX = x , currentY = y;
  // 判断刚才获取到的锁的版本号和实际版本号是否一致
  // validate这里插入了内存屏障，防止指令重排序
  if(!lock.validate(stampVersion)) {
    // 获取读锁，此时进入悲观读状态
    stampVersion = lock.readLock();
    try {
      currentX = x;
      currentY = y;
    }finally {
      // 释放读锁
      lock.unlockRead(stampVersion);
    }
  }
  return currentX , currentY;
}
```



### 变量

```java
//获取CPU的可用线程数量，用于确定自旋的时候循环次数
private static final int NCPU = Runtime.getRuntime().availableProcessors();

//根据NCPU确定自旋的次数限制(并不是一定这么多次，因为实际代码中是随机的)
private static final int SPINS = (NCPU > 1) ? 1 << 6 : 0;

//头节点上的自旋次数
private static final int HEAD_SPINS = (NCPU > 1) ? 1 << 10 : 0;

//头节点上的最大自旋次数
private static final int MAX_HEAD_SPINS = (NCPU > 1) ? 1 << 16 : 0;

//等待自旋锁溢出的周期数
private static final int OVERFLOW_YIELD_RATE = 7; // must be power 2 - 1

//在溢出之前读线程计数用到的bit位数
private static final int LG_READERS = 7;

//一个读状态单位：0000 0000 0001
private static final long RUNIT = 1L;
//一个写状态单位：0000 1000 0000
private static final long WBIT  = 1L << LG_READERS;
//读状态标识：0000 0111 1111
private static final long RBITS = WBIT - 1L;
//读锁计数的最大值：0000 0111 1110
private static final long RFULL = RBITS - 1L;
//读线程个数和写线程个数的掩码：0000 1111 1111
private static final long ABITS = RBITS | WBIT;
//// 读线程个数的反数，高25位全部为1:1111 1111 1111 1111 1111 1111 1111 1111 1111 1111 1111 1111 1111 1111 1000 0000
private static final long SBITS = ~RBITS; // note overlap with ABITS

// state的初始值
private static final long ORIGIN = WBIT << 1;

// 中断标识
private static final long INTERRUPTED = 1L;

// 节点状态值，等待/取消
private static final int WAITING   = -1;
private static final int CANCELLED =  1;

// 节点模式，读模式/写模式
private static final int RMODE = 0;
private static final int WMODE = 1;


//等待队列的头节点
private transient volatile WNode whead;
//等待队列的尾节点
private transient volatile WNode wtail;

// 锁状态
private transient volatile long state;
////因为读状态只有7位很小，所以当超过了128之后将使用此变量来记录
private transient int readerOverflow;
```





# FYI

- [聊聊并发（十二）—AQS分析](https://my.oschina.net/xianggao/blog/532709)
- [线程安全实现与CLH队列](https://www.jianshu.com/p/0f6d3530d46b)
- [锁原理 - AQS 源码分析：有了 synchronized 为什么还要重复造轮子](https://www.cnblogs.com/binarylei/p/12555166.html)
- [java锁（7）改进读写锁StampedLock详解](https://www.jianshu.com/p/8b47af35e6ce)