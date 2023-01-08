---
title:  "jsoup을 이용해 HTML 파싱하기"
excerpt: ""

categories:

- Java

tags:

- [jsoup, gradle, parsing]

toc: true
toc_sticky: true

date: 2023-01-08
last_modified_at: 2023-01-08
---

# build.gradle에 jsoup 추가하기
```gradle
implementation 'org.jsoup:jsoup:1.15.3'
```

# HTML String 파싱하기

```java
import org.jsoup.Jsoup;
import org.jsoup.nodes.Document;

...

// 문서 전체의 경우
Document doc = Jsoup.parse(htmlStr);

// 문서 일부의 경우
Document doc = Jsoup.parseBodyFragment(htmlStr);

// url의 경우
Document doc  Jsoup.connect("http://weburl.com/").get();
```

# 문서 데이터 추출 및 조작

## Element 찾기

### 여러 Element 찾기 
```java
Elements elements = doc.getElementsById("id");

Elements elements = doc.getElementsByTag("Tag Name");

Elements elements = doc.getElementsByClass("class name");
```

### 단일 Element 찾기
```java
Element element = doc.getElementById("id");

Element element = doc.getElementByTag("Tag Name");

Element element = doc.getElementByClass("class name");
```

### CSS 스타일로 선택하기
```java
Elements elements = doc.getElementsById(".myDiv");
```


## 데이터 추출하기

```java

for(int i = 0 ; i < elements.size(); i++) {
  Element element = elements.get(i);
  
  String text = element.text();
  String attribute = element.attr("key");
  String id = element.id();
  String className = element.className();
}

```