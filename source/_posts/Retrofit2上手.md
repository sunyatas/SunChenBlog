---
title: Retrofit2上手
date: 2016-11-12 02:27
categories:
- Android
- Retrofit


---
好久没有静下心来认真写博客了，这几天好好了解了一下retrofit2
<!-- more -->
## 什么是Retrofit2？
Retrofit2当然是[Retrofit](https://github.com/square/retrofit)的升级版啦(。・・)ノ，好，那么[Retrofit](https://github.com/square/retrofit)又是什么呢？

[Retrofit](https://github.com/square/retrofit)是[square公司](https://squareup.com/)发布的一个用于**Android**和**Java**的类型安全的HTTP client，使用注解来描述HTTP请求, 进行URL参数替换， 默认集成query参数支持, 除此而外, 它还提供Multipart请求和文件上传的功能。

目前Retrofit2的底层是依赖OkHttp实现的，也就是说Retrofit本质上就是对OkHttp的更进一步封装。

我很崇拜的[JakeWharton](https://github.com/JakeWharton)也参与了其中大部分的代码编写工作。

所谓JakeWharton种树，后人乘凉，再加上[Retrofit](https://github.com/square/retrofit)+[Okhttp3](https://github.com/square/okhttp)+[Rxjava](https://github.com/ReactiveX/RxJava)算是目前很火的网络框架了，实在是很有必要进行学习。

ps:不知道是不是受美国大选影响，从8号开始就明显感觉GFW的威力增强不少，[Google](https://www.google.com)基本已无法访问，甚至连[Github](https://github.com)都已经不能打开完整页面了,真是感觉（╯‵□′）╯︵┴─┴。。。



####简单使用

首先在build.gradle文件中添加依赖
```
dependencies {  
    compile 'com.squareup.retrofit2:retrofit:2.1.0'
}
```
由于Retrofit2是构建在OkHttp之上的，因此默认情况下使用OkHttp作为网络层（当然也支持自定义，不过这里不管，就用OkHttp3），所以我们不需要再另外将OkHttp添加到依赖中。


这里我用来测试的是[豆瓣api](https://developers.douban.com/wiki/?title=api_v2)中获取图书信息的的api，具体的请求地址为https://api.douban.com/v2/book/1003078

![小王子](http://image.sunchen.cc/doubanxiaowangzi.png)

可以看到我们的地址分为两部分"https://api.douban.com/v2/book" 以及最后的编号"1003078"。其中前半部分可以看做是**baseURl**，后半部分是可以根据我们需要请求的不同数据进行动态变化的。

由于**reterfit**需要将http api转化成java interface，所以首先我要先定义一个接口文件，命名为Api。

```java

public interface Api {
    //https://api.douban.com/v2/book/1003078
    @GET("v2/book/{id}")
    Call<ResponseBody> getBookDetail(@Path("id") String idStr);
}

```
  
接下来在Activity 
```java
public class MainActivity extends AppCompatActivity {

    @BindView(R.id.tv_show)
    TextView tvShow;
    @BindView(R.id.btn_to_reserve)
    Button btnToReserve;
    @BindView(R.id.activity_main)
    RelativeLayout activityMain;
    private Api api;
    public static final String API_BASE_URL = "https://api.douban.com";
 
  

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        ButterKnife.bind(this);
    }

    @OnClick(R.id.btn_to_reserve, R.id.btn_nanfeng)
    public void onClick(View view) {
        switch (view.getId()) {
            case R.id.btn_to_reserve:
                Retrofit retrofit = new Retrofit.Builder()
                .baseUrl(API_BASE_URL).build();
                api = retrofit.create(Api.class);

        Call<ResponseBody> bookDetail = api.getBookDetail("1003078");
        bookDetail.enqueue(new Callback<ResponseBody>() {
            @Override
            public void onResponse(Call<ResponseBody> call, retrofit2.Response<ResponseBody> response) {
                try {
                    String string = response.body().string();
                    Toast.makeText(MainActivity.this, string, Toast.LENGTH_SHORT).show();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }

            @Override
            public void onFailure(Call<ResponseBody> call, Throwable t) {
                   Toast.makeText(MainActivity.this, "获取数据失败",               
                   Toast.LENGTH_SHORT).show();
            }
        });
        break;


        }


    }
}
```
最后别忘了在manifests文件中添加网络权限哦！

在程序成功跑起来之后点击按钮，弹出Toast显示出数据内容，yes！最基本的get请求就已经成功了。

![](http://image.sunchen.cc/doubantest.jpg)  

但是~~~　　看看上面啊(￣∩￣)↗↗↗↗ ，我们只是获取到了纯粹的json格式数据，如果不对它进行转换的话，在实际应用中就并没有什么吊用啦，辣么在Retrofit2中应该怎么办呢？

####转换器

下面截图自[Retrofit官方介绍](https://square.github.io/retrofit/)
![](http://image.sunchen.cc/Retrofit_detail.png)

可见在默认情况下，Retrofit只允许将HTTP主体反序列化成OkHttp的ResponseBody类型，并且它只能接受@Body的RequestBody类型。 

不过我们可以添加**转换器**以支持其他类型。并且为我们提供了六个同级模块的序列化库。

我个人和大部分人比较熟悉的可能就是[Gson](https://github.com/google/gson)和[Jackson](http://wiki.fasterxml.com/JacksonHome)了，毕竟这两个的使用频率应该算是最高的了吧，不过如果是使用阿里的[fastjson](https://github.com/alibaba/fastjson/wiki)的话，可能要多走几步路自己单独创建转换器（建立一个继承Converter.Factory的类，并在构建适配器时传递实例）。

我选择的是Gson转换器，首先我需要在build.gradle 文件来导入 Retrofit 2的 GSON 转换器，添加依赖：
```
compile 'com.squareup.retrofit2:converter-gson:2.1.0'
```

接下来在Retrofit的builder前调用
```java
.addConverterFactory(GsonConverterFactory.create())
```
 来集成 **Gson**作为默认的 **Json**转换器.

接下来创建实体类BookDetails:
```java
public class BookDetails {
    private RatingBean rating;
    private String subtitle;
    private String pubdate;
    private String origin_title;
    private String image;
    private String binding;
    private String catalog;
    private String pages;
    private ImagesBean images;
    private String alt;
    private String id;
    private String publisher;
    private String isbn10;
    private String isbn13;
    private String title;
    private String url;
    private String alt_title;
    private String author_intro;
    private String summary;
    private String price;
    private List<String> author;
    private List<TagsBean> tags;
    private List<String> translator;

.....

//珍惜生命请自生成get、set方法
}

```
代码太长，get set方法就不贴了，不要漏了get和set方法。

接下来将Api文件中之前的get请求中的请求体**ResponseBody**改为**BookDetails**：
```
    @GET("v2/book/{id}")
    Call<BookDetails> getBookDetail(@Path("id") String idStr);

```

同理，Activity文件中的**ResponseBody**一样改为改为**BookDetails**：
```
                Call<BookDetails> bookDetailsCall = api.getBookDetail("1003078");
                bookDetailsCall.enqueue(new Callback<BookDetails>() {
                    @Override
                    public void onResponse(Call<BookDetails> call, Response<BookDetails> response) {
                        //获取作者信息
                        String author_intro = response.body().getAuthor_intro();
                        Toast.makeText(MainActivity.this, author_intro, Toast.LENGTH_SHORT).show();
                    }

                    @Override
                    public void onFailure(Call<BookDetails> call, Throwable t) {
                        Toast.makeText(MainActivity.this, "获取数据失败", Toast.LENGTH_SHORT).show();
                    }
                });
```
这里我选了弹出作者信息来进行测试，因为他的字最多，我喜欢看故事≡ω≡ 。好，程序运行后同样点击按钮进行测试，嗯成功的弹出了作者信息，不过由于是Toast，所以我想看完的话就要一直一直点了～～

![](http://image.sunchen.cc/image/7/18/becc53a279ae43fc7cd5c6bbeb8c2.jpg)

嗯，大功告成这下可以想取什么数据取什么了，剩下的明天继续写，洗澡睡觉去了，明天还要去还尾田电影票**↓↓↓↓↓**
![](http://image.sunchen.cc/gold.jpg)
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 
<!-- ####POST请求

1.用@POST标注请求方法，用@Body标注参数（默认将对象转换成json字符串）

2.用@FormUrlEncoded提交form表单







