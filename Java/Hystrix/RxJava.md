# 观察者类型

被观察者通过`subscribe`方法订阅观察者，一个被观察者可以订阅多个观察者

方法介绍：

1. onSubscribe：订阅观察者时的操作
2. onNext
3. onError：报错时
4. onComplete：
5. onSuccess：执行成功时
6. lift：链接下一个被观察者，通过第一个被观察者的调用行为来调用操作，所以第一次被观察者显示调用上述5个方法的行为很重要

| 观察者              | 被观察者                             | 备注                                                     |
| ------------------- | ------------------------------------ | -------------------------------------------------------- |
| FlowableSubscriber  | Flowable — FlowableOnSubscribe       | 所有方法                                                 |
| Observer            | Observable —  ObservableOnSubscribe  | 所有方法                                                 |
| SingleObserver      | Single — SingleOnSubscribe           | 只包含1,3,5方法                                          |
| CompletableObserver | Completable — CompletableOnSubscribe | 只包含1,4,5方法                                          |
| MaybeObserver       | Maybe — MaybeOnSubscribe             | 类似于Single和Completable的混合体，执行success或complete |

## Actions

subscribe用很多重载方法，用于简化观察者，其都属于Actions

- **Action**：无参数类型
- **Consumer**<T>：单一参数类型
- **BiConsumer<T1, T2>**:双参数类型
- **Consumer<Obejct[]>**:多参数类型

## Observable和Flowable区别

Observable和Flowable其中前者不需要背压（BackPressure）参数和请求资源(request)操作。Observable不支持背压，而Flowable支持背压（背压是什么？后面再说，先明白区别）。关键是什么时候用呢，下面根据官方的建议：

使用Observable - 不超过1000个元素、随着时间流逝基本不会出现OOM - GUI事件或者1000Hz频率以下的元素 - 平台不支持Java Steam(Java8新特性) - Observable开销比Flowable低

使用Flowable - 超过10k+的元素(可以知道上限) - 读取硬盘操作（可以指定读取多少行） - 通过JDBC读取数据库 - 网络（流）IO操作

## 背压

背压就是**生产者（被观察者）的生产速度大于消费者（观察者）消费速度**从而导致的问题。

举一个简单点的例子，如果被观察者快速发送消息，但是观察者处理消息的很缓慢，如果没有特定的流（Flow）控制，就会导致大量消息积压占用系统资源，最终导致十分缓慢。怎么优化和减少这种情况后面再探讨，不过可以注意到，Flowable创建的时候已经设置了BackpressureStrategy，而且Subscriber使用了request来控制最大的流量。

# FYI

- [RxJava2.0学习文档](https://maxwell-nc.github.io/android/rxjava2-1.html)
- [RxJava Doc](http://reactivex.io/RxJava/javadoc/)
- [RxJava 2 - Tutorial](http://www.vogella.com/tutorials/RxJava/article.html)
- [RxJava-CSDN](https://blog.csdn.net/xx326664162/article/details/52068014)