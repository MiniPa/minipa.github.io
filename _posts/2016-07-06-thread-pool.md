---
layout: post
title: Java 线程 八、ThreadPool 线程池
categories: [Thread 线程--高并发]
description: Java Thread 并发处理
keywords: 线程, Thread, 高并发, 线程池, ThreadPool
topmost: false
---

#### ThreadPool 线程池

#### 1.目的

- 线程是稀缺**资源**，不能频繁的创建
- **解耦**作用；线程的创建于执行完全分开，方便维护
- 应当将其放入一个池子中，可以给其他任务进行**复用**

#### 2.ExecutorService Executors 创建参数

```java
ThreadPoolExecutor(
    int corePoolSize,                     // 基本数量
    int maximumPoolSize,                  // 最大数量
    long keepAliveTime,                   // 空闲后存活时间
    TimeUnit unit,                        // 同上
    BlockingQueue<Runnable> workQueue,    // 阻塞队列
    RejectedExecutionHandler handler      // 饱和策略 拒绝策略
) 
// threadFactory 线程构造工程
```
// 都是 ThreadPoolExecutor

```java
public static ExecutorService newCachedThreadPool() {
	return new **ThreadPoolExecutor**(
        0, Integer.MAX_VALUE, 60L, TimeUnit.SECONDS,new SynchronousQueue<Runnable>());
}
```



#### 3.池策略

```
池化：
特点：管理昂贵资源: 如connection threads等, 封装创建&销毁等操作
优势：资源复用,性能提高. 优化管理和监控
Executor ExecutorService AbstractExecutor ScheduleExecutor
ForkJoinPool ThreadPoolExecutor ScheduledThreadPoolExecutor
```

- **Cached**ThreadPool()            // 无限

- **Fixed**ThreadPool()                // 固定大小

- **Single**ThreadExecutor()       // 单线程一个人干
- **Schedduled**ThreadPool()    // 可以设置延迟时间



#### 4.拒绝策略

**RejectedExecutorHandler**:   队列满 线程池大小 >= maximumPoolSize 触发驳回，会调用此接口 rejectedExecution方法,JDK驳回策略4

- **Abort**Policy 抛出运行时异常 --默认 
- - [ ] **CallerRuns**Policy 同步调用  啥意思?
- **Discard**Policy 丢弃
- **DiscardOldest**Policy 取出队首丢弃 重新提交



#### 5.监控,日志 **HookMethods**

```java
protected void beforeExecute(Thread t, Runnable r) { }
protected void afterExecute(Thread t, Runnable r) { }
protected void terminated() { }
```

- Hystrix 监控 隔离 线程池  
  SpringBoot actuator 线程池监控,暴漏线程池到SpringBoot监控可视化

- 隔离  
  情况：多业务共享ThreadPool，其中一个消耗了所有线程，导致TP不能用 

  策略：按业务隔离，下单业务一个线程池，获取数据的另一个线程池

```java
@Configuration // 配置
public class TreadPoolConfig {
    /**
     * 消费队列线程
     * @return
     */
    @Bean(value = "consumerQueueThreadPool")
    public ExecutorService buildConsumerQueueThreadPool(){
        ThreadFactory namedThreadFactory = 
            new ThreadFactoryBuilder().setNameFormat("consumer-queue-thread-%d").build();

        ExecutorService pool = new ThreadPoolExecutor(
	          5, 5, 0L, TimeUnit.MILLISECONDS,
                new ArrayBlockingQueue<Runnable>(5),
                namedThreadFactory,new 
                ThreadPoolExecutor.AbortPolicy()
	  	);
        return pool ;
    }
} // 使用 其实就是用了一个ThreadPool的Bean

    @Resource(name = "consumerQueueThreadPool")
    private ExecutorService consumerQueueThreadPool;

    @Override
    public void execute() {
        //消费队列
        for (int i = 0; i < 5; i++) {
            consumerQueueThreadPool.execute(new ConsumerQueueThread());
        }

    }
```

- 定义两个线程池，用于执行订单、处理用户

```java
/**
 * Function:订单服务
 *
 */
public class CommandOrder extends HystrixCommand<String> {

    private final static Logger LOGGER = LoggerFactory.getLogger(CommandOrder.class);

    private String orderName;

    public CommandOrder(String orderName) {
        super(Setter.withGroupKey(
                //服务分组
                HystrixCommandGroupKey.Factory.asKey("OrderGroup"))
                //线程分组
                .andThreadPoolKey(HystrixThreadPoolKey.Factory.asKey("OrderPool"))

                //线程池配置
                .andThreadPoolPropertiesDefaults(HystrixThreadPoolProperties.Setter()
                        .withCoreSize(10)
                        .withKeepAliveTimeMinutes(5)
                        .withMaxQueueSize(10)
                        .withQueueSizeRejectionThreshold(10000))

                .andCommandPropertiesDefaults(
                        HystrixCommandProperties.Setter()
                                                  .withExecutionIsolationStrategy(HystrixCommandProperties.ExecutionIsolationStrategy.THREAD))
        );
        this.orderName = orderName;
    }

    @Override
    public String run() throws Exception {
        LOGGER.info("orderName=[{}]", orderName);
        TimeUnit.MILLISECONDS.sleep(100);
        return "OrderName=" + orderName;
    }
}

/**
 * Function:用户服务
 *
 */
public class CommandUser extends HystrixCommand<String> {

    private final static Logger LOGGER = LoggerFactory.getLogger(CommandUser.class);
    private String userName;

    public CommandUser(String userName) {
        super(Setter.withGroupKey(
                //服务分组
                HystrixCommandGroupKey.Factory.asKey("UserGroup"))
                //线程分组
                .andThreadPoolKey(HystrixThreadPoolKey.Factory.asKey("UserPool"))

                //线程池配置
                .andThreadPoolPropertiesDefaults(HystrixThreadPoolProperties.Setter()
                        .withCoreSize(10)
                        .withKeepAliveTimeMinutes(5)
                        .withMaxQueueSize(10)
                        .withQueueSizeRejectionThreshold(10000))
                //线程池隔离
                .andCommandPropertiesDefaults(
                        HystrixCommandProperties.Setter()
                                .withExecutionIsolationStrategy(HystrixCommandProperties.ExecutionIsolationStrategy.THREAD))
        )
        ;
        this.userName = userName;
    }

    @Override
    public String run() throws Exception {
        LOGGER.info("userName=[{}]", userName);
        TimeUnit.MILLISECONDS.sleep(100);
        return "userName=" + userName;
    }
}
```

- 后台模拟运行

```java
public static void main(String[] args) throws Exception {
    CommandOrder commandPhone = new CommandOrder("手机");
    CommandOrder command = new CommandOrder("电视");

    //阻塞方式执行
    String execute = commandPhone.execute();
    LOGGER.info("execute=[{}]", execute);

    //异步非阻塞方式
    Future<String> queue = command.queue();
    String value = queue.get(200, TimeUnit.MILLISECONDS);
    LOGGER.info("value=[{}]", value);

    CommandUser commandUser = new CommandUser("张三");
    String name = commandUser.execute();
    LOGGER.info("name=[{}]", name);
}
```

![result](/imagess/posts/2016-07-06-thread-pool/result.png)

- 实现原理： 用一个 Map 来存放不同业务对应的线程池

![theory](/imagess/posts/2016-07-06-thread-pool/theory.png)



#### 6.状态

**shutdown**() 当线程池所有线程运行完后 线程池结束  
**shutdownNow**() //立刻结束

```
RUNNING 接受新任务 处理队列中任务
SHUTDOWN 不接受新任务 处理队列中任务
STOP 不接受 不处理
TIDYING 所有任务终止 workerCounter为0 回掉terminate()
TERMINATED 终态 terminated()执行完成
```

![threadpoolstatus2](/imagess/posts/2016-07-06-thread-pool/threadpoolstatus2.jpg)

![threadpoolstatus](/imagess/posts/2016-07-06-thread-pool/threadpoolstatus.png)



#### 7.执行流程

##### execute()

- 1.获取当前线程池的状态

- 2.当前线程数量小于 coreSize 时创建一个新的线程运行 
- 3.如果当前线程处于运行状态，并且写入阻塞队列成功

**双重检查**，再次获取线程状态

- 4.如果线程状态变了（非运行状态）就需要从阻塞队列移除任务，  
  并尝试判断线程是否全部执行完毕。同时执行拒绝策略 

- 5.如果当前线程池为空就新创建一个线程并执行 
- 6.如果在第三步的判断为非运行状态，尝试新建线程，如果失败则执行拒绝策略  

// 这段解释的垃圾  

![execute](/imagess/posts/2016-07-06-thread-pool/execute.png)

##### Flow 执行流程

![threadpoolflow](/imagess/posts/2016-07-06-thread-pool/threadpoolflow.jpg)

![advice](/imagess/posts/2016-07-06-thread-pool/advice.jpg)



#### 8.工作原理

[Blog]([https://crossoverjie.top/JCSprout/#/thread/thread-gone2?id=%e7%ba%bf%e7%a8%8b%e6%b1%a0%e7%9a%84%e5%b7%a5%e4%bd%9c%e5%8e%9f%e7%90%86](https://crossoverjie.top/JCSprout/#/thread/thread-gone2?id=线程池的工作原理))

线程池中，线程最后会被封装为 ThreadPoolExecutor.**Worker** 对象，而这个Worker 是实现了 **Runnable** 接口的，所以他自己本身就是一个线程。

案例：一个核心线程、最大线程数、阻塞队列都为2的线程池，往线程池丢一个任务

![threadpool](/imagess/posts/2016-07-06-thread-pool/threadpool.jpg)  

![theory1](/imagess/posts/2016-07-06-thread-pool/theory1.png)

![theory2](/imagess/posts/2016-07-06-thread-pool/theory2.jpg)



#### 9.使用建议

- 线程数量安排 // 如下公式只是个人见解

```
启动线程数 = [ 任务执行时间 / ( 任务执行时间 - IO等待时间 ) ] x CPU内核数
任务执行 10min, IO 8min 2核CPU 
	    threads = 10/(10-8)*2=10
控制线程池大小,尽量使用有界队列并且设置大小 避免OOM
设置合理的驳回策略 使用于各自的业务
```

- 配置线程池线程数量


```
IO  密集型：Thread运行时间短，可多配，如上计算
CPU 密集型：（大量复杂的运算）分配较少的线程，如 CPU个数相当的数量 // 还得实际去测试
```

- 关闭线程池

```java
// shutdown() 停止接受新任务，队列任务执行完毕
// shutdownNow() 停止接受新任务，中断所有的任务，状态变为 stop

long start = System.currentTimeMillis();
for (int i = 0; i <= 5; i++) {
pool.execute(new Job());
}

pool.shutdown();

while (!pool.awaitTermination(1, TimeUnit.SECONDS)) {
	LOGGER.info("线程还在执行。。。");
}
long end = System.currentTimeMillis();
LOGGER.info("一共处理了【{}】", (end - start));
```



#### 10.Fork/Join

Fork/Join JDK1.7后加入的并行计算框架  

```
并行: 系统中有多个任务同时执行
并发: 系统中有多个任务同时存在
```

**Fork**: 拆分大任务为小任务分别计算  
**Join**: 获取到子任务执行结果,并进行合并(这是个递归过程)  子任务分配到不同核上运行----效率最高

```java
// 伪代码如下
Result solve(Problem problem){
	if(problem is small) {
		solve directly
	} else {
		split problems into independent parts
		fork new subtasks to solve each part
		join all subtasks
		compose result from subresults
	}
}
```

核心类: **ForkJoinPool** 接受 **ForkJoinTask** 并计算结果

子类:     **RecursiveTask**(有返回值) **RecursiveAction**(无返回值)

![forkjoin](/imagess/posts/2016-07-06-thread-pool/forkjoin.png)

案例：计算超大数组所有元素的合

```java
package thread.parallel;

import java.util.Arrays;
import java.util.Random;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.RecursiveTask;
/*
 * ForkJoin 并行计算框架
 */
public class SumTask extends RecursiveTask<Integer> {
	
	private static final long serialVersionUID = -1054283494278538276L;
	private static final int THRESHOLD = 10000000;
//	private static final int THRESHOLD = 90000000;
	private long[] array;
	private int low;
	private int high;
	
	public SumTask(long[] array, int low, int high) {
		this.array = array;
		this.low = low;
		this.high = high;
	}

	@Override
	protected Integer compute() {
		int sum = 0;
		if (high - low < THRESHOLD) {// 小于阈值直接计算
			for (int i = 0; i < array.length; i++) {
				sum += array[i];
			}
		} else {
			int mid = (low + high) >>> 1;// 大任务分割成2个小任务
			SumTask left = new SumTask(array, low, mid);
			SumTask right = new SumTask(array, mid, high);
			left.fork();// 分别计算
			right.fork();
			System.out.println("left.join() " + left.join() + " right.join() " + right.join());
			sum = left.join() + right.join();
		}
		return sum;
	}
	
	public static void main(String[] args) throws InterruptedException, ExecutionException {
		long[] array = genArray(90000000);
//		System.out.println(Arrays.toString(array));
	SumTask sumTask = new SumTask(array, 0, array.length-1);
	
	long begin = System.currentTimeMillis();
	
	ForkJoinPool forkJoinPool = new ForkJoinPool();
	forkJoinPool.submit(sumTask);
	Integer result = sumTask.get();
	
	long end = System.currentTimeMillis();
	
	System.out.println(String.format("结果 %s 耗时 %sms", result, end-begin));
	}
	
	private static long[] genArray(int size) {
		long[] array = new long[size];
		for (int i = 0; i < size; i++) {
			array[i] = new Random().nextLong();
		}
		return array;
	}

}
```










## 参考：

