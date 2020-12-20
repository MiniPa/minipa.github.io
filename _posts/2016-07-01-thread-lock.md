---
layout: post
title: Java 线程 三、锁
categories: [Thread 线程--高并发]
description: Java Thread 并发处理
keywords: 线程, Thread, 高并发
topmost: false
---

#### lock 锁

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

##### 4.案例

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


























## 参考：

