---
layout: post
title: Java 线程 四、同步锁原理 Monitor
comments: true,
categories: [Thread 线程--高并发]
description: Java Thread 并发处理
keywords: 线程, Thread, 高并发、Monitor
topmost: false
---

#### Monitor

vm通过进入、退出Monitor(对象监视器)，实现对方法、同步块同步，每一个对象都有且仅有一个 monitor。

###### 1.进入区 **Entry** Set   

表线程通过synchronized  

要求获取对象锁-> 对象未被锁-> 进入入拥有者;否则则在进入区等待，一旦对象锁被其他线程释放,立即参与竞争。 

###### 2.拥有者 The Owner 

表示某一线程成功竞争到对象锁 

###### 3.等待区 **Wait** Set 

表示线程通过对象的wait方法,释放对象的锁，并在等待区等待被唤醒 **notify()**  

"Entry Set" --线程-- "Waiting for monitor entry"  
"Wait Set"  --线程-- "in Object.**wait()**"  
称被synchronized保护起来的代码段为临界区 -- Entry Set

![monitor](/images/posts/2016-07-01-thread-monitor/monitor.png)

![monitor2](/images/posts/2016-07-01-thread-monitor/monitor2.png)



###### 4.查看编译后源码

```java
    public static void main(String[] args) {
        synchronized (Synchronize.class){
            System.out.println("Synchronize");
        }
    }// javap -c Synchronize 查看编译后信息
public class com.crossoverjie.synchronize.Synchronize {
  public com.crossoverjie.synchronize.Synchronize();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String[]);
    Code:
       0: ldc           #2                  // class com/crossoverjie/synchronize/Synchronize
       2: dup
       3: astore_1
       **4: monitorenter**
       5: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
       8: ldc           #4                  // String Synchronize
      10: invokevirtual #5                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      13: aload_1
      **14: monitorexit**
      15: goto          23
      18: astore_2
      19: aload_1
      20: monitorexit
      21: aload_2
      22: athrow
      23: return
    Exception table:
       from    to  target type
           5    15    18   any
          18    21    18   any
}

```










## 参考：

