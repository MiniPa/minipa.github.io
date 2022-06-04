---
layout: post
title: Java 线程 七、Thread volatile
comments: true,
categories: [Thread 线程, 高并发]
description: Java Thread 并发处理
keywords: 线程, Thread, 高并发, volatile
topmost: false
---



#### 1.Java线程缓存机制：

允许线程保存共享成员的私有copy,线程进入/离开，同步代码块才与共享成员变量的原始值对比    
-->提高速度  
-->故多线程要注意让线程及时得到共享成员变化量



#### 2.采用共享内存的方式进行线程通信的

 // 采用以下方式用主线程关闭 A 线程

```java
public class Volatile implements Runnable{

    private static volatile boolean flag = true ;

    @Override
    public void run() {
        while (flag){
            System.out.println(Thread.currentThread().getName() + "正在运行。。。");
        }
        System.out.println(Thread.currentThread().getName() +"执行完毕");
    }

    public static void main(String[] args) 
                           throws InterruptedException {
        Volatile aVolatile = new Volatile();
        new Thread(aVolatile,"thread A").start();
        System.out.println("main 线程正在运行") ;
        TimeUnit.MILLISECONDS.sleep(100) ;
        aVolatile.stopThread();
    }
    private void stopThread(){
        flag = false ;
    }
}
```



#### 3.顺序性 -- **双重检查锁** **单例模式** volatile实现

- 此处 volatile 防止指令重排

singleton = new Singleton(); // 其实是三步

```
分配内存空间(1)
初始化对象(2)
将singleton 对象指向分配的内存地址(3)
```

 volatile 是为了让以上三步顺序执行，反之3可能在2之前操作，导致某线程对象没有实例化，使用报错。

```java
public class Singleton {
    private static volatile Singleton singleton;
    private Singleton() {
    }
    public static Singleton getInstance() {
        if (singleton == null) {
            synchronized (Singleton.class) {
                if (singleton == null) {
                    singleton = new Singleton();
                }
            }
        }
        return singleton;
    }
}
```

#### 4.可见性 -- 控制停止线程的标记

如果没有用 volatile 来修饰 flag，可能其中一个线程调用了 stop()方法修改了 flag 的值  
并不会立即刷新到主内存中，导致这个循环并不会立即停止

```java
private volatile boolean flag ; // 可见性
private void run(){
    new Thread(new Runnable() {
        @Override
        public void run() {
            while (flag) {
                doSomeThing();
            }
        }
    });
}

private void stop(){
    flag = false ;
}
```

#### 5.非原子性 -- 

```java
// 非原子性 n -- 输出结果不是1000 若原子应该是1000
public class JoinThread extends Thread{

	public static volatile int n =0;
	public static synchronized void inc(){
		n++;
	}
	
	public void run(){
		for (int i = 0; i < 10; i++) {
			n+=1;-->改成inc();//则具有原子性
			try {
				sleep(10);
			} catch (Exception e) {
				e.printStackTrace();
			}
		}
	}

	public static void main(String[] args) throws InterruptedException {
		
		Thread[] threads = new Thread[100];
		for (int i = 0; i < threads.length; i++) {
			threads[i] = new JoinThread();
		}
		for (int i = 0; i < threads.length; i++) {
			threads[i].start();
		}
		for (int i = 0; i < threads.length; i++) {
			threads[i].join();
		}
		
		System.out.println("n="+JoinThread.n);
	}
	
}
```








## 参考：

[Blog]([https://crossoverjie.top/JCSprout/#/thread/Threadcore?id=%e5%8f%8c%e9%87%8d%e6%a3%80%e6%9f%a5%e9%94%81%e7%9a%84%e5%8d%95%e4%be%8b%e6%a8%a1%e5%bc%8f](https://crossoverjie.top/JCSprout/#/thread/Threadcore?id=双重检查锁的单例模式))
