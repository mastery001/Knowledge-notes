[TOC]

## 引言
本篇博客旨在突出体现JMM的基本作用，以及其特性；当然面试中也有很多朋友会遇到这个题目，简单了解下还是有助于找工作的。

## Java内存模型的抽象
>Java线程之间的通信由Java内存模型（本文简称为JMM）控制，JMM决定一个线程对共享变量的写入何时对另一个线程可见。

从抽象的角度来看，JMM定义了线程和主内存之间的抽象关系：线程之间的共享变量存储在主内存（main memory）中，每个线程都有一个私有的本地内存（local memory），本地内存中存储了该线程以读/写共享变量的副本。本地内存是JMM的一个抽象概念，并不真实存在。它涵盖了缓存，写缓冲区，寄存器以及其他的硬件和编译器优化

## 1. 重排序
为了程序能够更高效的运行，编译器和处理器都会对指令进行重排序；重排序分为以下三种类型：

1. 编译器优化的重排序
2. 指令级并行的重排序
3. 内存系统的重排序

**只要是重排序都有可能会导致多线程内出现内存可见性的问题**

### 1.1 内存屏障指令
为了保证内存可见性，java编译器在生成指令序列的适当位置会插入内存屏障指令来禁止特定类型的处理器重排序。JMM把内存屏障指令分为下列四类：

| 屏障类型 | 指令示例  | 说明 |
| --------- | --------- | --------- |
| LoadLoad Barriers |   Load1; LoadLoad; Load2  |  确保Load1数据的装载，之前于Load2及所有后续装载指令的装载。
| StoreStore Barriers   |   Store1; StoreStore; Store2  | 确保Store1数据对其他处理器可见（刷新到内存），之前于Store2及所有后续存储指令的存储。
| LoadStore Barriers  |  Load1; LoadStore; Store2  |  确保Load1数据装载，之前于Store2及所有后续的存储指令刷新到内存。
| StoreLoad Barriers  |  Store1; StoreLoad; Load2  |  确保Store1数据对其他处理器变得可见（指刷新到内存），之前于Load2及所有后续装载指令的装载。StoreLoad Barriers会使该屏障之前的所有内存访问指令（存储和装载指令）完成之后，才执行该屏障之后的内存访问指令。

### 1.2 happens-before
>如果一个操作执行的结果需要对另一个操作可见，那么这两个操作之间必须存在happens-before关系。这里提到的两个操作既可以是在一个线程之内，也可以是在不同线程之间。

happens-before规则如下：

- 程序顺序规则：一个线程中的每个操作，happens- before 于该线程中的任意后续操作。
- 监视器锁规则：对一个监视器锁的解锁，happens- before 于随后对这个监视器锁的加锁。
- volatile变量规则：对一个volatile域的写，happens- before 于任意后续对这个volatile域的读。
- 传递性：如果A happens- before B，且B happens- before C，那么A happens- before C。

**注意，两个操作之间具有happens-before关系，并不意味着前一个操作必须要在后一个操作之前执行！happens-before仅仅要求前一个操作（执行的结果）对后一个操作可见，且前一个操作按顺序排在第二个操作之前**

## 2. 顺序一致性模型
>JMM对正确同步的多线程程序，其执行将具有顺序一致性(sequentially consistent)---即程序的执行结果和在顺序一致性模型中得到的结果相同；（这里同步是指广义上的同步，包括同步原语(lock,volatile,final)的正确使用）

特性：

- 一个线程中所有操作都必须按照程序的顺序来执行
- 不管程序是否同步，所有线程都只能看到一个单一的操作执行顺序。在此模型下，每个操作都必须原子执行且立即对所有线程可见

## 内存语义
JMM还定义了volatile、锁、final的内存语义，具体细节已经有同仁总结的很好了，所以这里就不多阐述了。

附上链接：

- [volatile](http://www.infoq.com/cn/articles/java-memory-model-4)
- [锁](http://www.infoq.com/cn/articles/java-memory-model-5)
- [final](http://www.infoq.com/cn/articles/java-memory-model-6)

