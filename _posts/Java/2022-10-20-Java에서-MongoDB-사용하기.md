---
title:  "Java에서 MongoDB 사용하기" 
excerpt: ""

categories:
  - Java
tags:
  - [Mongo]

toc: true
toc_sticky: true
 
date: 2022-10-20
last_modified_at: 2022-10-20
---

# 종속성 추가하기

## Maven 환경

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-mongodb</artifactId>
  <version>2.7.5</version>
</dependency>

```

## Gradle 환경

```properties
dependencies {
    implementation group: 'org.springframework.boot', name: 'spring-boot-starter-data-mongodb', version: '2.7.5'
}
```

# application.properties 작성

```properties
spring.data.mongodb.host=Database IP
spring.data.mongodb.port=Database Port
spring.data.mongodb.authentication-database=admin
spring.data.mongodb.database=Database Name
spring.data.mongodb.username=Username
spring.data.mongodb.password=Password
```


# Model 및 Repository 구현

## Model 구현 (데이터의 구조)

```java
@Getter
@Setter
@Document(collection="컬렉션 이름")
public class TestModel {
    private String name;
    private int age;
}
```

## Repository 구현 (데이터베이스에 접근하는 인터페이스)
이때 모델과 키값을 제네릭으로 지정한 MongoRepository를 상속받습니다.

```java
public interface TestRepository extends MongoRepository<TestModel, String> {
    TestModel findByName(String name);      // Name Field가 일치하는 Document 찾기
  
    int countByName(String name);           // Name Field가 일치하는 Document의 수
}
```