---
layout: post
title: Java 线程 三、锁
categories: [Thread 线程--高并发]
description: Java Thread 并发处理
keywords: 线程, Thread, 高并发
topmost: false
---

#### Lock 锁

##### 1.基础特征

  1.比 synchronized **更精确的线程语意**、**更好的性能**、允许更灵活结构、可具有差别很大的属性   
  2.可支持多个相关的 **Condition** 对象  
  3.要求程序员释放 且必须在 **finally** 中释放 **--** synchronized 自动释放

```java
trylock - 以非阻塞方式去拿锁 

java.util.concurrent.locks.Lock;
java.util.concurrent.locks.ReentrantLock;

lock.lock() 
lock.unlock(); 
trylock() 
lockInterruptibly()
```

**ReentrantLock**:  可重入、互斥锁  Lock，有与使用 synchronized 方法和语句

所访问的隐式监视器锁相同的一些基本行为和语义，但功能更强大
  	
##### 2.特性  ​	

提供对对象相关隐式监视器锁的访问
锁获取和释放在一个块结构内
多个锁时必须反顺序释放 
在获取词法范围内释放锁

```java
 Lock lock = ...; 
lock.lock();
try {
    // access the resource protected by this lock

} finally {
    lock.unlock();
}
```

##### 3.使用

  1.锁定和取消锁定出现在不同**作用范围**中时  
  必须谨慎地确保保持锁定时所执行的所有代码用 try-finally 或 try-catch 加以保护，以确保在必要时释放锁。 2.如**保证排序**、**非重入用法**或**死锁检测**   
  3.建议：除了在其自身的实现中之外,决不要以这种方式使用 lock 实例  
  内存同步效应

##### 4.死锁

A和B线程互相等待对方释放锁，某线程释放锁出现异常，如死循环之类的 

```
1）尽量一个线程只获取一个锁
2）一个线程占用一个资源
3）尝试使用定时锁，保证锁最终会被释放
```

  5.案例

```java
package com.huawei.interview;

import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;
++++++++++++++++++++++···············
public class ThreadTest {
	private int j;
	private Lock lock = new ReentrantLock();
	
	public static void main(String[] args) {
		ThreadTest tt=new ThreadTest();
		for(int i=0; i<2; i++){
			new Thread(tt.new Adder()).start();
				//tt.new Adder();
			new Thread(tt.new Substractor()).start();
		}
	}
	
	private class Subtractor implements Runnable {
		public void run() {
			while (true) {
				
				/*
				 * synchronized(ThreadTest.this){
				 * System.out.println("j--="+j--)} 
				 * */
				
				lock.lock();
				try {
					System.out.println("j--" + j--);
				} finally {
					lock.unlock();
				}
			}
		}
	}
	
	private class Adder implements Runnable{
		public void run(){
		
			/*
			 * synchronized(ThreadTest.this){
			 * System.out.println("j--="+j--)} 
			 *  */
			
			lock.lock();
			try {
				System.out.println("j++="+j++);
			} finally {
				lock.unlock();
			}
		}
	}
}
```



#### Synchronized 同步锁

##### 1.synchronized **block**: 只能用对象做参数 synchronized(**obj**)

```
synchronized 锁的取+释都是隐式的，通过编译后加上不同机器指令实现 String 每赋值一次 就会创建一个新的String实例，故同步String不能给此值赋新值->无法同步
synchronized 锁住的是对象 对象不能变
建议用 static final 定义同步量
```

##### 2.synchronized **method** -- 解决脏数据

- public synchronized void method() --method 同步
  	同对象-同方法 才能同步方法 - 静态方法行

- 单例模式：线程不安全--传统设计--产生多个对象
  	锁住 -- synchronized getInstance()-方法安全
    	/ 定义单例对象 时创建内部对象 -- 简单但不如上灵活

- 锁定方法实际是锁定此类
  --非静态 同类中method1锁定未执行完 method2不执行
  --静态   中两锁定的不同方法互不影响
  --大量同类多锁非常影响性能

##### 3.注意事项：

1. synchronized 不能**继承** 此子**不属于方法定义的一部分**  
   子类方法内**调用**父类同步方法 --> 使得子类方法具有同步性质
2. s 不能修饰 **接口**
3. s 不能用在构造方法上-->用s-block解决
4. s 可自由放置位置??-不能在返回值后面
5. s 不能同步类变量 -- 不能在定义时候修饰

#####  4.method内 synchronized(**this**) -- 属于同步块,互相影响

- 内部类  synchronized(**this**)--只影响 InnerClass

- 内部类 synchronized(**OuterClass**.**this**) -- 可与外部类同块锁定，不担心同步锁释放问题。 

- 静态方法同步 ？-- 不能用 this --> 用 Class 对象同步静态方法 

```java
static method(){ synchronized(OuterClass.**class**}
  synchronized(instance.**getClass()**)
```

-  静态-class 非-this 互不影响


- 非静态中可用 class 同步静态 静态中不能用 this 同步非静态


如：public static synchronized int n = 0; 错误

```java
public class SyncThread extends Thread{
	
	private static String sync="";
	private String methodType="";
	
	private static void method(String s){
		synchronized(sync){
		sync=s;
		System.out.println(s);
		while(true);
	}
	}
	public void method1(){
		System.out.println("method1() run");
		method("method1");
	}
	public static void staticMethod1(){
		System.out.println("staticMethod1() run");
		method("staticMethod1");
	}
	public SyncThread(String methodType){
		this.methodType=methodType;
	}
	public void run(){
		if (methodType.equals("static")) {
			staticMethod1();
		}else if (methodType.equals("nonstatic")) {
			method1();
		}
	}
	
	public static void main(String[] args)throws Exception{
		SyncThread sample1 = new SyncThread("nonstatic");
		SyncThread sample2 = new SyncThread("static");
		sample1.start();
		sample2.start();
	}
	
}
// staticMethod1() run
// staticMethod1
// 你会发现
```

#### 锁与内存

Java 对象，锁和对象的内存布局，访问定位 [锁与内存]( https://www.cnblogs.com/cosmos-wong/p/12403256.html)

1. 同步 **方法**：多线程排队执行同步方法 // 一方法等另一方法结束才能用
2. 同步 **块**：保证安全 提升效率，有效的缩小同步范围 可以提高同步效率  
   块里用synchronized(**this**)  // this不能被object代替
3. 同步 **类对象**： **静态方法**被synchronized修饰后 锁的是静态方法所属类的类对象  
   java每个类都有一个类对象Class的实例  
   jvm加载当前类时候会创建一个class实例  
   描述当前类 每个类都有且只有一个类对象，所以静态方法一定具有同步效果
4. **互斥锁**  读锁 互斥的、写锁 非互斥的   
   sychornized锁的对象相同，锁的代码区不同，两锁互相排斥  
   A() B()两方法都各自锁住 ,如果调用两方法的线程的锁对象相同，则会产生互斥锁

```java
public class SyncDemo02 {
	public static Shop s = new Shop();
	//用静态的对象类型属性为毛下面锁this锁不住
	public static void main(String[] args) {
//	   final Shop s = new Shop();
	     
		Thread t1 = new Thread(){
	         public void run(){
		       s.buy();
	         }
	      };
	      Thread t2 = new Thread(){
		    public void run(){
			 s.buy();
		    }
	      };
		
	      t1.start();
	      t2.start();
	}
}

class Shop{
	public void buy(){
		try {
			//获得当前线程对象
			Thread t = Thread.currentThread();
			System.out.println(t.getName() + ":正在挑选衣服");
			Thread.sleep(2000);
			
			//要使得线程看到的是同一个资源才行
			synchronized(this){//this指的到底是什么？--代码块
				System.out.println(t.getName()+":正在试衣间");
				Thread.sleep(2000);
			}
			System.out.println(t.getName()+":结账离开");
			
		} catch (Exception e) {
			// TODO: handle exception
		}
	}
}

public synchronized static void dosome(){
	//静态方法属于类对象 所以和类的具体对象没有关系
	Thread t = Thread.currentThread();
	System.out.println(t.getName()+":正在调用dosome()");
	try {
		Thread.sleep(2000);
	} catch (Exception e) {
		// TODO: handle exception
	}
	System.out.println(t.getName()+"：调用完毕");
}

class Table{
	private int beans = 20;
	//同步方法
	public synchronized int getBean(){
		if (beans==0) {
			throw new RuntimeException("没有豆子了");
		}
		//模拟线程切换时机
		Thread.yield();
		return beans--;
	}
}
```



#### Lock 升级:

##### 1.锁类型

对象头 synchronize 会修改被锁对象 Mark Word (对象头) 中一些属性

![locktype](/images/posts/2016-07-01-thread-lock/locktype.png)

![locktype2](/images/posts/2016-07-01-thread-lock/locktype2.png)

##### 2.锁状态

高 ==>  低：**无锁-**--->**偏向锁**---->**轻量锁**---->**重量锁**

**锁自旋**：预计持锁线程很快释放锁，则当前线程自旋，等待获取锁，如果某个锁自旋很少成功获得，那么下一次就会减少自旋。

**轻量锁**：大多Threads不存在竞争，CAS比互斥开销少，竞争激烈 则是 CAS+互斥 双开销。 

**偏向锁**：lock不存在多Thread竞争,应由一线程多次获得lock，偏向第一个访问锁的线程。   
提高带有同步，却没有竞争的程序性能，竞争激烈情况下，作用不大。  
-XX:-**UseBiasedLocking**  来关闭偏向锁，并默认进入轻量

锁可以随竞争**升级**，但不能**降级**。



##### 3.轻量锁

- 1）同步块 无 Thread 进入 -- 无 

- 2）Thread进入，在**栈帧**中创建一个**锁记录**(**Lock Record**)区域，同时将被锁对象的对象头中 **Mark Word** 拷贝到锁记录中，再尝试使用 CAS 将 Mark Word 更新为指向锁记录的指针   
  **success** ==> Thread 获得轻量锁  
  **failure** ==> JVM 检查 Lock Record 的 Mark Word 是否指向当前 Thread 的锁记录
  - yes -- Current Thread 有被锁对象的锁，直接进入
  - no -- Others Thread抢占了锁，存在竞争 ==> 轻量锁膨胀为重量锁 

- 解锁：**CAS** 实现，会尝试锁记录替换回被锁对象的 Mark Word  
  success ==> 同步操作完成  
  failure ==> 说明有 Others Thread尝试获取锁，唤醒被挂起线程(此时已经是重量锁了) 

T1, T2 两个线程调用 sychronize 代码( 出现偏向锁 升级为轻量锁)，假设T1 获取到执行权, T2 线程与 T1 线程出现竞争. 将轻量锁属性设置1.。

T2 线程没有获取到执行权, 就开始自旋等待(自旋一定次数, jvm 参数可配置)，自旋的开销, 比映射到内核切换线程的开销要低。



##### 4.偏向锁

只有一个线程访问同步锁，这时候不存在竞争，这时候加偏向锁，获取锁消耗。  
只有T1 线程调用sychronize 代码 (出现**偏向锁**):  

- step1  首先通过 被锁定对象 的对象头属性, 判断是否是偏向锁.

- case1  如果是偏向锁 : 比较 ThreadId 是否相同,相同则执行.

- case2  如果不是偏向锁 : , 通过 cas 修改 ThreadId, 将线程的 threadId 设置进去.

这个时候只有偏向锁. 每次T1 线程调用的时候, 只需要判断是否偏向锁, 对比ThreadId 就可以了.



##### 5.重量锁

T1, T2 两个线程调用 sychronize 代码(出现 **轻量锁** 升级为**重量锁**)  
第二种情况, 提到了自旋一定次数. 那么如果超过指定次数. 轻量锁就会升级成为重量锁。  
重量锁就是利用Monitor 指针. 通过系统来分配切换代码的执行权。



#### Lock 升级漫画

[BLog](https://blog.csdn.net/xvshu/article/details/88039489) 这位大兄弟的图画的挺有趣
![1](/images/posts/2016-07-01-thread-lock/1.png)
![2](/images/posts/2016-07-01-thread-lock/2.png)
![3](/images/posts/2016-07-01-thread-lock/3.png)
![4](/images/posts/2016-07-01-thread-lock/4.png)
![5](/images/posts/2016-07-01-thread-lock/5.png)
![6](/images/posts/2016-07-01-thread-lock/6.png)
![7](/images/posts/2016-07-01-thread-lock/7.png)
![8](/images/posts/2016-07-01-thread-lock/8.png)
![9](/images/posts/2016-07-01-thread-lock/9.png)

#### ReentrantLock 公平锁/非公平锁

##### 1.ReentrantLock 可重入互斥锁

重入锁：每次获取锁，同步状态+1，释放锁同步-1，等于0时最终释放锁  
基于 **AQS**(AbstractQueuedSynchronizer)

[Blog1](https://www.jianshu.com/p/4358b1466ec9)
[Blog2](https://blog.csdn.net/fuyuwei2015/article/details/83719444)
[Blog3](https://www.zhihu.com/question/57794716/answer/606126905)

默认非公平锁 ==> 吞吐和效率比公平锁高  
公平锁 Fair 要关心 Queue 情况，按队列 FIFO 获取锁，造成大量上下文切换，效率低些

```java
public ReentrantLock() {
	sync = new NonfairSync();
}

//公平锁
public ReentrantLock(boolean fair) {
     sync = fair ? new FairSync() : new NonfairSync();
}

// =================================
// 获取锁
private ReentrantLock lock = new ReentrantLock(); // 默认NonFair
public void run() {
    lock.lock();
    try {
        //do bussiness
    } catch (InterruptedException e) {
        e.printStackTrace();
    } finally {
        lock.unlock();
    }
}
```



##### 2.Fair 公平锁

- Fair 实现 

```java
final void lock() {
    acquire(1);
}

//AbstractQueuedSynchronizer 中的 acquire()
public final void acquire(int arg) {
    if (!tryAcquire(arg) && 
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

- **tryAcquire**(arg) ==> 尝试获取锁 由其子类实现

```java
protected final boolean tryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) { // =0 当前Thread可尝试获取锁
        // 判断AOS中是否还有其它线程，如果有则不会尝试获取锁
        // (这是公平锁特有的情况)  
        if (!hasQueuedPredecessors() &&
            // 没有则用CAS将AQS state改为1，获取锁
            compareAndSetState(0, acquires)) {

            setExclusiveOwnerThread(current);// 独占线程
            return true;
        }
    }
    // 支持重入
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0)
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```

- **addWaiter**(Node.EXCLUSIVE) ==> 当前Thread 写入队列

```java
addWaiter(Node.EXCLUSIVE) 线程包装为Node
    private Node addWaiter(Node mode) {
        Node node = new Node(Thread.currentThread(), mode);

        // Try the fast path of enq; backup to full enq on failure
        Node pred = tail;
        if (pred != null) {
            node.prev = pred;
            if (compareAndSetTail(pred, node)) {
                pred.next = node;
                return node;
            }
        }
        enq(node);
        return node;
    }
// 先判断队列是否为空，不为空则将封装的Node利用CAS写入队尾
// 并发失败则需调用 enq(node) 来写入

    private Node enq(final Node node) {
        for (;;) {
            Node t = tail;
            if (t == null) { // Must initialize
                if (compareAndSetHead(new Node()))
                    tail = head;
            } else {
                node.prev = t;
                if (compareAndSetTail(t, node)) {
                    t.next = node;
                    return t;
                }
            }
        }
    }
// 如上相当于 自旋加上 CAS 保证一定能写入队列
```

- **acquireQueued**(addWaiter(Node.EXCLUSIVE), arg)) 挂起Thread等待

```java
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            // 获取上个节点是否为头节点，如果时则尝试获取一次锁 
            final Node p = node.predecessor();
            if (p == head && tryAcquire(arg)) { // 成功万事大吉
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            // 不是头节点 失败，根据上节点p的waitStatus来处理
            // waitStatus 用于记录当前节点的状态，如节点取消、节点等待等
            if (shouldParkAfterFailedAcquire(p, node) && 
                // 返回Thread是否需要挂起
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
// 利用 LockSupport 的 part 方法来挂起当前线程的，直到被唤醒
private final boolean parkAndCheckInterrupt() {
    LockSupport.park(this);
    return Thread.interrupted();
}
```

##### 3.UnFair 非公平锁

Fair：        后来的人要到队尾排队买票  
NonFair：抢占式，直接尝试获取锁

- NonFair 实现


```java
final void lock() {
    //直接尝试获取锁
    if (compareAndSetState(0, 1))
        setExclusiveOwnerThread(Thread.currentThread());
    else
        acquire(1);
}
```

- **tryAcquire**(arg) 尝试获取锁，NonFair不用判断Queue中是否还有其它锁

```java
final boolean nonfairTryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        //没有 !hasQueuedPredecessors() 判断
        if (compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```

- 释放锁 Fair 和 NonFair 流程一样

```java
public void unlock() {
    sync.release(1);
}

public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            //唤醒被挂起的线程
            unparkSuccessor(h);
        return true;
    }
    return false;
}

//尝试释放锁
protected final boolean tryRelease(int releases) {
    int c = getState() - releases;
    if (Thread.currentThread() != getExclusiveOwnerThread())
        throw new IllegalMonitorStateException();
    boolean free = false;
    if (c == 0) {
        free = true;
        setExclusiveOwnerThread(null);
    }
    setState(c);
    return free;
}
```



#### ReentrantReadWriteLock 读写锁

[**读写锁**]Read-Write Lock 本质是一种**多线程设计模式**

![rwl1](/images/posts/2016-07-01-thread-lock/rwl1.png)

![rw2](/images/posts/2016-07-01-thread-lock/rw2.png)

![rw3](/images/posts/2016-07-01-thread-lock/rw3.png)



- 参与的角色：

**Reader**(读者)：对 SharedResource 角色执行 Read 操作  
**Writer**(写者)：对 SharedResource 角色执行 Write 操作

**SharedResource**(共享资源)：表示对 Reader 和 Writer 两者共享的资源  
**ReadWriteLock**(读写锁)，提供了 SharedResource 角色实现 Read 操作和 Write 操作时所需的锁

针对 Read 操作提供 readLock 和 readUnlock，对 Write 操作提供 writeLock 和 writeUnlock

```
A-Read B-Wrting  read-write conflict(读写冲突)
A-Read B-Reading  A无需等待

A-Write B-Writing  write-write conflict(写写冲突)
A-Write B-Reading  read-write conflict(读写冲突)
```

![rw4](/images/posts/2016-07-01-thread-lock/rw4.jpg)  

- 使用

```java
class CachedData {
   Object data;
   volatile boolean cacheValid;
   ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();

   void processCachedData() {
     rwl.readLock().lock();//@1
     if (!cacheValid) {
        // 必须在写锁锁上前释放所有的读锁
        rwl.readLock().unlock();//@4
        rwl.writeLock().lock();//@2
        
	// 重新确认状态，防止其他线程介入先锁了线程并修改了状态
        if (!cacheValid) {//@3
          data = ...
          cacheValid = true;
        }
        // 读锁可以在写锁内被锁上
        rwl.readLock().lock();
        rwl.writeLock().unlock(); // 手动解锁写锁，读锁仍保持
     }

     use(data);
     rwl.readLock().unlock();
   }
 }
```

读锁是排写锁操作，但不排其他读锁的操作，多个读线程可并发  
写锁是可以获得读锁的，但写锁锁下前要释放所有读锁  
写锁必须在所有的读锁释放后才能锁











## 参考：

