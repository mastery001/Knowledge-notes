# Safepoint

`Safepoint`是java代码中一个线程可能暂停执行的一个位置，SafePoint保存了其他位置没有的一些运行信息。在这个位置上保存了线程上下文的任何信息，包括对象或者非对象的内部指针。

## 工作原理

在HotSpot虚拟机中，safepoint 协议是主动协作的 。每一个用户线程在安全点 上都会检测一个标志位，来决定自己是否暂停执行。

对于JIT编译后的代码，JIT会在代码特定的位置(通常来说，在方法的返回处和counted loop的结束处)上插入安全点代码。对于解释执行的代码，JVM会设置一个2字节的`dispatch tables`,解释器执行的时候会经常去检查这个dispatch tables，当有safepoint请求的时候，就会让线程去进行safepoint检查。

# STW

## 什么是STW?

`Stop The World`是JVM等待所有用户线程进行`safepoint`并阻塞，做一些全局性的操作的行为。进行`STW`的原因可能是：

- Garbage collection pause(垃圾回收中断)
- Code Deoptimization
- Flushing code  cache
- Class redefinition(e.g. hot swap or instrumentation)
- Biased lock revocation（撤销偏向锁）
- Various debug operation (e.g. deadlock check or stacktrace dump)

更加详细的参考：[http://hg.openjdk.java.net/jdk7/jdk7/hotspot/file/9b0ca45cd756/src/share/vm/runtime/vm_operations.hpp](http://hg.openjdk.java.net/jdk7/jdk7/hotspot/file/9b0ca45cd756/src/share/vm/runtime/vm_operations.hpp)



## STW的四个阶段

根据-XX:+PrintSafepointStatistics –XX:PrintSafepointStatisticsCount=1 参数虚拟机打印的日志文件可以看出，safepoint的执行一共可以分为四个阶段：

- Spin阶段。因为jvm在决定进入全局safepoint的时候，有的线程在安全点上，而有的线程不在安全点上，这个阶段是等待未在安全点上的用户线程进入安全点。
- Block阶段。即使进入safepoint，用户线程这时候仍然是running状态，保证用户不在继续执行，需要将用户线程阻塞。[http://blog.csdn.net/iter_zc/article/details/41892567](https://link.zhihu.com/?target=http%3A//blog.csdn.net/iter_zc/article/details/41892567) 这篇bog详细说明了如何将用户线程阻塞。
- Cleanup。这个阶段是JVM做的一些内部的清理工作。
- VM Operation. JVM执行的一些全局性工作，例如GC,代码反优化。



## 相关参数



## XX:+PrintGCApplicationStoppedTime

```shell
2020-09-27T08:32:11.666+0800: 1080117.109: Total time for which application threads were stopped: 0.6995793 seconds, Stopping threads took: 0.3010033 seconds
```

- `1080117.109`:JVM启动后的秒数
- `0.6995793`：JVM发起STW开始到结束的时间
- `0.3010033`：线程进入safepoint的耗时

**如果是GC引发的STW，这条内容会紧挨着出现在GC log的下面**



# FYI

- [java GC进入safepoint的时间为什么会这么长？](https://www.zhihu.com/question/57722838)