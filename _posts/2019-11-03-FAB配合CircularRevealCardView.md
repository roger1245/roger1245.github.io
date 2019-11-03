---
layout:	 post
title:	CircularRevealCardView
subtitle: ""
date: 2019-11-3
author: "roger"
head-img: "img/post-bg-js-module.jpg"
tags:
    - Android
---

# CircularRevealCardView配合FAB使用

这是效果

![](https://raw.githubusercontent.com/roger1245/ImgBed/master/img/rgview_circular.gif)



这其实就是一个curcular reveal effect

什么是curcular reveal effect ，

>  它是Material Design当中的一个概念
>
>  当您显示或隐藏一组UI元素时，curcular reveal 将为用户提供视觉连续性。

那么如何实现上面的效果呢？

首先，我们要导入material design官方库

```k
implementation 'com.google.android.material:material:1.2.0-alpha01'
```

然后我们要更改我们的主题，让它继承自material components theme。

material components theme其实是一系列主题，demo中继承的是

```k
style name="AppTheme" parent="Theme.MaterialComponents.DayNight.NoActionBar"
```



在我们的xml布局中，添加CircularRevealCardView和FloatingActionButton

CircularRevealCardView继承自MaterialCardView，实现了CirculareRevealWidget接口

同时将根布局改为CoordinatorLayout

然后我们需要为CircularRevealWidget添加behavior

```k
app:layout_behavior="@string/fab_transformation_sheet_behavior"
```

我们注意到在FAB点击后，页面除了右下角外是灰色的，这其实是通过一个View，我们称之为遮罩

```k
<View
        android:id="@+id/scrim"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="@color/dark_scrim"
        android:visibility="invisible"
        app:layout_behavior="@string/fab_transformation_scrim_behavior" />
```

注意到我们在它内部也定义了一个behavior

总结一下：

#### FabTransformationSheetBehavior 

当FloatingActionButton是展开的时候，拥有该behavior的sheet将会显示。

一个sheet通常宽度和高度比屏幕小，同时有elevatioin，也许还会有一个遮罩(scrim)在下面

#### FabTransformationScrimBehavior

当FloatingActionButton是展开的时候，拥有该behavior的scrim将会显示

这两个behavior的不同在于sheet是有一个水波纹效果，scrim是瞬间展开的

至此，我们就实现了我们想要的效果，具体代码可以参考[github](https://github.com/roger1245/RgView)



参考：

[google animation samples](https://github.com/android/animation-samples)