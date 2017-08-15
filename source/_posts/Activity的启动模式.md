---
title: Activity的启动模式
date: 2016-11-30 21:20
categories:
- Android
tags:
- Activity
- 启动模式
---

对Activity的启动模式做一些归纳。
<!-- more -->

### standard：
标准模式，每次启动一个Activity都会创建一个新的实例，不管这个实例是否已经存在。这是典型的多实例实现，一个任务栈中可以有多个实例。每个实例也可以属于不同的任务栈。任务栈是一种“后进先出”的栈结构。谁启动了这个Activity，那么这个Activity就运行在启动它的那个Activity所在的栈中。

### singleTop：
栈顶复用模式，在这种模式下，如果新Activity已经位于任务栈的栈顶，那么该Activity不会被重新创建，它的`onNewIntent`方法会被回调，通过此方法的参数我们可以取出当前请求的信息，但`onCreate`和`onStart`方法不会被系统调用，应为它并没有发生改变。如果新Activity的实例已经存在但不是位于栈顶，那么新Activity仍会重新重建。

### singleTask：
栈内复用模式，这是一种单例模式，在这种模式下，只要Activity在一个栈中存在，那么多次启动此Activity都不会重新创建实例，当然同样也会回调`onNewIntent`方法。由于`singleTask`默认具有`clearTop`的效果，因此在进行重复请求的Activity上面的所有Activity都会被移出至栈外（当然是和这个Activity在同一个栈内的位于其之上的Activity）。

### singleInstance：
单实例模式，这是一种加强的`singleTask`模式，除了具备`singleTask`的所有特性外还加强了一点，那就是具有此模式的Activity只能单独的位于一个任务栈中，因此在启动具有此模式的Activity时系统会为它创建一个新的任务栈

默认情况下所有Activity所需的任务栈的名字为应用的包名，可以为每个Activity都单独指定`TaskAffinity`属性，这个属性值必须不能和包名相同，否则就相当于没有指定。`TaskAffinity`属性主要和`singleTask`启动模式或者`allowTaskReparenting`属性配对使用，在其他情况下没有意义。

另外，任务栈分为前台任务栈和后台任务栈，后台任务栈中的Activity处于暂停状态，用户可以通过切换将后天任务栈再次调回到前台。

当`TaskAffiliate`和`singleTask`模式配对使用时，它是具有该模式的Activity的目前任务栈的名字，待启动的Activity会运行在名字和`TaskAffinity`相同的任务栈中。

假如现在有2个应用A和B，A启动了B的一个Activity C，然后从按HOME键回到桌面，然后再单击B的桌面图标，这个时候并不是启动了B的主Activity，而是重新显示了已经被已经被应用A启动的Activity C，，或者说，C从A的任务栈转移到了B的任务栈中。可以这么理解，由于A启动了C这个时候C只能运行在A的任务栈中，但是C属于B应用，正常情况下，它的`TaskAffinity`肯定不可能和A任务栈相同（包名不同）。所以当B被启动后，B会创建自己的任务栈，这个时候系统发现C原本想要的任务栈已经被创建了，所以就把C从A的任务栈中转移过来了。


