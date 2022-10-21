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
<dependencies>
    <dependency>
        <groupId>org.mongodb</groupId>
        <artifactId>mongodb-driver-sync</artifactId>
        <version>4.7.2</version>
    </dependency>
</dependencies>
```

## Gradle 환경

```properties
dependencies {
    implementation 'org.mongodb:mongodb-driver-sync:4.7.2'
}
```

# 데이터베이스 접근하기

```java
MongoClient mongoClient = MongoClients.create("mongodb://데이터베이스주소:포트");
MongoDatabase database = mongoClient.getDatabase("데이터베이스 이름");
```

# 쿼리 필터 방식

```java
eq("필드네임", "값") // equal
lt("필드네임", "값") // less than
gt("필드네임", "값") // greater than
```

# Document 찾기

## 단일 Document

```java
MongoCollection<Document> collection = database.getCollection("콜렉션 이름");

// 필드를 명시적으로 포함하거나 제외합니다.
Bson projectionFields = Projections.fields(
            Projections.include("title", "imdb"), // title과 imdb 포함
            Projections.excludeId());             // _id 제외

Document doc = collection.find(eq("title", "The Room")) // eq("title", "The Room") : 쿼리 필터  
            .projection(projectionFields)           // title과 imdb 필드를 포함하고 _id 필드를 제거
            .sort(Sorts.descending("imdb.rating")) // Document를 정렬
            .first();     // 단일 Document(첫번쨰)만 요청하기
if (doc == null) {
    System.out.println("No results found.");
} else {
    System.out.println(doc.toJson());
}
```

## 여러 Document

```java
MongoCollection<Document> collection = database.getCollection("콜렉션 이름");

// 필드를 명시적으로 포함하거나 제외합니다.
Bson projectionFields = Projections.fields(
            Projections.include("title", "imdb"), // title과 imdb 포함
            Projections.excludeId());             // _id 제외

MongoCursor<Document> cursor = collection.find(eq("title", "The Room")) // eq("title", "The Room") : 쿼리 필터  
            .projection(projectionFields)           // title과 imdb 필드를 포함하고 _id 필드를 제거
            .sort(Sorts.descending("title"))        // Document를 정렬

try {
    while(cursor.hasNext()) {
        System.out.println(cursor.next().toJson());
    }
} finally {
    cursor.close();
}
```

# Document 삽입 

## 단일 Document
Document 객체를 생성해 insertOne 메서드를 이용해 삽입합니다.

```java
MongoCollection<Document> collection = database.getCollection("콜렉션 이름");
try {
    // Document 객체 생성 
    Document doc = new Document()
            .append("_id", new ObjectId())
            .append("title", "Ski Bloopers")
            .append("genres", Arrays.asList("Documentary", "Comedy"));

    InsertOneResult result = collection.insertOne(doc);

    System.out.println("Success! Inserted document id: " + result.getInsertedId());

} catch (MongoException me) {
    System.err.println("Unable to insert due to an error: " + me);
}
```
## 여러 Document
Document List를 생성해 insertMany 메서드를 이용해 삽입합니다.
```java
MongoCollection<Document> collection = database.getCollection("콜렉션 이름");
List<Document> docList = Arrays.asList(
        new Document().append("title", "Short Circuit 3"),
        new Document().append("title", "The Lego Frozen Movie"));

try {
    InsertManyResult result = collection.insertMany(docList);
    System.out.println("Inserted document ids: " + result.getInsertedIds());
} catch (MongoException me) {
    System.err.println("Unable to insert due to an error: " + me);
}
```

# Document 업데이트 

## 단일 Document

```java
MongoCollection<Document> collection = database.getCollection("콜렉션 이름");

// 쿼리 작성
Document query = new Document().append("title",  "Cool Runnings 2");

// 업데이트 내용을 작성
Bson updates = Updates.combine(
    Updates.set("runtime", 99),     // 필드 값을 지정
    Updates.addToSet("genres", "Sports"),   // 필드를 추가해 값을 지정 (필드가 이미 있으면 동작X) 
    Updates.currentTimestamp("lastUpdated") // lastUpdated 필드에 현재 시간 추가
);

// 업데이트 옵션 작성
UpdateOptions options = new UpdateOptions().upsert(true); // 쿼리에 해당하는 Document를 찾지 못한 경우 삽입

try {
    // 업데이트
    UpdateResult result = collection.updateOne(query, updates, options);
    System.out.println("Modified document count: " + result.getModifiedCount());
    System.out.println("Upserted id: " + result.getUpsertedId());
  
} catch (MongoException me) {
    System.err.println("Unable to update due to an error: " + me);
}
```

## 여러 Document

```java
MongoCollection<Document> collection = database.getCollection("콜렉션 이름");

// 쿼리 작성
Bson query = gt("num_mflix_comments", 50);

// 업데이트 내용을 작성
Bson updates = Updates.combine(
        Updates.addToSet("genres", "Frequently Discussed"),
        Updates.currentTimestamp("lastUpdated"));

try {
    UpdateResult result = collection.updateMany(query, updates);    // 쿼리에 해당하는 모든 Document를 업데이트
    System.out.println("Modified document count: " + result.getModifiedCount());
} catch (MongoException me) {
    System.err.println("Unable to update due to an error: " + me);
}
```

## Document 교체 (replace)
 
```java

MongoCollection<Document> collection = database.getCollection("콜렉션 이름");

// 쿼리 작성
Bson query = eq("title", "Music of the Heart");

// Repalce Document 작성 
Document replaceDocument = new Document()
        .append("title", "50 Violins")
        .append("fullplot", " A dramatization of the true story of Roberta Guaspari who co-founded the Opus 118 Harlem School of Music");

// Option 작성
ReplaceOptions option = new ReplaceOptions().upsert(true);

// Replace 
UpdateResult result = collection.replaceOne(query, replaceDocument, option);

```

# Document 삭제

## 단일 Document

```java

MongoCollection<Document> collection = database.getCollection("콜렉션 이름");

// 쿼리 작성
Bson query = eq("title", "The Garbage Pail Kids Movie");

try {
    // 쿼리에 해당하는 Document 1개 제거
    DeleteResult result = collection.deleteOne(query);
    System.out.println("Deleted document count: " + result.getDeletedCount());
    
} catch (MongoException me) {
    System.err.println("Unable to delete due to an error: " + me);
}
```

## 여러 Document 
```java

MongoCollection<Document> collection = database.getCollection("콜렉션 이름");

// 쿼리 작성
Bson query = eq("title", "The Garbage Pail Kids Movie");

try {
    // 쿼리에 해당하는 모든 데이터 제거
    DeleteResult result = collection.deleteMany(query);
    System.out.println("Deleted document count: " + result.getDeletedCount());
} catch (MongoException me) {
    System.err.println("Unable to delete due to an error: " + me);
}
```
