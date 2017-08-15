---
title: 使用RadioGroup和RadioButton完成底部Tab
date: 2016-12-12 23:20
categories:
- Android
tags:
- View
---

由于一些原因，需要撸几个演示Demo，设计图千篇一律的都是上面为内容，底部导航栏。
<!-- more -->
其实在今年之前Android并没有推出过官方的底部导航栏，一直以来google所提倡的设计理念都是顶部Tab，如下图所示：

![google提倡的顶部Tab设计](http://image.sunchen.cc/image/7/08/02af6b7aa215f7316536c71578013.png)

底部导航栏？不好意思，那是ios的设计语言。

但在国内的环境下，能遵循google设计规范的设计师实在是少之又少，大部分情况下都是对ios照搬照抄。
其实要实现和ios上的UIToolBar控件一样的底部导航栏效果，方式可以说方式是五花八门。之前也写过一篇使用TabLayout的方式去实现，但这一次由于只是一个demo，我就采用了更为简单的方式去实现，那就是用RadioButton。

话不多说先看效果图：

![](http://image.sunchen.cc/image/f/5e/4e734e3235264055df12b8602712b.gif)

思路很简单：RadioButton本身即是单选按钮，在使用RadioGroup进行管理的情况下可以通过`setOnCheckedChangeListener`方法可以很简单的对各个不同RadioButton的点选状态进行监听，接着内容页便可以做出相应的切换。

所以 需要考虑的事情就是如何使得Radiobutton变得像是一个Tab。

首先图标和文字是必须的，而RadioButton并没有像ImageView一样的`src`属性，而使用`background`属性又会导致图标变形，使用`button`属性设置图标虽然不会发生变形但是不能控制图标位置，会导致图标和文字会横向排列，而我们需要的是图标和文字水平方向居中纵向排列，这时候就要用到`drawableTop`属性了，同时设置`button`属性为`@null`，屏蔽默认的点击效果。

**单个RadioButton的设置如下：**

```xml
<RadioButton
    android:id="@+id/rb_home"
    android:layout_width="0dp"
    android:layout_height="match_parent"
    android:layout_marginBottom="8dp"
    android:layout_marginTop="8dp"
    android:layout_weight="1"
    android:button="@null"
    android:checked="true"
    android:drawableTop="@drawable/bottom_home_selector"
    android:gravity="center"
    android:text="首页"
    android:textColor="@color/bottom_tab_text_selector" />
```
**底部icon切换效果：**

```xml
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:drawable="@mipmap/home_pressed" android:state_checked="true" />
    <item android:drawable="@mipmap/home_default" android:state_checkable="false" />
</selector>
```

**底部文字切换效果：**
```xml
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:color="@color/colorBottomSelectText" 
          android:state_checked="true"/>
    <item android:color="@color/colorBottomNormalText" />
</selector>
```


<!--![](http://7xwgdu.com1.z0.glb.clouddn.com/bottom_navigation_bar.gif)-->
<p>
用这种方式打造底部导航栏的话，在xml文件中无法对图标大小进行控制，必须使用`setBounds`方法来设置图标大小：

```java
public void initBottomSize(RadioButton v) {
    Drawable drawable = v.getCompoundDrawables()[1];
    drawable.setBounds(0, 0, 40, 40);
    v.setCompoundDrawables(null, drawable, null, null);
}
```

**最后**

其实在今年的某一天，google突然推出了关于[Bottom navigation](https://material.google.com/components/bottom-navigation.html)的设计规范，同时更新了名为[bottom-navigation-bar](https://github.com/Ashok-Varma/BottomNavigation)依赖，虽然是很打脸，不过也算是一种妥协。目前知乎、掘金等App都已经开始在使用此设计规范。（[同时知乎用户的相关讨论很有趣](https://www.zhihu.com/question/41390254?rf=41396746)）。

**因此， 上述的完成底部Tab的方法已经完全不建议在任何正式项目中启用。**

之所以写出来是因为还可以有其他的用处。




