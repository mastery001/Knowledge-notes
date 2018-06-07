Command类型

- HystrixCommand
- HystrixCollapser

执行类型与执行对应表

| Invokable        | 场景                 |
| ---------------- | -------------------- |
| CommandCollapser | HystrixCollapser注解 |
| CommandCollapser | 执行类型是OBSERVABLE |
| GenericCommand   | 默认                 |

- SYNCHRONOUS：同步
- ASYNCHRONOUS：异步，返回类型是Future
- OBSERVABLE：观察者，返回类型是Observable



核心类代码：

- HystrixCommandAspect:76
- AbstractCommand:336
- Hystrix:363
- ​

重点对象：断路器：**HystrixCircuitBreaker circuitBreaker** 

```
// key.name()相同时使用同一个断路器
protected HystrixCircuitBreakerImpl(HystrixCommandKey key, HystrixCommandGroupKey commandGroup, HystrixCommandProperties properties, HystrixCommandMetrics metrics) {
    this.properties = properties;
    this.metrics = metrics;
}
```

circuitOpen：断路器状态，默认为false，失败率过高时会开启断路器

配置参数类：

| 参数                                             | 备注                                                         |
| ------------------------------------------------ | ------------------------------------------------------------ |
| circuitBreakerForceOpen                          | 如果是true，则拒绝所有请求                                   |
| circuitBreakerForceClosed                        | 如果是true，则接受所有请求                                   |
| circuitBreakerRequestVolumeThreshold             | 最大请求数，默认为20                                         |
| circuitBreakerErrorThresholdPercentage           | 失败百分比，默认50%                                          |
| circuitBreakerSleepWindowInMilliseconds          | 当断路器开启，并且当前时间距失败关闭的时间窗口，默认为5000，在时间窗口内允许一个请求通过,当断路器开启时抛出异常`Hystrix circuit short-circuited and is OPEN` |
| executionIsolationStrategy                       | 隔离策略，两种THREAD(TryableSemaphoreNoOp.DEFAULT), SEMAPHORE(TryableSemaphoreActual)，默认为THREAD |
| executionIsolationSemaphoreMaxConcurrentRequests | 最大执行信号量，默认值为10，当信号量获取不到则抛出异常`could not acquire a semaphore for execution` , |
| executionIsolationThreadInterruptOnTimeout       | 默认为true，执行超时线程是否可中断                           |
| fallbackEnabled                                  | 默认为true，是否开启fallback                                 |
| fallbackIsolationSemaphoreMaxConcurrentRequests  | fallback执行最大并行个数，默认为10                           |
| executionTimeoutEnabled                          | 默认为true                                                   |
|                                                  |                                                              |
| executionIsolationThreadPoolKeyOverride          | 默认`hystrix.command.{key.name()}.threadPoolKeyOverride`     |
|                                                  |                                                              |
|                                                  |                                                              |

备忘TODO：getRunObservableDecoratedForMetricsAndErrorHandling



# FYI

- [hystrix使用总结](http://throwable.coding.me/2017/10/15/hystrix/)
- [Hystrix技术解析](https://www.jianshu.com/p/3e11ac385c73)

