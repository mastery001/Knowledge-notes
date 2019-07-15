[TOC]

# 限流算法

# 令牌桶算法

> 令牌桶算法最初来源于计算机网络。在网络传输数据时，为了防止网络拥塞，需限制流出网络的流量，使流量以比较均匀的速度向外发送。
>
> 令牌桶算法是网络流量整形（Traffic Shaping）和速率限制（Rate Limiting）中最常使用的一种算法。典型情况下，令牌桶算法用来控制发送到网络上的数据的数目，并允许突发数据的发送。
>
> 原理：大小固定的令牌桶可自行以恒定的速率源源不断地产生令牌。如果令牌不被消耗，或者被消耗的速度小于产生的速度，令牌就会不断地增多，直到把桶填满。后面再产生的令牌就会从桶中溢出。最后桶中可以保存的最大令牌数永远不会超过桶的大小。

**算法过程**

![](https://ws2.sinaimg.cn/large/006tNc79gy1fsxwckns0zj310k0s2go4.jpg)

算法描述：

- 假如用户配置的平均发送速率为r，则每隔1/r秒一个令牌被加入到桶中（每秒会有r个令牌放入桶中）；
- 假设桶中最多可以存放b个令牌。如果令牌到达时令牌桶已经满了，那么这个令牌会被丢弃；
- 当一个n个字节的数据包到达时，就从令牌桶中删除n个令牌（不同大小的数据包，消耗的令牌数量不一样），并且数据包被发送到网络；
- 如果令牌桶中少于n个令牌，那么不会删除令牌，并且认为这个数据包在流量限制之外（n个字节，需要n个令牌。该数据包将被缓存或丢弃）；
- 算法允许最长b个字节的突发，但从长期运行结果看，数据包的速率被限制成常量r。对于在流量限制外的数据包可以以不同的方式处理：（1）它们可以被丢弃；（2）它们可以排放在队列中以便当令牌桶中累积了足够多的令牌时再传输；（3）它们可以继续发送，但需要做特殊标记，网络过载的时候将这些特殊标记的包丢弃。

**注意：**

令牌桶算法不能与另外一种常见算法**漏桶算法**相混淆。这两种算法的主要区别在于漏桶算法能够**强行限制数据的传输速率**，而令牌桶算法在能够**限制数据的平均传输速率**外，还允许某种程度的**突发传输**。在令牌桶算法中，只要令牌桶中存在令牌，那么就允许突发地传输数据直到达到用户配置的门限，因此它适合于具有突发特性的流量。

# RateLimiter实现

## SmoothRateLimiter核心参数

```Java
  /**
   * The currently stored permits. 当前存储的permits
   */
  double storedPermits;

  /**
   * The maximum number of stored permits. 最大可存储permits
   */
  double maxPermits;

  /**
   * The interval between two unit requests, at our stable rate. E.g., a stable rate of 5 permits per second has a stable interval of 200ms.
   * 两个请求单元的的请求间隔，e.g 当permitsPerSecond=5时，那么请求间隔为200ms
   */
  double stableIntervalMicros;

  /**
   * The time when the next request (no matter its size) will be granted. After granting a
   * request, this is pushed further in the future. Large requests push this further than small requests.
   * 下一次请求将能获取的时间；允许某次请求拿走超出剩余令牌数的令牌，但是下一次请求将为此付出代价，一直等到令牌亏空补上，并且桶中有足够本次请求使用的令牌为止
   */
  private long nextFreeTicketMicros = 0L; // could be either in the past or future
```



## 创建过程

```java
// SleepingStopwatch.createFromSystemTimer()默认的计时器 纳秒级别
// permitsPerSecond为1秒内的令牌数
return create(SleepingStopwatch.createFromSystemTimer(), permitsPerSecond);

/************** method: RateLimiter.create **************/
// SmoothBursty：令牌桶实现  maxBurstSeconds:当限流器未使用时最大可存放的令牌数
RateLimiter rateLimiter = new SmoothBursty(stopwatch, 1.0 /* maxBurstSeconds */);
// 设置速率
rateLimiter.setRate(permitsPerSecond);
return rateLimiter;

/************** method: RateLimiter.setRate **************/
// 校验参数，速率必须大于0，且不能为NaN= 0.0/0.0
checkArgument(permitsPerSecond > 0.0 && !Double.isNaN(permitsPerSecond), "rate must be positive");
// 全局同步锁，Object对象
synchronized (mutex()) {
   // 子类设置rate
   doSetRate(permitsPerSecond, stopwatch.readMicros());
}

/************** method: SmoothRateLimiter.doSetRate **************/
// 更新storedPermits和nextFreeTicketMicros时间
// 初始时storedPermits为0.0 ；nextFreeTicketMicros为当前过去的毫秒数
resync(nowMicros);
// 请求间隔毫秒数
double stableIntervalMicros = SECONDS.toMicros(1L) / permitsPerSecond;
this.stableIntervalMicros = stableIntervalMicros;
// 见SmoothBursty.doSetRate
doSetRate(permitsPerSecond, stableIntervalMicros);

/************** method: SmoothRateLimiter.resync **************/
// if nextFreeTicket is in the past, resync to now
// 如果nextFreeTicketMicros的时间已经被获取，则同步到当前时间，
// 以coolDownIntervalMicros速率进行添加令牌
if (nowMicros > nextFreeTicketMicros) {
    // coolDownIntervalMicros当SmoothBursty时返回stableIntervalMicros
	storedPermits = min(maxPermits,
		storedPermits + (nowMicros - nextFreeTicketMicros) / coolDownIntervalMicros());
    nextFreeTicketMicros = nowMicros;
}

/************** method: SmoothBursty.doSetRate **************/
// maxPermits初始时为0.0
double oldMaxPermits = this.maxPermits;
// 最大令牌数 = 每秒可存放的最大令牌数 * 速率
maxPermits = maxBurstSeconds * permitsPerSecond;
// storedPermits 为当前存放的令牌数
if (oldMaxPermits == Double.POSITIVE_INFINITY) {
    // if we don't special-case this, we would get storedPermits == NaN, below
	storedPermits = maxPermits;
} else {
    // 计算重新设置速率时的storedPermits 为什么这样计算暂时不是很明白
	storedPermits = (oldMaxPermits == 0.0) 
        ? 0.0 // initial state
        : storedPermits * maxPermits / oldMaxPermits;
}
```

## acquire过程

```Java
// 默认获取一个permits
/************** method: RateLimiter.acquire **************/
// 保留permits下获取等待的时长
//reserve操作：permits须大于0，加锁synchronized (mutex())后调用reserveAndGetWaitLength
long microsToWait = reserve(permits);   
// 当microsToWait>0时执行sleep方法
stopwatch.sleepMicrosUninterruptibly(microsToWait);
// 返回sleep的时间--即获取令牌所消耗的时间
return 1.0 * microsToWait / SECONDS.toMicros(1L);

/************** method: RateLimiter.reserveAndGetWaitLength **************/
// 核心操作，获取permits可使用的时间 实现见SmoothRateLimiter
// nowMicros为从RateLimiter#start方法调用时间到当前所过去的毫秒数
long momentAvailable = reserveEarliestAvailable(permits, nowMicros);
return max(momentAvailable - nowMicros, 0);

/************** method: SmoothRateLimiter.reserveEarliestAvailable **************/
// 同步更新令牌数
resync(nowMicros);
long returnValue = nextFreeTicketMicros;
// 从请求的permits和当前存放的permits选中最小值
double storedPermitsToSpend = min(requiredPermits, this.storedPermits);
double freshPermits = requiredPermits - storedPermitsToSpend;
// storedPermitsToWaitTime方法当为SmoothBursty实现时默认为0
// 当令牌不足时计算需要等待的时间
long waitMicros = storedPermitsToWaitTime(this.storedPermits, storedPermitsToSpend)
        + (long) (freshPermits * stableIntervalMicros);
try {
    // 计算下一次请求获取的时间，这里是做了权衡，当令牌不足被提前使用时，下一次请求获取令牌的时间将被延长
	this.nextFreeTicketMicros = LongMath.checkedAdd(nextFreeTicketMicros, waitMicros);
} catch (ArithmeticException e) {
	this.nextFreeTicketMicros = Long.MAX_VALUE;
}
// 减去消耗的令牌数
this.storedPermits -= storedPermitsToSpend;
return returnValue;

```



# FYI

- [令牌桶算法限流](https://blog.csdn.net/SunnyYoona/article/details/51228456)