---
title: OkHttp
date: 2017-04-20 01:40:34
tags: [AndroidNote, OkHttp]
---

[github](https://github.com/square/okhttp)

**Gradle**

    compile 'com.squareup.okhttp3:okhttp:(insert latest version)'

## Get A URL

- 添加依赖库
- 创建一个okHttpClient实例 
- 实例化一个请求Request
- 实例一个Response返回对象

代码如下:

    OkHttpClient client = new OkHttpClient();
    
    String run(String url) throws IOException {
      Request request = new Request.Builder()
          .url(url)
          .build();
    
      Response response = client.newCall(request).execute();
      return response.body().string();
    }

## POST TO A SERVER

- 添加依赖库
- 创建一个okHttpClient实例
- 实例一个请求的内容体RequestBody并放入提交内容
- 实例化一个请求Request
- 实例一个Response返回对象

代码如下：

    public static final MediaType JSON
        = MediaType.parse("application/json; charset=utf-8");
    
    OkHttpClient client = new OkHttpClient();
    
    String post(String url, String json) throws IOException {
      RequestBody body = RequestBody.create(JSON, json);
      Request request = new Request.Builder()
          .url(url)
          .post(body)
          .build();
      Response response = client.newCall(request).execute();
      return response.body().string();
    }