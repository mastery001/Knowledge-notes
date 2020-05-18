# Node

节点模式：

- SHARED：共享
- EXCLUSIVE：排他

节点状态：

- 0：初始化状态
- CANCELLED =  1：线程被cancelled
- SIGNAL = -1：节点的继任节点被阻塞了，到时需要通知
- CONDITION = -2：线程正在等待condition
- PROPAGATE = -3：仅在SHARED模式生效，锁的获取会传播给后续的节点

# acqure

```Java
/**
* 1. tryAcquire:尝试获取exclusive mode下的对象
* 2. acquireQueued：
*/ 
if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
```

【acquireQueued】

```Java
boolean interrupted = false;
for(;;) {
  // 获取前置节点
  final Node p = node.predecessor();
  // 如果前面没有人等待获取锁，则尝试获取
  if (p == head && tryAcquire(arg)) {
    // 将当前线程设置为头结点
  	setHead(node);
    // help gc
    p.next = null;
  }
  // 如果允许被挂起，则执行挂起操作：parkAndCheckInterrupt；让出资源并立即返回
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
// 跳过被cancelled的节点
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



# FYI

- [聊聊并发（十二）—AQS分析](https://my.oschina.net/xianggao/blog/532709)
- [线程安全实现与CLH队列](https://www.jianshu.com/p/0f6d3530d46b)