# 概念

- 资源state：acquire和release来控制state的增减，state > 0时代表资源被占用；0被释放
- 锁状态：独占锁和共享锁

# CLH队列

队列构成：非逻辑队列，由CAS来控制内存状态

- head：空结点 `new Node()` ;对应headOffset的内存地址
- tail：初始等于head，后续为新结点；对应tailOffset的内存地址
- 双指针：
  - prev：前置结点
  - next：后继结点
- waitStatus：对应Node的结点状态

## Node

结点模式：

- SHARED：共享
- EXCLUSIVE：排他

结点状态：

- 0：初始化状态
- CANCELLED =  1：线程被cancelled
- SIGNAL = -1：结点的继任节点被阻塞了，到时需要通知
- CONDITION = -2：线程正在等待condition
- PROPAGATE = -3：仅在SHARED模式生效，锁的获取会传播给后续的节点

# acquire

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



# release

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



# 问题

【公平与非公平】

- 公平：必须判断是头结点才允许获取锁
- 非公平：只要能获取到资源就获取锁

【为什么是双向队列】

1. 插入效率
2. 唤醒后继者时需要判断当前节点的前置节点是否是头结点



# FYI

- [聊聊并发（十二）—AQS分析](https://my.oschina.net/xianggao/blog/532709)
- [线程安全实现与CLH队列](https://www.jianshu.com/p/0f6d3530d46b)