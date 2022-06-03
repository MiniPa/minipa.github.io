---
layout: post
title: disruptor 高性能内存队列
comments: true,
categories: [Disruptor, Queue]
description: 高性能内存队列， Storm、Camel、Log4j2，以及我们自己的先目均用此来提升性能
keywords: Disruptor, Queue
topmost: true
---

## Java核心技术 思维导图 节点（Concurrency 并发 -> disruptor）
<iframe id="embed_dom" name="embed_dom" frameborder="0" style="display:block;width:525px; height:245px;" src="https://www.processon.com/embed/623b22e5e401fd070bbe3acd"></iframe>

## Disruptor
### 1.介绍
一个并发组件，能够在无锁情况下实现网络并发查询操作

[Disruptor Github](https://github.com/LMAX-Exchange/disruptor.git)  
[Dsiruptor 开发手册](https://lmax-exchange.github.io/disruptor/developer-guide/index.html)

### 2.核心结构设计

![disruptor](/images/types/cur/disruptor.png)

- Ring Buffer 环形缓冲区
曾经是Disruptor中的核心对象，不过从3.0版本开始，只负责对通过Disruptor进行交换的数据(事件)进行存储和更新
在一些高级使用中，Ring Buffer可以由用户自定义的来代替

- Sequence 
通过递增的序号管理进行交换的数据(事件)，对数据(事件)的处理过程总是沿着序号逐个递增处理的
一个Sequence用于跟踪标识某个特定的事件处理者(RingBuffer/Consumer)的进度
虽然可以使用AutomicLong标识进度，
但定义Sequence另一个目的是防止CPU缓存伪共享(Disruptor高性能关键点之一)

- Sequencer 真正核心
这个接口有两个实现类 SingleProducerSequence 和 MultiProducerSequence，它们定义生产者和消费者之间快速、正确地传递数据的并发算法

- Sequence Barrier
保持RingBuffer的main published Sequence和Consumer依赖的其它Sequence的引用
Sequence Barrier还定义决定Consumer是否还有可处理的事件逻辑

- Event
在Disruptor中，在生产者和消费者之间进行交换的数据称为事件(Event)
它不是Disruptor定义的类型，而是使用者自己定义并指定的(可以看作一个Bean)

- EventHandler
Disruptor定义的事件处理接口，由用户实现，用于处理事件，是Consumer的真正实现

- EventProcessor
EventProcessor 持有特定消费者(Consumer)的Sequence，
	并提供用来调用事件处理实现事件循环(Event loop)

- Wait Strategy
	定义Consumer如何等待下一个事件策略。(Disruptor提供了多种策略)

- Producer
生产者，泛指Disruptor发布事件的代码，Disruptor没有定义特定的接口或类型

### 2.高性能设计
- 1.在性能测试中发现竟然与I/O操作处于同样的数量级
- 2.一个线程每秒处理6百万订单/s
- 3.依据并发竞争的激烈程度的不同，Disruptor比ArrayBlockingQueue吞吐量快 4~7倍，平均延迟差了3个数量级  


#### 2.1 高性能设计
```java
1）环形数组：减少垃圾回收，数组对缓存机制更友好
为了避免垃圾回收，采用数组而非链表
同时，数组对处理器的缓存机制更加友好（占位符方式解决了伪共享问题）

2）元素定位：下标递增、位运算定位更快
数组长度2^n，通过位运算，加快定位的速度
下标采取递增的形式
不用担心 index 溢出的问题，index 是 long 类型，即使 100万QPS 的处理速度，也需要 30万年 才能用完

3）无锁设计：先申请位置，再进行处理
每个生产者或者消费者线程，会先申请可以操作的元素在数组中的位置，申请到之后，直接在该位置写入或者读取数据
```

##### 无锁设计
- 问题
```aidl
 问题1： 如何防止多个线程重复写同一个元素？
  Disruptor的解决方法是，每个线程获取不同的一段数组空间进行操作。
  这个通过CAS很容易达到。只需要在分配元素的时候，通过CAS判断一下这段空间是否已经分配出去即可。

问题2：如何防止读取的时候，读到还未写的元素？
Disruptor在多个生产者的情况下，引入了一个与Ring Buffer大小相同的buffer：available Buffer。当某个位置写入成功的时候，便把availble Buffer相应的位置置位，标记为写入成功。
读取的时候，会遍历available Buffer，来判断元素是否已经就绪
```

- 读取数据
```aidl
生产者多线程写入的情况会复杂很多：

1.申请读取到序号n
2.若 writer cursor >= n，这时仍然无法确定连续可读的最大下标。
	从reader cursor开始读取available Buffer，一直查到第一个不可用的元素，然后返回最大连续可读元素的位置
3.消费者读取元素
```
图示：
```aidl
读线程读到下标为2的元素，三个线程 Writer1/Writer2/Writer3 正在向RingBuffer相应位置写数据，写线程被分配到的最大元素下标是11
读线程申请读取到下标从 3到11的元素，判断writer cursor>=11
	然后开始读取availableBuffer，从3开始，往后读取，发现下标为7的元素没有生产成功，于是WaitFor(11)返回6
然后，消费者读取下标从3到6共计4个元素
```

![disruptor-read](/images/types/cur/disruptor-read.png)

- 写数据
```aidl
多个生产者写入的时候：

1.申请写入m个元素
2.若是有m个元素可以写入，则返回最大的序列号
	每个生产者会被分配一段独享的空间
3.生产者写入元素，写入元素的同时设置available Buffer里面相应的位置，以标记自己哪些位置是已经写入成功的。
```
图示：
```aidl
Writer1和Writer2两个线程写入数组，都申请可写的数组空间。
Writer1被分配了下标3到下表5的空间，Writer2被分配了下标6到下标9的空间。

Writer1写入下标3位置的元素，同时把available Buffer相应位置置位，标记已经写入成功，往后移一位，开始写下标4位置的元素。Writer2同样的方式。最终都写入完成。
```
![disruptor-write](/images/types/cur/disruptor-write.png)

##### WaitStrategy 等待策略
```aidl
BlockingWaitStrategy: 为等待屏障的EventProcessors使用一个 Lock 和 condition variable   ==> 优先吞吐量
	当吞吐量和低延迟不像CPU资源那么重要时，可以使用此策略
LiteBlockingWaitStrategy: BlockingWaitStrategy的变体，它试图在锁非争用时忽略条件唤醒
	显示微基准测试的性能改进
	然而，这种等待策略应该被认为是实验性的，因为我还没有完全证明锁省略代码的正确性


BusySpinWaitStrategy; 为等待barrier的事件处理器使用一个繁忙的旋转循环
	这个策略将使用CPU资源来避免可能带来延迟抖动的系统调用。当线程可以绑定到特定的CPU核时，最好使用它


TimeoutBlockingWaitStrategy: 超时阻塞策略
LiteTimeoutBlockingWaitStrategy: TimeoutBlockingWaitStrategy的变体，它试图在锁不竞争时忽略条件唤醒。 实现接口
	

PhasedBackoffWaitStrategy: 分阶段等待策略，用于在barrier上等待EventProcessors
	当吞吐量和低延迟不像CPU资源那么重要时，可以使用此策略。
	旋转，然后生成，然后使用配置的回退WaitStrategy等待

SleepingWaitStrategy: 睡眠策略
	最初是旋转的，然后使用Thread.yield()，最后睡眠(locksupport - parknanos (n))，以获得操作系统和JVM在eventprocessor等待barrier时允许的最小nanos数。 
	这种策略是性能和CPU资源之间的一种很好的折衷。
	潜伏期峰值可能发生在平静期之后。
	它还将减少对生成线程的影响，因为它不需要向任何条件变量发出信号来唤醒事件处理线程。


YieldingWaitStrategy: yield策略，使用Thread.yield()来处理在初始旋转后等待屏障的事件处理器。 
	这种策略将使用100%的CPU，但如果其他线程需要CPU资源，则这种策略比繁忙的旋转策略更容易放弃CPU
```


### 3.应用
- log4j-2 对比 log4j 升级了 disruptor：
  - 单线程情况下，loggers all async与Async Appender吞吐量相差不大
  - 在64个线程的时候，loggers all async的吞吐量比Async Appender增加了12倍，是Sync模式的68倍
 
## 4.代码分析与案例
### 4.1 Disruptor 核心对象
```java
public class Disruptor<T>
{
    private final RingBuffer<T> ringBuffer; 
    private final Executor executor;
    private final ConsumerRepository<T> consumerRepository = new ConsumerRepository<>();
    private final AtomicBoolean started = new AtomicBoolean(false);
// ...
}
```
![disruptor-java](/images/types/cur/java-disruptor.png)

- one example
```java
@Data
@Slf4j
public class TurtleDisruptor {

    public static final String INIT_METHOD = "start";
    public static final String DESTROY_METHOD = "stop";

    private Disruptor<TurtleEvent> disruptor;

    private ApplicationContext applicationContext;

    public TurtleDisruptor(ApplicationContext applicationContext) {
        this.applicationContext = applicationContext;
    }

    public void start() {
        TurtleEventFactory eventFactory = new TurtleEventFactory();
        int ringBufferSize = 2 << 24;  // 1024 * 1024 * 32
        disruptor = new Disruptor<>(eventFactory, ringBufferSize, new DisruptorThreadFactory(), ProducerType.SINGLE, new BlockingWaitStrategy());

        TurtleEventHandler[] eventHandlers = new TurtleEventHandler[1];
        for (int i = 0; i < eventHandlers.length; i++) {
            eventHandlers[i] = applicationContext.getBean(TurtleEventHandler.class);
        }
        disruptor.handleEventsWithWorkerPool(eventHandlers).then(new CleanEventHandler());
        disruptor.start();
        log.info("Disruptor started");
    }

    public void stop() {
        log.info("Disruptor stop");
        disruptor.shutdown();
    }

    public static class DisruptorThreadFactory implements ThreadFactory {
        private static final AtomicInteger poolNum = new AtomicInteger(1);
        private final ThreadGroup group;
        private final AtomicInteger threadNum = new AtomicInteger(1);
        private final String namePrefix;  // 为每个创建的线程添加前缀

        DisruptorThreadFactory(){
            SecurityManager sm = System.getSecurityManager();
            // 取得线程组
            group = (sm != null) ? sm.getThreadGroup() : Thread.currentThread().getThreadGroup();
            namePrefix = "disruptor-pool-thread-" + poolNum.getAndIncrement() + "-";
        }

        @Override
        public Thread newThread(Runnable r) {
            // 创建线程，设置线程组和线程名
            Thread t = new Thread(group, r, namePrefix + threadNum.getAndIncrement(), 0);
            if (t.isDaemon()) t.setDaemon(false);
            if (t.getPriority() != Thread.NORM_PRIORITY) t.setPriority(Thread.NORM_PRIORITY);
            return t;
        }
    }
}
```

### 4.2 WorkHandler:
当工作单元在RingBuffer中可用时，为它们实现回调接口；类型参数: 事件实现存储数据，以便在交换或事件的并行协调期间共享
```java
public interface WorkHandler<T>
{
    void onEvent(T event) throws Exception;
}
```
- 使用 Disruptor 处理自定义 netty 相关事件
```java
@Slf4j
public class TurtleEventHandler implements WorkHandler<TurtleEvent> {

    private TurtleExporterRegister turtleExporterRegister;
    private NettyReqHolder nettyReqHolder;
    private TurtleExecutor turtleExecutor;

    public TurtleEventHandler(TurtleExporterRegister turtleExporterRegister, NettyReqHolder nettyReqHolder,
                              TurtleExecutor turtleExecutor) {
        this.turtleExporterRegister = turtleExporterRegister;
        this.nettyReqHolder = nettyReqHolder;
        this.turtleExecutor = turtleExecutor;
    }

    @Override
    public void onEvent(TurtleEvent event) throws Exception {
        ProtoBufMsgBO protoBufMsgBO = event.getProtoBufMsgBO();
        ITurtleExporter exporter = turtleExporterRegister.getExporter(protoBufMsgBO.getPath(), protoBufMsgBO.getVersion());
        if (exporter == null) {
            nettyReqHolder.sendAckMsg(protoBufMsgBO.getSeq(), new TurtleException("target exporter does not exist"));
            return;
        }

        TurtleExporterHolder.setPath(protoBufMsgBO.getPath());
        TurtleExporterHolder.setVersion(protoBufMsgBO.getVersion());
        TurtleExporterHolder.setMeta(protoBufMsgBO.getMeta());
        try {
            turtleExecutor.submit(() -> {
                TurtleExporterHolder.setPath(protoBufMsgBO.getPath());
                TurtleExporterHolder.setVersion(protoBufMsgBO.getVersion());
                TurtleExporterHolder.setMeta(protoBufMsgBO.getMeta());
                try {
                    if (exporter.hasReturn()) {
                        if (exporter.isAsync()) TurtleExporterHolder.setTaskFuture(new TaskFuture<>(protoBufMsgBO, nettyReqHolder, turtleExecutor));
                        Object result = this.invoke(exporter, protoBufMsgBO);
                        if (exporter.isAsync()) {
                            if (result instanceof TaskFuture) {
                                ((TaskFuture) result).setProtoBufMsgBO(protoBufMsgBO).setFutureExecutor(nettyReqHolder).executeAsync();
                            }
                        } else {
                            nettyReqHolder.sendAckMsg(protoBufMsgBO.getSeq(), result);
                        }
                    } else {
                        nettyReqHolder.removeReq(protoBufMsgBO.getSeq());
                        this.invoke(exporter, protoBufMsgBO);
                    }
                } catch (RuntimeException e) {
                    this.handleError(protoBufMsgBO, exporter, e);
                } catch (Exception e) {
                    this.handleError(protoBufMsgBO, exporter, new RuntimeException(e.getMessage(), e));
                } finally {
                    TurtleExporterHolder.clear();
                }
            });
        } finally {
            TurtleExporterHolder.clear();
        }
    }

    private Object invoke(ITurtleExporter exporter, ProtoBufMsgBO protoBufMsgBO) {
        Object[] payload = protoBufMsgBO.getPayload();
        Object[] args = new Object[exporter.getParamSize()];
        if (ArrayUtils.isNotEmpty(payload)) {
            for (int i = 0; i < payload.length; i++) {
                if (i < args.length) args[i] = payload[i];
            }
        }
        return exporter.invoke(args);
    }

    private void handleError(ProtoBufMsgBO protoBufMsgBO, ITurtleExporter exporter, RuntimeException e) {
        if (exporter.hasReturn()) {
            if (exporter.isAsync()) {
                TurtleExporterHolder.getTaskFuture().setError(e);
            } else {
                nettyReqHolder.sendAckMsg(protoBufMsgBO.getSeq(), e);
            }
        } else {
            log.error(e.getMessage(), e);
        }
    }
}
```

### 4.3 EventHandler 
当事件在RingBuffer中可用时，将实现处理事件的回调接口  
类型参数: 事件实现存储数据，以便在交换或事件的并行协调期间共享
```java
public interface EventHandler<T>
{
    void onEvent(T event, long sequence, boolean endOfBatch) throws Exception;
}
```
- 种子接口: SequenceReportingEventHandler < T >
```java
由 BatchEventProcessor 使用，用于设置回调，允许事件处理程序在事件处理程序之后使用事件时发出通知
	onEvent(Object, long, boolean)调用
通常，这将用于处理程序执行一些批处理操作，如写入IO设备;操作完成后，实现应该调用sequence .set(long)来更新序列，并允许依赖于此处理程序的其他进程继续进行

类型参数: 事件实现存储数据，以便在交换或事件的并行协调期间共享

public interface SequenceReportingEventHandler<T> extends EventHandler<T> {
    void setSequenceCallback(Sequence sequenceCallback);
}
```

### 4.4 EventTranslator
- 实现将数据表示转换(写入)为从RingBuffer声明的事件
```java
public interface EventTranslator<T>
{
    void translateTo(T event, long sequence);
}
```
当发布到RingBuffer时，提供一个EventTranslator  
在发布序列更新之前，RingBuffer将按顺序选择下一个可用的事件，并将其提供给EventTranslator(它应该更新事件)  
类型参数: 事件实现存储数据，以便在交换或事件的并行协调期间共享
![eventTranslator](/images/types/cur/eventTranslator.png)

- EventTranslatorOneArg < T > 实现将另一个数据表示转换为从RingBuffer声明的事件
```java
public class TurtleEventProducer {

    private RingBuffer<TurtleEvent> ringBuffer;
    private final TurtleEventHandler turtleEventHandler;

    public TurtleEventProducer(RingBuffer<TurtleEvent> ringBuffer, TurtleEventHandler turtleEventHandler) {
        this.ringBuffer = ringBuffer;
        this.turtleEventHandler = turtleEventHandler;
    }

    public TurtleEventProducer(TurtleEventHandler turtleEventHandler) {
        this.turtleEventHandler = turtleEventHandler;
    }

    private static final EventTranslatorOneArg<TurtleEvent, ProtoBufMsgBO> TRANSLATOR = (event, sequence, protoBufMsgBO) ->
        event.setProtoBufMsgBO(protoBufMsgBO);

    public void onData(ProtoBufMsgBO protoBufMsgBO) {
        //ringBuffer.publishEvent(TRANSLATOR, protoBufMsgBO);
        try {
            turtleEventHandler.onEvent(new TurtleEvent(protoBufMsgBO));
        } catch (Exception e) {
            log.error(e.getMessage(), e);
        }
    }
}

```










