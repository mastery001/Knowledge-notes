# 存储

- 数据结构：

  - 链表：hash、key、value、next

    - hash：取值可能

      - `MOVED`：`01`；扩容时的node状态

      - `TREEBIN`：`-2`；红黑树

      - `RESERVED`：`-3`；compute插入时的一种状态，先提前占位cas到tab中

      - `> 0`：正常的hash值

  - 红黑树：parent、left、right、prev、red

- 容量控制：

  - 默认：16
  - 指定：计算方式，`(long)(1.0 + (long)initialCapacity / loadFactor)`
    - concurrencyLevel`：最多并发update的线程数，如果initialCapacity小于该值，则需要使用该值

- 寻址方式：取模，`(tab.length - 1 ) & hash`

- 操作方式：cas操作，通过`((long)i << ASHIFT) + ABASE`定位
  - `ASHIFT`：数组元素的偏移地址
  - `ABASE`：数组的起始地址
- hash冲突：当发生hash冲突时
  - size<8时：单链表
  - size>=8时：红黑树

关键方法：

- **tableSizeFor**：找到离当前最近的2的N次幂(比当前值要大)
- **spread**：hash函数，高16位与低16位异或，并取正数
- **initTable**：初始化设定容量的Node数组
  - 巧用CAS和Thread.yield()来实现初始化，只有第一个线程能实例化，其他线程会进入自旋状态

# 操作

## put

1. 获取hash值，hash
2. 进入自旋
   1. tab == null，初始化initTable
   2. 如果该hash位置无元素，则cas写入【中断自旋】
   3. 如果tab处于resize阶段，则调用`helpTransfer`帮助迁移
   4. 锁住table[i]，操作链表和红黑树插入；如果链表长度大于8，则调用`treeifyBin`进化成红黑树【中断自旋】

## get

1. 获取hash值，hash
2. 定位当前元素tab[(n -1 ) & hash]=e
   1. 如果元素e为空，则返回null
   2. 如果存在元素e，eh=e.hash
      1. `eh = hash`：如果key相同，则直接返回
      2. `eh < 0`：通过e.find方法遍历
         1. 链表：不加锁
         2. 红黑树：遍历时可能加锁（需要平衡旋转时）
      3. 否则直接根据next遍历链表



# 常见问题

> 与1.7的区别

![img](https://img2018.cnblogs.com/blog/1415794/201907/1415794-20190706174520770-1254862721.png)

主要三大区别：

1. 1.7定位到数据要2次(Segment -> entry)，而1.8只要1次
2. 去除Segment结构(ReentrantLock,每次Segment都需要上锁)，而1.8直接使用数组，并通过cas等手段减少初始化或设置第一个元素时的锁
3. 添加红黑树的结构







