---
layout:	 post
title:	"仿造QQ的recyclerView效果实现"
subtitle: ""
date: 2020-4-14
author: "roger"
head-img: "img/post-bg-js-module.jpg"
tags:
    - Android
---




最近在Google官方的github库，看到了一个有意思的recyclerView效果。

像这样：

<img src="https://raw.githubusercontent.com/roger1245/ImgBed/master/img/4-14-1.1.gif" style="zoom: 67%;" />

个人感觉似乎和QQ的效果差不多，只不过QQ用的是Fling动画，而这里用的是Spring动画。有意思的是，似乎关于实现该效果所使用的EdgeEffectFactory这个类网上博客介绍不多，正好看其API比较简单，于是打算写篇博客来介绍这个动画的实现效果。

## EdgeEffectFactory的API介绍

EdgeEffectFactory是存在于Recyclerview的内部的一个静态类，即可以通过`RecyclerView.EdgeEffectFactory`得到，官方解释这个类的作用是`让你自定义RecyclerView的过度边缘滚动的效果`，看上面的gif可以知道，确实是实现了一个自定义的过度边缘滚动效果。

我们需要创建一个EdgeEffectFactory并重写其createEdgeEffect方法，在createEdgeEffect中需要返回一个EdgeEffect对象

就像这样：

![](https://raw.githubusercontent.com/roger1245/ImgBed/master/img/4-14-2.1.png)

## EdgeEffect的API介绍

上面说到了我们要给createEdgeEffect中返回一个EdgeEffect对象，那么这个EdgeEffect类是什么东东？

Android官网说这个类用作：“当用户滚动到2D空间中的内容边界之外时，此类将执行可滚动窗口小部件边缘使用的图形效果。 ”

大体可以理解为过度边缘滚动效果的具体实现

我们需要重写这个类中的`onPull(deltaDistance: Float)`，`onPull(deltaDistance: Float, displacement: Float)`，`onRelease()`和`onAbsorb`方法。这四个方法（其实是3个）的调用时机非常好理解。

### onPull介绍：

![](https://raw.githubusercontent.com/roger1245/ImgBed/master/img/4-14-3.png)

onPull调用时机就是在用户朝远离边缘的方向拉动的时候，我们需要在这个方法里面去更新recyclerView的每一个可见item的`translationY`值

### onRelease介绍：

![](https://raw.githubusercontent.com/roger1245/ImgBed/master/img/4-14-4.png)

onRelease调用时机就是在用户放开的手指的那一刻，我们需要在这里让recyclerView的每一个可见item的translationY值变成0，为了实现一个translationY之间的过渡，我们可以使用属性动画，使用SpringAnimation或者FlingAnimation。示例使用的是SpringAnimation（弹簧动画）。

### onAbsorb介绍：

![](https://raw.githubusercontent.com/roger1245/ImgBed/master/img/4-14-5.png)

onAbsorb的调用时机就是recyclerView在脱离用户手指滑动期间，到了recyclerview的边缘并且此时速度不为0。我们需要在这里使用一个SpringAnimation或着FlingAnimation来做一个类似“缓冲”的效果

## 实现代码：

```kotlin
val edgeEffectFactory = object : RecyclerView.EdgeEffectFactory() {
        override fun createEdgeEffect(view: RecyclerView, direction: Int): EdgeEffect {
            return object : EdgeEffect(view.context) {
                override fun onPull(deltaDistance: Float) {
                    super.onPull(deltaDistance)
                    handlePull(deltaDistance)
                }
				
                override fun onPull(deltaDistance: Float, displacement: Float) {
                    super.onPull(deltaDistance, displacement)
                    handlePull(deltaDistance)
                }
				//更新recyclerView的每一个可见item的`translationY`值
                private fun handlePull(deltaDistance: Float) {
                    val sign = if (direction == DIRECTION_BOTTOM) -1 else 1
                    val translationYDelta = sign * view.height *  deltaDistance * OVERSCROLL_TRANSLATION_MAGNITUDE
                    //一个内联函数，更新每一个recyclerview的可见item的translationY值
                    view.forEachVisibleHolder { holder: CheeseHolder ->
                        holder.translationY.cancel()
                        holder.itemView.translationY += translationYDelta

                    }
                }
				//在这里让recyclerView的每一个可见item的translationY值变成0，使用到了SpringAnimation
                override fun onRelease() {
                    super.onRelease()
                    view.forEachVisibleHolder { holder: CheeseHolder ->
                        holder.translationY.start()
                    }
                }
				//用SpringAnimation来做一个recyclerview的到达边缘的惯性缓冲效果
                override fun onAbsorb(velocity: Int) {
                    super.onAbsorb(velocity)
                    val sign = if (direction == DIRECTION_BOTTOM) -1 else 1
                    val translationVelocity = sign * velocity * FLING_TRANSLATION_MAGNITUDE
                    view.forEachVisibleHolder { holder: CheeseHolder ->
                        holder.translationY.setStartVelocity(translationVelocity).start()
                    }
                }
            }
        }
    }
```

其中的CheeseHolder是一个Recyclerview的ViewHolder，我们在里面存有一个SpringAnimation

```kotlin
    class CheeseHolder(view: View) : RecyclerView.ViewHolder(view) {
        val translationY: SpringAnimation = SpringAnimation(itemView, SpringAnimation.TRANSLATION_Y)
                .setSpring(
                        SpringForce()
                                .setFinalPosition(0f)
                                .setDampingRatio(SpringForce.DAMPING_RATIO_MEDIUM_BOUNCY)
                                .setStiffness(SpringForce.STIFFNESS_LOW)
                )
		...
    }
```

## 最后

只要通过

```kotlin
recyclerview.edgeEffectFactory = adapter.edgeEffectFactory
```



至此，就实现了开篇的gif效果。

这篇文章其实是参考Google官方的[Motion](https://github.com/android/animation-samples/tree/master/Motion)库的代码。

自己也写一个实现：[github](https://github.com/roger1245/RgView)

如果有什么好的建议欢迎评论指正。