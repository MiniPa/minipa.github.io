---
layout: post
title: Java 线程 六、CAS 原理
comments: true,
categories: [Thread 线程, 高并发]
description: Java Thread 并发处理
keywords: 线程, Thread, 高并发, CAS
topmost: false
---




#### CAS Compare and Swap

- 乐观锁的一种, 实现**非阻塞原子**操作  
  应用：java.util.concurrent.atomic，针对Java部分数据类型的原子封装，在现有的数据类型基础上,提供原子性的操作,保证线程安全。

- **ABA**问题：

compare 之后, set 之前,另一个线程 set 了 value (小概率事件) T(Thread)  

1. 执行一，出现 ABA  
   T1 取 a=3, executing ==>    执行中...
   T2 取 a=3, executing ==> a=a&times;2=6==> CMS(3=3) ==> set a=6 回主内存 ==> End  
   T3 取 a=6, executing ==> a=a - 3=3 ==> CMS(6=6) ==> set a=3 回主内存 ==> End  
   T1 执行 a=10+a=13 ==> CMS(3=3) ==> set a=13 回主内存 ==> End **a=13**  
2. 执行二，未出现 ABA：  
   T1先执行完 ==> 主内存 a = 10+a = 13 ==> End  
   T2开始执行，并执行完 ==> 主内存 a = 13&times;2 = 26 ==> End   
   T3开始执行，并执行完 ==> 主内存 a = 26 -3 = 23 ==> End **a=23**   
   综上所述 ABA 导致两次执行结果不一致

```java
public final int incrementAndGet() {// 非阻塞原子性的++i
	for(;;){
		int cucrent = get();
		int next = current + 1;
		if(compareAndSet(current,next)){
			return next;
		}
	}
}
// 此处并没有使用 synchronized 来保证原子性
```

- 优化 增加版本号，当值 与版本号均相等，CAS成功    
  每次执行修改操作后，版本号加1，或用天然有序的时间戳  
  ==> 问题：高并发,会有大量失败，其它请见 java-threads-OL PCC



#### 案例

this----当前 AutomicInteger 对象
offset----value属性在内存中的位置(非value值在内存中位置)
expect----期望值
update----新值  
当内存中value值等于expect值时,将内存中value值更新为update值

```java
private volatile int value;

public final int get() {
	return value;
}
private static final Unsafe unsafe = Unsafe.getUnsafe();
private static final long valueOffset;

static {
	try{
		valueOffset = unsafe.objectFieldOffset
			(AtomicInteger.class.getDeclaredField("value"));
			// 反射出value属性,获取其在内存中的位置
	} catch (Exception e) {
		throw new Error(e);
	}
}
public final boolean CompareAndSet(int expect, int update) {
	return unsafe.compareAndSwapInt(this,valueOffset,expect,update);
}

// 比较并设置 用Unsafe类JNI方法实现,使用CAS指令保证读写改是一个原子操作
```

Unsafe 用于执行低级别 不安全的操作方法的集合

此类中方法大部分是对**内存的直接操作**,所以**不安全**

反射 并发包等 都间接地使用了Unsafe









## 参考：

