
# NIO+Reactor实现

底层依赖JDK1.4的NIO

1. AbstractMultiworkerIOReactor：相当于Netty的Boss；负责处理连接事件，只有一个线程
    - DefaultConnectingIOReactor：处理客户端的连接事件
    - DefaultListeningIOReactor：负责处理服务端的bind事件，接收客户端的连接
2. BaseIOReactor：相当于Netty的Worker线程，处理I/O事件，由AbstractMultiworkerIOReactor初始化，线程大小可配置

## 具体实现
- HttpAsyncRequestExecutor：处理IO事件，实际会调用DefaultClientExchangeHandlerImpl
- DefaultClientExchangeHandlerImpl：交换层，用于处理建连和转发回调callback结果；实际调用MainClientExec
- MainClientExec：具体的执行类；用于准备连接阶段和接收结果进行HttpProcess链式处理

## 连接池

三个连接维护者：

1. `leased::HashSet`:已用连接
2. `available::LinkedList`:可用连接链表，从链表首部取和归还连接
3. `pending::HashMap`:建连中的连接；加入到requestQueue中，有ConnectingIOReactor来进行建连，建连成功后，加入到leased或available中

两个链接状态管理者：

1. `leasingRequests::LinkedList`:请求中
    1. `validatePendingRequests`:连接池可主动触发检验是否有请求没有得到处理
    2. `processPendingRequests`：在某连接失效或被关闭的时候会调用唤醒处理
    3. `processNextPendingRequest`:在其他request的回调事件中，或连接归还阶段调用
2. `completedRequests::ConcurrentLinkedQueue`:请求完成，与`fireCallbacks`中处理已完成的请求，调用时机如下
    1. `lease`阶段：获取连接阶段
    2. `release`：归还连接阶段
    3. `validatePendingRequests`：主动调用
    4. `...`：其他request的回调事件中


## IO线程的注意事项

- 当处理IO的业务Handler出现异常，并且未捕获到时，IO线程会自动关闭
- 线程池单个target默认的连接最大个数为2，实现为CPool，如不指定大小，将会影响到实际的吞吐量