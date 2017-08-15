---
title: WebView加载页面
date: 2016-12-08 15:09
categories:
- Android
tags:
- WebView
---
```java
//加载网络页面
webView.loadUrl("https://lvyou.baidu.com");
//加载assets下的html目录下的html页面
webView.loadUrl("file:///android_asset/html/test.html")
```
用此方法加载页面的话，在webView加载的页面中点击链接时会在手机默认浏览器中打开，解决这个问题的话需要设置setWebViewClient方法
```
webView.setWebViewClient(new WebViewClient() {
    @Override
    public boolean shouldOverrideUrlLoading(WebView view, String url) {
        view.loadUrl(url);
        return true;
    }
});
```

<!-- more -->
如果webView只是访问本地页面的话并不需要对网络权限进行请求，因为没有对网络进行访问。

## WebView基本设置

webView的属性设置是通过`webView.getSttings()`来进行的。

```java
WebSettings settings = webHome.getSettings();
//表示支持js
settings.setJavaScriptEnabled(true);

// 设置可以支持缩放
settings.setSupportZoom(true);

// 设置出现缩放工具
settings.setBuiltInZoomControls(true);

//设置可在大视野范围内上下左右拖动，并且可以任意比例缩放
settings.setUseWideViewPort(true);

//设置默认加载的可视范围是大视野范围
settings.setLoadWithOverviewMode(true);

//设置默认的字体大小，默认为16，有效值区间在1-72之间。
mWebView.getSettings().setDefaultFontSize(18);

//自适应屏幕
settings.setLayoutAlgorithm(WebSettings.LayoutAlgorithm.SINGLE_COLUMN);

```

用WebView组件显示普通网页时一般会出现横向滚动条，这样会导致页面查看起来非常不方便。==LayoutAlgorithm==是一个枚举，用来控制html的布局，总共有三种类型：

**NORMAL**：正常显示，没有渲染变化。

**SINGLE_COLUMN**：把所有内容放到WebView组件等宽的一列中。

**NARROW_COLUMNS**：可能的话，使所有列的宽度不超过屏幕宽度。

## js调用Android本地方法

```java
payWebView.addJavascriptInterface(new Object() {

    @JavascriptInterface
    public void close() {
        AlipayActivity.this.finish();
    }


}, "TS");
```




