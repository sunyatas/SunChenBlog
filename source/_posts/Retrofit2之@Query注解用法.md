---
title: Retrofit2之@Query注解用法
date: 2016-11-22 23:50
categories:
- Android
- Retrofit

---

## 单个Query参数

现在，假设请求地址为
{% note success %} https://api.domain.com/tasks?id=123 {% endnote %}

请求方式同样为GET请求。
<!-- more -->
这与之前的豆瓣图书信息api请求的格式是有区别的，之前的地址https://api.douban.com/v2/book/1003078 其实可以看成是以下格式

{% note success %} https://api.domain.com/id {% endnote %}

那么对于
{% note success %} https://api.domain.com/tasks?id=123 {% endnote %}

这种格式的请求地址该如何使用retrofit发起请求呢？

可以看到，示例返回的响应是只有 id=123的tasks.

这个时候可以使用**@Query**注解

baseUrl = https://api.domain.com;

```java
public interface Api{  
    @GET("/tasks")
    Call<Task> getTask(@Query("id") long taskId);
}
```

调用getTask() 方法需传入参数taskId. 此参数将会被 Retrofit 映射到给定的@Query()参数名。本例中 参数名为id, 最终的请求url看起来像这样
```
/tasks?id=taskId
```
与baseUrl进行拼接即可得到完整请求地址。

## 多个Query参数

假设请求地址的格式为
```
 https://api.domain.com/tasks?id=123&id=124&id=125
```
在这种情况下需要传递多个同名的query参数. 类似上面从API请求task的例子，同样可以扩展query参数以接受多个任务的id的列表.

预期的服务器响应应该是query参数给定的**ids=[123, 124, 125]**的task集合。

而Retrofit2执行多个同名query参数请求的方法是通过提供id的List集合作为参数。Retrofit2会自动连接给定的List集合中名称相同的多个query参数.

示例如下
```java
public interface Api{  
    @GET("/tasks")
    Call<List<Task>> getTask(@Query("id") List<Long> taskIdList);
}
```
当调用getTask方法时()，需传入参数taskIdList，将**123**，**124**，**125**写入集合后再传递给getTask方法，即可将url拼接成
```java
/tasks?id=123&id=124&id=125
```

## @Query与其他注解的混合使用

以下请求的示例api来自于[豆瓣Api V2](https://developers.douban.com/wiki/?title=api_v2)。

同样是获取图书资料的Api，根据豆瓣的Api的文档

> 对于使用 GET 方式的获取数据 API，可以通过 fields 参数指定返回数据中的信息项的字段，以减少返回数据中开发者并不关心的部分。
> fields 参数的格式目前只支持逗号分隔的字段名，没有 fields 参数或 fields 参数为 all 表示不做过滤。

可以知道请求的Api可以变成如下格式
```
 https://api.douban.com/v2/book/''?fields=''
```
可以看到需要传递的参数有两个，一个是需要查询的书本的id，一个是我们指定的需要服务器返回的数据的字段名。

结合之前在[Retrofit上手](http://sunchen.com.cn/use_retrofit/)中用到的@Path注解，可以写出。
```java
public interface Api{  
    @GET("v2/book/{id}")
    Call<ResponseBody> getBookWithFields(@Path("id") String idStr, @Query("fields") String string);
}
```
这样，当调用getBookWithFields()方法时，则可以将需要查询的书本id以及我们想要的信息字段以String类型传入其中。

这里我最终传入的依旧是《小王子》的书本id**1003078**，指定服务器返回的内容为**title,author**，分别是书名以及作者

最后拼接的url为
> https://api.douban.com/v2/book/1003078?fields=title,author 

成功返回数据
```json
{"title":"小王子","author":["（法）圣埃克苏佩里"]}
```

