[TOC]
# 主要特点

- 基于C/S架构，协议简单
- 基于libevent的事件处理
- 自主内存管理
- 基于客户端式的分布式实现

## libevent
libevent是一套跨平台的事件处理接口的封装，高性能的事件触发的网络库，内部支持poll、select(Windows)、epoll(Linux)、kquque(BSD)、/dev/pool(Solaris)

Memcached使用其进行网络并发连接的处理，能够保持在很大并发情况下，仍然能够保持快速的响应能力。

## 自主内存管理
包括两部分：

1. 数据存储方式：Slab Allocation
2. 数据过期方式：Lazy Expiration + LRU

### 数据存储方式
Slab Allocation的原理是按照预先设定的大小，将整体分配的内存分割成特定长度的块(chunk)，并将尺寸相同的块分成组(chunks)；

- Page：分配给Slab的内存空间，默认为1MB；Slab根据其大小切换成chunk
- Chunk：用于缓存记录的内存空间
- Slab Class：特定大小的chunk组

Memcached根据接收到数据的大小来决定使用哪个chunk；例如Slab Class存在50B，75B，100B等大小的chunk，进来一个80B的数据，则其会选择100B的chunk来将该数据缓存。【**缺陷：80B使用100B的chunk，则剩下的20B无法使用，浪费了**】

### 数据过期方式
1. Lazy Expiration
  当数据get时查看该记录的时间戳是否过期，过期则清除，这种方式为懒清除

2. LRU
  最近最少使用的缓存清除策略，大部分缓存默认采用这种策略，这里就不详细介绍

## 基于客户端的分布式
根据客户端自实现的负载均衡算法来实现，维护成本大

# 经验与技巧

Memcached Memcached一些特性和限制 

- 在 Memcached Memcached 中可以保存的item数据量是没有限制的，只有内存足够 数据量是没有限制的，只有内存足够
- Memcached Memcached单进程最大使用内存为 单进程最大使用内存为2G，要使用更多内存，可以分多个端口开启多个 ，要使用更多内存，可以分多个端口开启多个Memcached Memcached进程
- 最大30天的数据过期时间 天的数据过期时间, 设置为永久的也会在这个时间过期，常量 设置为永久的也会在这个时间过期，常量REALTIME_MAXDELTA REALTIME_MAXDELTA
   60*60*24*30 控制
- 最大键长为250字节，大于该长度无法存储，常量 字节，大于该长度无法存储，常量KEY_MAX_LENGTH 250 KEY_MAX_LENGTH 250 控制
- 单个item最大数据是1MB，超过1MB数据不予存储，常量 数据不予存储，常量POWER_BLOCK 1048576 POWER_BLOCK 1048576 进行控制，
   它是默认的slab大小
- 最大同时连接数是 最大同时连接数是200，通过 conn_init() conn_init()中的freetotal freetotal 进行控制，最大软连接数是 进行控制，最大软连接数是1024，通过
   settings.maxconns=1024 settings.maxconns=1024 进行控制
- 跟空间占用相关的参数： 跟空间占用相关的参数：settings.factor=1.25, settings.chunk_size=48, settings.factor=1.25, settings.chunk_size=48, 影响slab的数据占用和步进方式 的数据占用和步进方式

# 参考资料
- [Memcached原理](http://acsa.ustc.edu.cn/HPC2015/mellanox/download/memcached-%E5%8E%9F%E7%90%86.pdf)
- [libevent基础知识](http://blog.csdn.net/majianfei1023/article/details/46485705)