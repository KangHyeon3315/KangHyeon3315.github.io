---
title:  "Spring에서 MongoDB 사용하기"
excerpt: ""

categories:
- Spring
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
{% raw %}
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-mongodb</artifactId>
  <version>2.7.5</version>
</dependency>
  {% endraw %}
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

# Entity 및 Repository 구현

## Entity 구현 (데이터의 구조)

```java
{%raw%}

@Getter
@Setter
@Builder
@Document(collection = "컬렉션 이름")
public class TestEntity {
  private String name;
  private int age;
}
{%endraw%}
```

## Repository 구현 (데이터베이스에 접근하는 인터페이스)

이때 모델과 키값을 제네릭으로 지정한 MongoRepository를 상속받습니다.

```java
{%raw%}

public interface TestRepository extends MongoRepository<TestEntity, String> {
}
{%endraw%}
```

# CRUD 구현하기

## Create (Insert) && Update (Update)

이때 id가 존재하는 경우 update하고, id가 존재하지 않는 경우 insert합니다.

```java
{%raw%}
  TestEntity entity=TestEntity.builder()
  .name("Kim")
  .age(20)
  .build();

  testRepository.save(entity);
  {%endraw%}
```

## Read (find)

우선 모든 document를 요청하는 findAll의 경우 repository에 이미 정의되어있으므로 호출만 하면 됩니다.

```java
{%raw%}
  List<TestEntity> documentList=testRepository.findAll();
  {%endraw%}
```

옵션을 추가하기 위해서는 다음과 같이 Repository Interface에 메소드를 추가해 구현이 가능합니다.

```java
{%raw%}
public interface TestRepository extends MongoRepository<TestEntity, String> {
  TestEntity findByName(String name); // Name Field가 일치하는 Document 찾기

  List<TestEntity> findAllByAge(int age); // Age Field가 일치하는 모든 Document 찾기

  int countByAge(int age); // Age Field가 일치하는 Document의 수 반환
}
{%endraw%}
```

## Delete

```java
{%raw%}

public interface TestRepository extends MongoRepository<TestEntity, String> {

  List<TestEntity> deleteAllByAge(int age);  // Age 필드가 일치하는 Document를 반환하고 제거
}
{%endraw%}
```