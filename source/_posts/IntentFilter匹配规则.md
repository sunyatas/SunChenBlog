---
title: IntentFilter匹配规则
date: 2016-12-03 01:23
categories:
- Android
tags:
- Activity
---
隐式调用Activity需要Intent能够匹配目标组件中的IntentFilter中所设置的过滤信息，如果不匹配则无法启动目标Activity。
<!-- more -->
一个Activity中可以有多个intent-filter，一个Intent只要能匹配任何一组intent-filter即可成功启动对应的Activity。

IntentFilter中的过滤信息有action、category、data。

### action的匹配规则
action是一个字符串。

action的匹配规则是Intent中的action必须能够和过滤规则中的action匹配，即字符串值完全一样。一个过滤规则中可以有多个action，只要Intent中的action能够和过滤规则中的任意一个action相同即可匹配成功。action区分大小写，大小写不同也会导致action匹配失败。

### category的匹配规则

category也是一个字符串。

它的匹配规则和action不同，在Intent中可以没有category，没有的话这个Intent仍然可以匹配成功，但是如果Intent中有category的话，那么不论有几个，每一个都要和过滤规则中的其中一个category相同。

当然也可以不设置category，因为系统在调用startActivity或者startActivityForResult的时候会默认为Intent加上`android.intent,category,DEFAULT`这个category。同时，由于这个原因，我们必须在intent-filter中指定`android，intent.category.DEFAULT`这个category。

### data匹配规则
#### data的数据格式

data的语法如下：

```
<intent-filter>
    <data
        android:mimeType="string"
        android:scheme="string"
        android:host="string"
        android:port="80"
        android:path="/string"
        android:pathPattern="string"
        android:pathPrefix="/string" />

...
</intent-filter>
```

可以发现data由两部分组成，**mimeType**和**URI**。

mimeType指媒体类型，比如image/jpeg、audio/mpeg4-generic、video/*等，可以表示图片、文本、视频等不同的媒体格式。

URI即
```
 android:scheme="string"
 android:host="string"
 android:port="80"
 android:path="/string"
 android:pathPattern="string"
 android:pathPrefix="/string"
```
这一部分。

这其中有很多属性，将这些属性拼起来的格式为:

`scheme://<host>:<port>/[<path>|<pathPrefix>|<pathPattern>]`

例如:

> content://com.example.test:200/folder/subfolser/etc

> http://www.baidu.com:80/search/info

下面一一介绍每一个属性的含义：

**Scheme**

URI的模式，比如http、file、content等，如果URI中没有指定scheme，那么整个URI的其他参数无效，即URI无效。

**Host**

URI的主机名，比如www.baidu.com，如果host未指定，那么整个URI中的其他参数无效，同样即URI无效。

**Port**

URI中的端口号，比如80，只有当URI中指定了scheme和host参数时port参数才是有效的。

**path、pathPattern、pathPrefix**

这三个参数表示路径信息，其中path表示完整的路径信息；pathPattern也表示完整的路径信息，但它里面可以包含通配符`*`,`*`表示0个或多个任意字符，需要注意的是，由于正则表达式的规范，如果想表达真实的字符串，那么`*`要写成`\\*`，`\\`要写成`\\\\`；pathPrefix表示路径的前缀信息。

#### data匹配规则

data的匹配规则和action类似， 如果过滤规则中定义了data,那么Intent中必须也要定义可匹配的data。

如下过滤规则：
```xml
<intent-filter>
    <data
        android:mimeType="image/*" />
    ...
</intent-filter>
```
这种规则指定了媒体类型为所有类型的图片，那么Intent中的mimeType属性必须为"image/*"才能匹配，这种情况下虽然过滤规则没有指定URL，但却有默认值，URI的默认值为content和file。

也就是说，虽然没有指定URI，但是Intent中的URL部分的schema必须为content或者file才能匹配，这单是尤其需要注意的。所以为了匹配上面的规则可以写出如下示例

```java
intent.setDataAndType(Url.parse("file://abc","image/png"));
```

另外如果要为Intent指定完整的data，必须调用setDataAndType()方法，不能先调用setData再调用setType，因为这两个方法彼此会清除对方的值。

```xml
<intent-filter>
    <data android:mimeType="video/mpeg" android:scheme="http"/>
    <data android:mimeType="audio/mepg" android:scheme="http"/>
    ...
</intent-filter>
```

这种规则定义了两组data规则，且每一个data都指定了完整的属性值，既有URI又有mimeType。为了匹配上面规则，可以写出如下示例
```java
intent.setDataAndType(Url.parse("http://abc","video/mpeg"));
```
或者
```java
intent.setDataAndType(Url.parse("http://abc","audio/mpeg"));
```
值得注意的地方
```java
<intent-filter>
    <data android:scheme="file" 
          android:host="sunchen.cc"/>
...
</intent-filter>
```
```java
<intent-filter>
    <data android:scheme="file"/>
    <data android:host="sunchen.cc"/>
...
</intent-filter>
```
和action不同，上面的这两种特殊写法，它们的作用是一样的。

Intent-filter的匹配规则对于Service和BroadcastReceiver也是同样的道理，不过系统对于Service的建议是尽量使用显示调用的方式来启动服务。






