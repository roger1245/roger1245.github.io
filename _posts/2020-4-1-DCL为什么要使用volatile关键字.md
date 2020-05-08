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



既然synchronized可以保证原子性，即从语句2->4 是不可分割的，要么执行完成，要么不执行完成。

问：加了同步锁为什么还会进行线程的的切换呢？

答：线程没有切换。我觉得是因为多核CPU实际上是多线程之间是并行执行的，虽然同步锁使得代码块具有原子性（要么执行完要么没有执行）。但其他线程是可以观察到获得锁的线程的执行的中间状态的。

个人理解：sInstance就是多线程之间的共享变量。也就是临界区资源。对临界区资源的读和写都是原子的，且串行的。



## 静态内部类单例模式

```java
public class SingleTon{
  private SingleTon(){}
 
  private static class SingleTonHoler{
     private static SingleTon INSTANCE = new SingleTon();
 }
 
  public static SingleTon getInstance(){
    return SingleTonHoler.INSTANCE;
  }
}
```

静态内部类的优点是：外部类加载时并不需要立即加载内部类，内部类不被加载则不去初始化INSTANCE，故而不占内存。即当SingleTon第一次被加载时，并不需要去加载SingleTonHoler，只有当getInstance()方法第一次被调用时，才会去初始化INSTANCE,第一次调用getInstance()方法会导致虚拟机加载SingleTonHoler类，这种方法不仅能确保线程安全，也能保证单例的唯一性，同时也延迟了单例的实例化。

## 枚举单例

```java
public enum SingleTon{
  INSTANCE;
        public void method(){
        //TODO
     }
}
```

参考：《Android源码设计模式》

[csdn](https://blog.csdn.net/Lin_coffee/article/details/79890361)

[csdn](https://blog.csdn.net/mnb65482/article/details/80458571)