JAVA进程内存 = JVM进程内存+heap内存+ 永久代内存+ 本地方法栈内存+线程栈内存 +堆外内存 +socket 缓冲区内存+元空间 

linux内存和JAVA堆中的关系 

- RES = JAVA正在存活的内存对象大小 + 未回收的对象大小 + 其它 
- VIART= JAVA中申请的内存大小，即 -Xmx -Xms + 其它 
- 其它 = 永久代内存+ 本地方法栈内存+线程栈内存 +堆外内存 +socket 缓冲区内存 +JVM进程内存 



![BD805679-6487-429E-BAFF-6D6D8F771513](resources/Java内存分析/BD805679-6487-429E-BAFF-6D6D8F771513.png)



[Java应用Top命令RES内存占用高分析](https://www.jianshu.com/p/479a715d461e)

# NMD工具

```
jcmd pid VM.native_memory summary scale=MB
jcmd pid VM.native_memory detail scale=MB

# 设置基线
jcmd pid VM.native_memory baseline 

jcmd pid VM.native_memory detail.diff scale=MB
jcmd pid VM.native_memory summary.diff scale=MB
```

[追踪JVM中的本地内存](https://blog.csdn.net/Developlee/article/details/100691997)