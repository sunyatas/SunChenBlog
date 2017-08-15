---
title: RxJava关键词搜索
date: 2017-02-27 15:15
categories:
- Android
tags:
- 搜索
- RxJava
---
用RxJava进行搜索功能的实现，只在输入框中输入文字（文字变化）的时候实时进行搜索操作。

<!-- more -->

```java
RxTextView.textChangeEvents(edSearch)
        .debounce(300, TimeUnit.MILLISECONDS)
        .observeOn(AndroidSchedulers.mainThread())
        .filter(new Func1<TextViewTextChangeEvent, Boolean>() {
            @Override
            public Boolean call(TextViewTextChangeEvent textViewTextChangeEvent) {
                boolean filter = !TextUtils.isEmpty(textViewTextChangeEvent.text());
                return filter;
            }
        })
        .switchMap(new Func1<TextViewTextChangeEvent, Observable<List<String>>>() {
            @Override
            public Observable<List<String>> call(TextViewTextChangeEvent textViewTextChangeEvent) {
                //请求网络
                List<String> list = new ArrayList<String>();
                list.add("abc");
                list.add("ada");
                return Observable.just(list);
            }
        })
        .subscribeOn(Schedulers.io())
        .observeOn(AndroidSchedulers.mainThread())
        .subscribe(new Action1<List<String>>() {
            @Override
            public void call(List<String> strings) {
                //显示结果列表
                Log.d("MainActivity", strings + "");
            }
        }, new Action1<Throwable>() {
            @Override
            public void call(Throwable throwable) {
                Log.d("异常", throwable.getMessage());
                throwable.printStackTrace();
            }
        });

```

这里用到了两个**过滤操作符**，用于从Observable发射的数据中进行选择

* `debounce`：只有在空闲了一段时间后才发射数据，通俗的说，就是如果一段时间没有操作，就执行一次操作。`debounce`的时间单位通过`TimeUnit`参数指定。

* `filter`：使用你指定的一个谓词函数测试数据项，只有通过测试的数据才会被发射，即return的值为true，才会被发射。

