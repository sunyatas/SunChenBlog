---
title: Retrofit2发送Post请求
date: 2016-11-27 15:21
categories:
- Android
- Retrofit
---
## POST和 GET的区别

- GET在浏览器回退时是无害的，而POST会再次提交请求。
- GET产生的URL地址可以被Bookmark，而POST不可以。
- GET请求会被浏览器主动cache，而POST不会，除非手动设置。
- GET请求只能进行url编码，而POST支持多种编码方式。
- GET请求参数会被完整保留在浏览器历史记录里，而POST中的参数不会被保留。
- GET请求在URL中传送的参数是有长度限制的，而POST么有。
- 对参数的数据类型，GET只接受ASCII字符，而POST没有限制。
- GET比POST更不安全，因为参数直接暴露在URL上，所以不能用来传递敏感信息。
- GET参数通过URL传递，POST放在Request body中。
<!-- more -->
由于使用GET方式的请求，Client会把http header和data一并发送出去，服务器直接返回数据。
而对于POST，Client 先发送header，服务器先返回信息性状态码,提示正在处理可以走通，Client再发送data，服务器再返回数据。

因此，在进行POST请求时，必须添加指定的header请求头字段。

我这里需要的请求头字段有：
> Cookie
a_curDateTime
a_cid
a_usersOs
a_organizationCode
a_singInfo

## 添加请求头

在Retrofit2中，设置HTTP header的方式几乎与在OKHttp中设置的方式一样，使用==Interceptor ==。

为了提高代码的可读性与可维护性，这里将具体的网络请求方式抽出到一个ServiceGenerator类中。

```java
public class ServiceGenerator {

    public static final String API_BASE_URL = " https://api.domain.com";
    private static OkHttpClient.Builder httpClient = new OkHttpClient.Builder();
    private static Retrofit.Builder builder = new Retrofit.Builder()
                         .baseUrl(API_BASE_URL)
                         .addConverterFactory(GsonConverterFactory.create());
    public static <S> S createService(Class<S> serviceClass) {
            httpClient.addInterceptor(new Interceptor() {
                @Override
                public Response intercept(Chain chain) throws IOException {
                    Request original = chain.request();
                    Request.Builder requestBuilder = original.newBuilder();
                    final String curDateTime = String.valueOf(System.currentTimeMillis());
                    requestBuilder.header("Cookie", "JSESSIONID=")
                            .header("a_curDateTime", curDateTime)
                            .header("a_cid", "true")
                            .header("a_usersOs", "androidUser")
                            .header("a_organizationCode", "nanfeng")
                            .header("a_singInfo", PasswordEncoder.MD5("true", "true")).method(original.method(), original.body());
                    Request request = requestBuilder.build();
                    return chain.proceed(request);
                }
            });
        OkHttpClient client = httpClient.build();
        Retrofit retrofit = builder.client(client).build();
        return retrofit.create(serviceClass);
    }
}
```

## 使用
```java
public interface Api {
    @POST("accessAll.ews")
    Call<ResponseBody> getHomeDetail(@QueryMap Map<String, String> stringStringMap);
}
```
接口方法的定义要根据实际需求来制定，这里调用getHomeDetail需要传入带有指定参数的Map。

```java
Api service = ServiceGenerator.createService(Api.class);
                Map<String, String> stringStringMap = new HashMap<>();
                stringStringMap.put("cmd", "getHomeData");
                Call<ResponseBody> responseBodyCall = service.getHomeDetail(stringStringMap);
                responseBodyCall.enqueue(new Callback<ResponseBody>() {
                    @Override
                    public void onResponse(Call<ResponseBody> call, Response<ResponseBody> response) {
                        try {
                            String string = response.body().string();
                            Toast.makeText(MainActivity.this, string, Toast.LENGTH_SHORT).show();
                        } catch (IOException e) {
                            e.printStackTrace();
                        }
                    }
                    @Override
                    public void onFailure(Call<ResponseBody> call, Throwable t) {
                    }
                });
```

可以看出，使用Retrofit进行POST和GET请求时本质上是没有区别的，只是POST请求需要根据服务端的需求进行请求头的添加吗。事实上，甚至POST和GET请求本身在原理上也并没有区别。



