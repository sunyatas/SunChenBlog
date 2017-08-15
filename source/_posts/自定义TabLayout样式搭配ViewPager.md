---
title: 自定义TabLayout样式搭配ViewPager
date: 2016-04-26 14:36
---


刚调试项目的时候报错了：

{% note danger %}Error:Execution failed for task ':app:clean'.> Unable to delete file {% endnote %}

<!-- more -->

前一刻使用模拟器调试还好好的没有任何问题，换上真机调试立马就报错，还好我也算是身经百战自临危不乱，淡定的看了下报错信息，不方，这错我见过，熟练的打开提示无法进行delete的目录下，关闭AS手动进行删除，重启AS，Clean Project，自信的等待clean完，报错

{% note danger %}Error:Execution failed for task ':app:clean'.> Unable to delete file{% endnote %}

这...

继续手动删除...各种sync，clean，rebuild就是无法解决

嗨呀

大概是google留下的bug，因为在Studio1.+的时候就莫名的会出现这个错误，现在都2.0正式版了，还是会时常莫名出现这个错误，google你要闹那样？

好吧，其实人生有很多东西是强求不来的，既然是官方留下来的坑，想埋也是埋不上的，既然今天碰上了，即时是怨憎会也是人生的一部分，有关玄学的东西，就试着看淡一点，时间到了，它自己就会解决，自然就会消失，所以就把Studio关了，打开markdown写下了这篇文章

回忆下前几天用过的**TabLayout**，

## TabLayout常用的方法如下： 

- addTab(TabLayout.Tab tab, int position, boolean setSelected) ：增加选项卡到layout中 

- addTab(TabLayout.Tab tab, boolean setSelected) 同上

- addTab(TabLayout.Tab tab) 同上 

- getTabAt(int index) ：得到选项卡 

- getTabCount() ：得到选项卡的总个数 

- getTabGravity() ：得到tab的Gravity 

- getTabMode() ：得到tab的模式 

- getTabTextColors() ：得到tab中文本的颜色 

- newTab() ：新建个 tab 

- removeAllTabs() ：移除所有的tab 

- removeTab(TabLayout.Tab tab) ：移除指定的tab 

- removeTabAt(int position) ：移除指定位置的tab 

- setOnTabSelectedListener(TabLayout.OnTabSelectedListener onTabSelectedListener) ：为每个 tab增加选择监听器 

- setScrollPosition(int position, float positionOffset, boolean updateSelectedText) ：设置滚动位置

- setTabGravity(int gravity) ：设置Gravity 

- setTabMode(int mode) ：设置 Mode,有两种值：TabLayout.MODE_SCROLLABLE和TabLayout.MODE_FIXED分别表示当tab的内容超过屏幕宽度是否支持横向水平滑动，第一种支持滑动，第二种不支持，默认不支持水平滑动。

- setTabTextColors(ColorStateList textColor) ：设置tab中文本的颜色 

- setTabTextColors(int normalColor, int selectedColor) ：设置tab中文本的颜色 默认 选中 

- setTabsFromPagerAdapter(PagerAdapter adapter) ：设置PagerAdapter 

- setupWithViewPager(ViewPager viewPager) ：和ViewPager联动


----------

这里我们要用到的最重要的方法是**setupWithViewPager**

## 描述一下我的大体思路 ##
* 首先我们需要将我们的Fragment放在viewPager中，实现基本的滑动效果，这里注意viewPager使用的adapter是FragmentPargerAdapter

* 使用
```java
tablayout.setUpWithViewpager(viewpager)
```
将tablayout和viewpager进行绑定

* 使用
```java
tablayout.getTabAt(i).setCustomView(view)
```
进行自定义tab样式

**FragmentPargerAdapter的代码如下：**

```java
/**
 * Created by Sunny   [晨]  on 2016/4/21   11:30.
 * 高效、简洁、强大且无注释
 */

public class RootViewPagerAdapter extends FragmentPagerAdapter {

    private List<Fragment> fragmentList;
    private String tabTitle[] = new String[]{"第一个", "第二个", "第三个"};
    private int tabImg[] = new int[]{R.drawable.select_tab_call, R.drawable.select_tab_shop, R.drawable.select_tab_wode};
    private Context context;


    public View getTabView(int position){
        View v = LayoutInflater.from(context).inflate(R.layout.item_tab,null);
        TextView titleText = (TextView) v.findViewById(R.id.tv1);
        ImageView titleImg = (ImageView) v.findViewById(R.id.img1);
        titleImg.setImageResource(tabImg[position]);
        titleText.setText(tabTitle[position]);
        return  v;
    }


    public RootViewPagerAdapter(FragmentManager fm, Context context, List<Fragment> fragmentList) {
        super(fm);
        this.context = context;
        this.fragmentList = fragmentList;
    }

    @Override
    public Fragment getItem(int position) {
        return fragmentList.get(position);
    }

    @Override
    public int getCount() {
        return fragmentList.size();
    }

    @Override
    public CharSequence getPageTitle(int position) {
        return tabTitle[position];
    }
}

```

## 设置adapter和tablayout与adapter进行绑定与自定义 ##
```java
        rootViewPagerAdapter = new RootViewPagerAdapter(getSupportFragmentManager(), this, fragmentList);

        viewpager_root.setAdapter(rootViewPagerAdapter);

        tablayout_root.setupWithViewPager(viewpager_root);

        for (int i = 0; i < tablayout_root.getTabCount(); i++) {
            TabLayout.Tab tab = tablayout_root.getTabAt(i);
            if (tab != null) {
                tab.setCustomView(rootViewPagerAdapter.getTabView(i));
            }
        }
        viewpager_root.setCurrentItem(1);
        viewpager_root.setCurrentItem(0);
```


## 这里注意：
> viewpager_root.setCurrentItem(1);

> viewpager_root.setCurrentItem(0);

为什么我要写两句看似啰嗦的代码呢？
因为我原本是打算默认显示第一个Fragment的，于是单纯的只写了`viewpager_root.setCurrentItem(0);`，成功是能成功，只是我的TabLayout的tab设置的选中效果没有显示，于是我试了一下`viewpager_root.setCurrentItem(1);`，发现选中效果这个时候又触发了

自定义Tab的字体的颜色设置如下：
```xml
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:color="@color/colorPrimary" android:state_pressed="true" />
    <item android:color="@color/colorPrimary" android:state_selected="true" />
    <item android:color="@color/capture_text_cover_bg" />
</selector>
```
我觉得这个地方应该没有问题
那么有可能是一加载布局的时候就应该让tab（或者是tab里的自定义元素）获取焦点
由于时间原因，这里先立一个flag，日后有空再进行测试

