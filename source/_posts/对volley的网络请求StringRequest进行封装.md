---
title: 对volley的网络请求StringRequest进行封装
date: 2016-04-20 14:14
---




之前有对Volley的stringRequest网络请求进行封装过，当时的想法是写了一个BaseActivity，在这里面创建网络请求send()方法：
<!-- more -->
```java
private RequestQueue requestQueue;

@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    requestQueue = Volley.newRequestQueue(getApplicationContext());
}

private void send(String url, String target, final Map<String, String> params, final CallBack callBack) {

    StringRequest stringRequest = new StringRequest(Request.Method.POST, url, new Response.Listener<String>() {
    
        @Override
        public void onResponse(String response) {

        }
    }, new Response.ErrorListener() {
    
        @Override
        public void onErrorResponse(VolleyError error) {

        }
    }){
    
        @Override
        public Map<String, String> getHeaders() throws AuthFailureError {

            return super.getHeaders();
        }

        @Override
        protected Map<String, String> getParams() throws AuthFailureError {
            return super.getParams();
        }

        @Override
        protected Response<String> parseNetworkResponse(NetworkResponse response) {
            return super.parseNetworkResponse(response);
        }
    };

        requestQueue.add(stringRequest);
        requestQueue.start();
    }

```

这样就完成了对volley网络框架的饿一个简单封装，然后让所有的Activity继承这个BaseActivity需要进行网络请求的时候就复写send()方法。一切用起来都感觉很好。

过来一段时间发现如果是需要在Fragment中进行网络请求，这个时候就比较尴尬了，调用getActivity的send方法似乎会出现一些问题，这样就不得不使用接口或者传参给父Activity，在父Activity中进行网络操作，虽然最后可以实现，可是总是感觉不够畅快，且网络请求多了多的话在父Activity中会显得很冗杂，所以这个时候作为一个强迫症患者我觉得有必要来一个封装的完美的Volley网络请求，可以供Activity和Fragment中任意调用。

其实也简单，抽出来，多传入一个参数上下文Context,测试后效果完美，代码如下。

```java

    private RequestQueue mQueue;

    public void send(String url, String target, final Map<String, String> params, final CallBack callBack, final Context context) {
        mQueue = Volley.newRequestQueue(context);
        StringRequest stringRequest = new StringRequest(Request.Method.POST, url, new Response.Listener<String>() {
            @Override
            public void onResponse(String response) {
                Toast.makeText(context, "asdadad", Toast.LENGTH_SHORT).show();
            }
        }, new Response.ErrorListener() {
            @Override
            public void onErrorResponse(VolleyError error) {
                Toast.makeText(context, "asdadad", Toast.LENGTH_SHORT).show();
            }
        }) {
            @Override
            //请求头
            public Map<String, String> getHeaders() throws AuthFailureError {
                HashMap<String, String> localHashMap = new HashMap<String, String>();
                localHashMap.put("Cookie", "");
                localHashMap.put("a_curDateTime", "");
                return localHashMap;
            }
            @Override
            protected Map<String, String> getParams() throws AuthFailureError {
                return super.getParams();
            }
            @Override
            protected Response<String> parseNetworkResponse(NetworkResponse response) {
                return super.parseNetworkResponse(response);
            }
        };
        mQueue.add(stringRequest);
        mQueue.start();
    }
```
CallBack类如下

```java
public abstract class CallBack {

    private Class clzss;

    public CallBack(Class clzss) {
        this.clzss = clzss;
    }

    public Class getClzss() {
        return clzss;
    }

    public abstract void onSuccess(Object result);

    public abstract void onFailure(Integer code);
}

```

