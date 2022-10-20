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

### Maven 환경
    
    <dependencies>
        <dependency>
            <groupId>org.mongodb</groupId>
            <artifactId>mongodb-driver-sync</artifactId>
            <version>4.7.2</version>
        </dependency>
    </dependencies>

### Gradle 환경

    dependencies {
        implementation 'org.mongodb:mongodb-driver-sync:4.7.2'
    }
    

---

# MongoDB 접속 및 데이터베이스 접근하기

      
    MongoClient mongoClient = MongoClients.create("mongodb://데이터베이스주소:포트");
    MongoDatabase database = mongoClient.getDatabase("데이터베이스 이름");

--- 

# 쿼리 필터 방식

    eq("필드네임", "값") // equal
    lt("필드네임", "값") // less than
    

---
     
# Document 찾기 (단일 Document)

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

# Document 찾기 (여러 Document)
    
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