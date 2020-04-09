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

对于首个拿锁者，它的时段instance肯定为null，那么进入new Singleton()创建对象，

而在首个拿锁者的创建对象期间，可能有其他线程调用getInstance()，那么它们也会通过第一个if，试图拿锁，然后阻塞。

这样的话，但第一个拿锁者完成了对象创建，之后的线程都不会通过第一个if了，而这期间阻塞的线程开始唤醒，它们则需要第二个if语句来避免再次创建对象。

### DCL模式隐藏的问题

`sInstance = new Singleton();`这条指令并不是一个原子指令。而是而是分为3个汇编指令。

1. 给Singleton的实例分配内存
2. 调用Singleton()的构造函数，初始化成员字段
3. 将sInstace对象指向分配的内存空间。

但是由于Java编译器允许处理器乱序执行，以及JDK1.5之前JMM中Cache，寄存器到主内存学回写顺序的规定，上面的第二条和第三条的顺序是无法保障的。

也就是，执行顺序可能使1-2-3，也可能是1-3-2.如果是后者，并且在3执行完毕，2未执行之前，有一个新线程调用getInstance()，那么第一个if就会非空，直接返回未完成初始化的instance实例，如果再调用相关的属性和方法，就会出错。

这里用到的是volatile关键字的禁止指令重排序优化。但是volatile禁止指令重排序在JDK5之后才被修复。

参考：《Android源码设计模式》

[csdn](https://blog.csdn.net/Lin_coffee/article/details/79890361)