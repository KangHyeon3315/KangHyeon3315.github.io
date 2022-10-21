---
title:  "OKHttp로 REST API 호출하기" 
excerpt: ""

categories:
  - Java
tags:
  - [http]

toc: true
toc_sticky: true
 
date: 2022-05-01
last_modified_at: 2022-05-01
---

# GET 방식

## 동기

```java
OkHttpClient client = new OkHttpClient();

Request request = new Request.Builder()
    .url("https://api.upbit.com/v1/market/all?isDetails=false")
    .get()
    .addHeader("Accept", "application/json")
    .build();
Response response = client.newCall(request).execute();

String json_data = response.body().string();
```

## 비동기

```java
OkHttpClient client = new OkHttpClient();

Request request = new Request.Builder()
    .url(url)
    .get()
    .addHeader("Accept", "application/json")
    .build();

client.newCall(request).enqueue(new Callback() {
    @Override
    public void onFailure(Call call, IOException e) { // 실패에 대한 비동기 callback
        System.out.println("GET 요청 실패" + e.toString());
    }
        
    @Override
    public void onResponse(Call call, Response response) throws IOException { // 성공에 대한 비동기 callback
        String json_data = response.body().string();
    }
});
```

# POST 방식

## 동기

```java
Request request = new Request.Builder()
    .url(url)
    .post(RequestBody.create(MediaType.parse("application/json"), json_str))
    .build();

Response response = client.newCall(request).execute();

String json_data = response.body().string();
```

## 비동기

```java
Request request = new Request.Builder()
    .url(url)
    .post(RequestBody.create(MediaType.parse("application/json"), json_str))
    .build();

client.newCall(request).enqueue(new Callback() {
    @Override
    public void onFailure(Call call, IOException e) { // 실패에 대한 비동기 callback
        System.out.println("POST 요청 실패" + e.toString());
    }

    @Override
    public void onResponse(Call call, Response response) throws IOException { // 성공에 대한 비동기 callback
        String json_data = response.body().string();
    }
});
```
