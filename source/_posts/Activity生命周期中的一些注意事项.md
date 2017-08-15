---
title: Activity生命周期中的一些注意事项
date: 2016-11-28 23:05
categories:
- Android
tags:
- Activity
- 生命周期
---
对Activity的生命周期进行一些归纳。
<!-- more -->

## 普通状态下的Activity生命周期
关于Activity的生命周期，曾经在网上看到这么一张图解，将Fragment和Activity的生命周期进行了同步比对，描述的细致入微，当时就存了下来。
![](http://image.sunchen.cc/complete_android_fragment_lifecycle.png)

由于图中内容非常直观清晰，所以就不写那么多老生常谈的东西了，只写一些值得注意的地方。
* 通常情况下执行`onPause之`后就会立即调用`onStop`。但特殊情况下，如果在执行`onPause`时快速的回到当前Activity，那么`onResume`会被直接调用，这一点在上图也有体现。但这种情况比较极端，用户操作很难实现该场景。
* 旧Activity的`onPause`执行完后新Activity的`onResume`才会执行。
* 新Activity若采用了透明主题，那么旧Activity不会回调`onStop`。
* 我们可以把`onStart`与`onStop`、`onResume`和`onPause`分别看成一对对应关系。`onStart`和`onStop`是从Activity是否可见的角度来进行回调，`onResume`和`onPause`是从Activity是否位于前台进行回调的，除了这个区别，在实际使用中没有其他明显区别。
* 不能在`onPause`中做重复级的操作（不能耗时过多影响下个页面加载）。

## 特殊情况下的Activity生命周期
相对于普通情况下Activity的生命周期，特殊情况下Activity的生命周期变化我们接触的较少，通常也是不求甚解。其实所谓特殊情况无非也就是两种情况下导致的Activity被异常杀死与重建，而这其中的变化就是它们对各自Activity状态保存与恢复的过程。

### 系统配置发生改变导致Activity被杀死重建
例如屏幕旋转过程中系统配置就会发生改变，此时默认情况下Activity就会被销毁并重新创建。

这时就会调用`onSaveInstanceState`方法进行状态数据的保存。
值得注意的是`onSaveInstanceState`方法只会出现在Activity被==异常终止的情况下==（即Activity即将被销毁并有机会进行重新显示的情况下）才会调用，正常情况下不会对其进行回调（正常销毁也不会触发）。且`onSaveInstanceState`调用时间是在`onStop`之前，和`onPause`之间没有既定的时序关系，在`onPause`之前或之后都可能调用。
当Activity被重新创建后，系统会调用`onRestoreInstanceState`，并把Activity销毁时`onSaveInstanceState`方法所保存的Bundle对象作为参数传递给`onRestoreInstanceState`和`onCreate`方法。`onRestoreInstanceState`的调用时序在`onStart`之后。
在`onSaveInstanceState`和`onRestoreInstanceState`方法中，系统会默认帮我们保存当前Activity的视图结构，并且在Activity重启后为我们恢复这些数据，比如文本框中用户输入的数据、ListView滚动位置等。
在manifests文件中给Activity指定`android:configChanges`，可以指定不让Activity在相应状态下进行重建。例如指定不在屏幕旋转时进行重新创建为的属性为`orientation`，当然最好再加上一个`screenSize`。
指定后自然就也不会回调`onSaveInstanceState`和`onRestoreInstanceState`方法了，取而代之的是会调用`onConfigurationChanged`方法，我们可以在这里面做一些自己的处理。

### 资源内存不足导致低优先级的Activity被杀死
**优先级从高到低如下排序：**
* 前台正在和用户交互的Activity
* 可见但非前台的Activity
* 后台Activity（已经被暂停的Activity，比如执行了`onStop`）
当内存不足时，就会按优先级由低到高去杀死目标Activity所在的进程，并在后续通过`onSaveInstanceState`和`onRestoreInstanceState`来存储和恢复数据。






