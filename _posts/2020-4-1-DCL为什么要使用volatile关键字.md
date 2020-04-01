---
layout:	 post
title:	"DCL为什么要使用volatile关键字"
subtitle: ""
date: 2020-4-1
author: "roger"
head-img: "img/post-bg-js-module.jpg"
tags:
    - Android
---



# 单例模式

## 饿汉模式

```java
public class Singleton {
    private static final Singleton instance = new Singleton();
    private Singleton() {}
    public static Singleton getInstance() {
		return instance;
	}
}
```

缺点：在声明静态对象时就已经初始化

## 懒汉模式

```java
public class Singleton {
	private static Singleton instance;
	private Singleton() {}

	public static synchronized Singleton getInstance() {
		if (instance == null)
			instance = new Singleton();
		return instance;
	}
}
```

优点：在用户第一次调用getInstance时才进行初始化。

缺点：每次调用getInstance()方法都会进行同步，这会消耗不必要的资源。 

## DCL模式

```java
public class Singleton {
	private static Singleton sInstance = null;
	private Singleton () {}
	public static Singleton getInstance() {
		if (sInstance == null)						//1
			synchronized (Singleton.class) {		//2
				if (sInstance == null) {			//3
					sInstance = new Singleton();	//4
				}						
			}
		return instance;
	}
}
```

为什么要有第二次判空？

第二次判断的目的在于有可能其他线程获取过锁，已经初始化变量。第二次检查通过，才会真正的初始化变量。

通俗的理解：线程A执行到2，已经拿到锁。此时切换到线程B，进入到1和2之间，由于拿不到锁，所以被阻塞。此时A顺利执行完实例构造。线程B拿到锁，进入2。如果没有第二次判空，那么会再次对sInstance进行赋值。第二次判空保证了多线程下的安全性。

### DCL模式隐藏的问题

`sInstance = new Singleton();`这条指令并不是一个原子指令。而是而是分为3个汇编指令。

1. 给Singleton的实例分配内存
2. 调用Singleton()的构造函数，初始化成员字段
3. 将sInstace对象指向分配的内存空间。

但是由于Java编译器允许处理器乱序执行，以及JDK1.5之前JMM中Cache，寄存器到主内存学回写顺序的规定，上面的第二条和第三条的顺序是无法保障的。

也就是，执行顺序可能使1-2-3，也可能是1-3-2.如果是后者，并且在3执行完毕，2未执行之前，被切换到线程B上，这时候sInstance因为已经在线程A内执行过了第三点，sInstance已经是非空了，所以线程B直接取走sInstance，再使用时就会出现出错。

如果是JDK1.5之后的版本，只需要将sInstance加上volatile关键字就可以保证sInstance对象每次都从主内存读取。

这里用到的是volatile关键字的禁止指令重排序优化。但是volatile禁止指令重排序在JDK5之后才被修复。

参考：《Android源码设计模式》