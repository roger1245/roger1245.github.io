---
layout:	 post
title:	ItemTouchHelper打造可拖拽的卡片布局
subtitle: ""
date: 2019-11-6
author: "roger"
head-img: "img/post-bg-js-module.jpg"
tags:
    - Kotlin
    - Android
---

# ItemTouchHelper打造可拖拽的卡片布局

这是效果

![](https://raw.githubusercontent.com/roger1245/ImgBed/master/img/rgview_11-6.gif)

## Activity.onCreate()

首先我们创建在Activity的onCreate()方法中

```k
val itemTouchHelper = ItemTouchHelper(touchHelperCallback)
itemTouchHelper.attachToRecyclerView(list)
```

这里的**touchHelperCallback**是单例对象，继承自`ItemTouchHelper.Callback()`

## 自定义ItemTouchHelper.Callback

需要实现几个方法

### 必须方法

#### getMovementFlags()

判断RecyclerView上的哪些方向操作交由ItemTouchHelper.Callback控制

此处我们直接

```k
override fun getMovementFlags(recyclerView: RecyclerView, viewHolder: RecyclerView.ViewHolder): Int {
			//dragFlags
            return makeMovementFlags(ItemTouchHelper.UP
                                    or ItemTouchHelper.DOWN
                                    or ItemTouchHelper.LEFT
                                    or ItemTouchHelper.RIGHT,
           //swipeFlags                         
            0
            )
        }
```



#### onMove()

我们重新排列viewModel的item顺序，viewModel将会通过LiveData实现UI的更新

```k
override fun onMove(recyclerView: RecyclerView, viewHolder: RecyclerView.ViewHolder, target: RecyclerView.ViewHolder): Boolean {
            viewModel.move(viewHolder.adapterPosition, target.adapterPosition)
            return true
        }
```

#### onSwiped()  

由于没有swipe的操作，不做任何事情

#### isItemViewSwipeEnabled() 

默认为true，选择返回false，

#### isLongPressDragEnabled()  

选择返回false，因为我们想要手动处理长按操作，通过`startDrag(ViewHolder)`方法



### 非必需方法

#### onSelectedChanged()  

当item被选中的时候，需要进行的操作，通常可以加深view的elevation

#### clearView() 

当item被取消选中的时候，需要进行的操作，通常将view的elevation设为正常值



## 长按拖动view

我们在RecyclerView的adapter中传入一个函数

```k
val adapter = CheeseGridAdapter(onItemLongClick = { holder ->
            itemTouchHelper.startDrag(holder)
        })
```

然后在adapter内部的onCreateViewHolder()中添加监听器

```k
itemView.setOnLongClickListener {
                onItemLongClick(this)
                true
            }
```

这样就实现了长按拖动view



## 数据更新

如何实现当view拖动的时候，其余的item的位置也会跟着变化呢？

这说明它们的数据的位置变更了。

这种情况，如果我们选择ListAdapter配合ViewModel，那么会减轻我们的工作量

首先，我们之前写的继承自ItemTouchHelper.Callback的类的onMove()方法中，我们

```k
viewModel.move(viewHolder.adapterPosition, target.adapterPosition)
```

我们看看这个move()方法做了什么

```k
    fun move(from: Int, to: Int) {
        _cheeses.value?.let { list ->
            val cheese = list.removeAt(from)
            list.add(to, cheese)
            _cheeses.value = list
        }
    }
```

这里的`_cheeses`对象是我们定义的`MutableLiveData`对象

这时候，如果我们在onCreate()中observe

```k
viewModel.cheeses.observe(this) { cheeses  ->

            adapter.submitList(cheeses)
        }
```

于是就实现了数据的自动的刷新

本篇文章其实就是解读了一下google官方的sample，如果有什么理解不对的地方，希望大家批评

最后贴上：

[我的github实现](https://github.com/roger1245/RgView)

[google官方sample](https://github.com/android/animation-samples)

