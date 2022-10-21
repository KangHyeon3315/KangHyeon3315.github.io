---
title:  "Spring boot JPA 설치 및 사용법 정리" 
excerpt: ""

categories:
  - Spring
tags:
  - []

toc: true
toc_sticky: true
 
date: 2022-08-29
last_modified_at: 2022-08-29
---

이번에는 Spring Boot JPA를 설치해 보도록 하겠습니다.

# 1. JPA란?


JPA는 Java Persistence API의 약자로 자바 ORM 기술에 대한 API 표준 명세입니다.

ORM : Object-Relational Mapping의 약자로 객체와 관계형 데이터베이스를 매핑하는 것을 의미합니다.

더 자세한 설명은 제가 JPA를 공부했던 출처인 아래 링크에서 보시는 것을 추천합니다.

[https://goddaehee.tistory.com/209](https://goddaehee.tistory.com/209)

# 2. Dependency 추가

build.gradle에 아래 dependency를 추가합니다.

```properties
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    runtimeOnly 'mysql:mysql-connector-java'
}
```

# 3. SpringBoot와 MySQL 연동

application.properties에 다음과 같이 mysql과 JPA 설정을 추가합니다.

```properties
# MySQL
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
# DB Source URL
spring.datasource.url=jdbc:mysql://[데이터베이스 IP]:[데이터베이스 PORT]/[데이터베이스 이름]
# DB user name & pw
spring.datasource.username=[유저 ID]
spring.datasource.password=[유저 PW]

# 하이버네이트가 실행한 모든 SQL문을 콘솔로 출력
spring.jpa.show-sql=true
# SQL문을 가독성 있게 표현
spring.jpa.properties.hibernate.format_sql=true

# 파싱 결과를 캐시에 저장하는 기능 끄기
spring.thymeleaf.cache=false
```

# 4. Entity 클래스 생성

Entity 클래스는 실제 데이터베이스 테이블과 매핑이 되는 클래스로 데이터베이스 테이블에 있는 칼럼들을 필드로 가지는 객체입니다.

```java
import lombok.*;

import javax.persistence.*;

@AllArgsConstructor // 모든 필드 값을 파라미터로 받는 생성자 생성
@NoArgsConstructor  // 파라미터가 없는 기본 생성자를 생성
@Builder // 파라미터를 활용하여 빌더 패턴을 자동으로 생성
@Getter // 필드를 가져오는 getter를 사용
@Entity(name="user") // Entity 클래스임을 나타냄, 테이블 이름을 작성
public class User {
    @Id // 기본키(Primary Key) 임을 명시
    @GeneratedValue(strategy = GenerationType.IDENTITY) // 키값의 자동 생성 전략을 설정할 수 있음
    private long id;                           // MySQL에서 AUTO_INCREMENT이므로 GenerationType.IDENTITY를 사용

    @Column(nullable = false, length = 30, unique = true) // 컬럼의 속성값을 설정
    private String username;

    @Column(nullable = false, length = 40)
    private String nickname;

    @Column(nullable = false, length = 100)
    private String password;

    @Column(nullable = false, length = 50)
    private String email;
}
```

# 5. Repository 생성

Repository는 데이터 액세스 계층을 구현하는 인터페이스입니다.

JpaRepository를 상속받은 Interface를 구현해 Repository를 생성합니다.

이때 JpaRepository 제네릭 타입으로 Entity와 기본 키 타입으로 지정합니다.

```java
import com.lab409.backend.entity.User;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    // 기능 구현
}
```

# 6. 사용법 정리

이번에 데이터를 어떻게 접근해 사용하는지 간단하게 정리해 보겠습니다.

우선 단순한 기능부터 확인해 보겠습니다.

```java
@Service
public class UserService {
private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public User save(User user) { 
        userRepository.save(user);   // DB에 저장
        return user;
    }

    public List<User> findAll() {
        return userRepository.findAll();   // 모든 데이터 가져오기
    }

    public Long count() {
        return userRepository.count();   // 데이터 수량 가져오기
    }

    public void delete(User user) {
        userRepository.delete(user);    // 데이터 삭제하기
    }
}
```

그리고 쿼리 메서드를 추가해 보겠습니다.

우선 규칙은 다음과 같이 정리가 가능합니다.

|method| 설명                |
|:---:|:------------------|
|findBy~~ | 쿼리를 요청하는 메서드<br/> ~~부분에 아래 키워드와 Entity 필드 이름을 이용해 쿼리를 작성한다.|
| countBy~~ |쿼리 결과 레코드 수를 요청하는 메서드<br/> ~~부분에 아래 키워드와 Entity 필드 이름을 이용해 쿼리를 작성한다.|

쿼리 메서드에 포함할 수 있는 키워드는 다음과 같습니다

|키워드| 설명                                               | 예제                                                          |
|:---:|:-------------------------------------------------|:------------------------------------------------------------|
| And | 여러 필드를 and로 검색                                   | findByUsernameAndPassword(String username, String password) |
| Or | 여러 필드를 or로 검색                                    | findByUsernameOrEmail(String username, String email)        |
| Between | 필드의 두 값 사이에 있는 항목 검색                             | findByCreatedAtBetween(Date fromDate, Date toDate)          |
| LessThan | 작은 항목 검색                                         | findByAgeLessThan(int age)                                  |
| GreaterThanEqual | 크거나 같은 항목 검색 |  findByAgeGreaterThanEqual(int age)                         |
|Like | like 검색 | findByNameLike(String name)|
| IsNull | null인 항목 검색 | findByJobIsNull() |
| In | 여러 값 중 하나인 항목 검색 | findJobIn(String[] jobs) | 
| OrderBy | 검색 결과를 정렬 | findByEmailOrderByNameAsc(String email)|

위 표를 이용해 findBy 쿼리를 작성하면 다음과 같이 작성이 가능합니다.

UserRepositry에 다음과 같이 메서드를 구현을 하면 됩니다.

```java
@Repository
public interface UserRepository extends JpaRepository<User, String> {
    Optional<User> findByUsername(String username);
}
```


출처

[Spring Boot에서 JPA 사용하기 (MySQL 사용)](https://goddaehee.tistory.com/209)

[[스프링부트 (7)] Spring Boot JPA(1) - 시작 및 기본 설정](https://memostack.tistory.com/155)

[JPA 사용법 (JpaRepository)](https://jobc.tistory.com/120)