---
title:  "Spring Webflux 알아보기"
excerpt: ""

categories:

- Spring
tags:
- [Webflux]

toc: true
toc_sticky: true

date: 2023-01-11
last_modified_at: 2023-01-11
---

메타코딩님의 [Springboot WebFlux](https://www.youtube.com/playlist?list=PL93mKxaRDidFH5gRwkDX5pQxtp0iv3guf) 강좌를 듣고 제가 이해한대로 요약한 내용입니다.

<br/><br/>

# Reactive Programming

## Reactive Programming이란?

### 기존의 기본적인 방식

일반적인 방법으로 어떠한 요청을 했을 때는 응답이 올떄까지 기다리다가 응답을 수신 받았을 때 처리하는 방식으로 진행이 된다.

이러한 방식의 경우 다음 2가지의 단점이 있다.

1. 어떠한 요청을 할때, 응답이 올때까지 대기해야 한다.
2. 요청을 하지 않으면 서버로부터 정보를 수신받을 수 없다.

전통적으로 대기해야 하는 문제를 해결하기 위해 스레드를 생성해 응답하도록 구현했다. 하지만 이 경우 스레드를 전환하는 시간이 늘어나 총 시간이 늘어날 수 있다.

### Reactive Programming를 통한 문제 해결

위에서 말한 두가지 문제를 Reactive Programming을 통해 해결할 수 있다.

1. 비동기 방식을 이용해 대기하지 않고 다른 일을 처리하도록 구현한다.
2. Stream(Flux)을 이용해 정보를 전송한다. -> 지속적인 응답이 가능하다. -> SSE 프로토콜

#### Mono란?

- Flux를 이용한 방식은 연결을 끊지 않기 때문에 여러번 응답이 가능하다. 하지만 개발 상황에 따라 1번만 응답 후 응답이 필요하지 않는경우가
  있을 수 있다.이 경우 Mono를 이용해 한번만 응답하도록 구현이 가능하다.


<br/><br/>

# Reactive Streams을 이용한 구독 패턴

reactive streams 아래 예시와 같이 Pusblisher를 이용해 Stream을 정의하고 Subscriber를 이용해 신호를 처리하는 방식을 이용합니다. 이러한 구독 패턴의 동작방식을 이해해야 Flux의 동작 방식을 이해할 수 있습니다.

```gradle
// build.gradle

implementation 'org.reactivestreams:reactive-streams:1.0.3'
```

## Publisher 만들기

```java
public class MyPublisher implements Publisher<Integer> {

    Iterable<Integer> its = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

    @Override
    public void subscribe(Subscriber<? super Integer> s) {
        System.out.println("구독!");

        MySubscription subscription = new MySubscription(s, its);
        System.out.println("구독 정보 생성 완료");

        s.onSubscribe(subscription);
    }
}
```

## Subscriber 만들기

```java
public class MySubscriber implements Subscriber<Integer> {

    private Subscription s;
    @Override
    public void onSubscribe(Subscription s) {
        System.out.println("구독정보 수신 완료");
        this.s = s;
        s.request(1); // (소비자가 한번에 처리할 수 있는 개수 요청)
    }

    @Override
    public void onNext(Integer integer) {
        System.out.printf("구독 데이터 수신 (%d)\n", integer);

        // 구독한 데이터룰 처리

        s.request(1); // 이후 구독 데이터가 필요할 때 요청
    }

    @Override
    public void onError(Throwable t) {
        System.out.println("구독 중 에러");
    }

    @Override
    public void onComplete() {
        System.out.println("구독 완료!");
    }
}
```

## Subscription 만들기

```java 
// 구독 정보 (구독자, 어떤 데이터를 구독할지)
public class MySubscription implements Subscription {

    private Subscriber s;
    private Iterator<Integer> it;
    public MySubscription(Subscriber s, Iterable<Integer> its) {
        this.s = s;
        this.it = its.iterator();
    }

    @Override
    public void request(long n) {
        while (n > 0) {
            if(it.hasNext()) {
                s.onNext(it.next());
            } else {
                s.onComplete();
                break;
            }
            n--;
        }
    }

    @Override
    public void cancel() {

    }
}
```

## 동작 순서

```java
    // 1. Publisher 생성
    MyPublisher pub = new MyPublisher();

    // 2. Subscriber를 생성 후 publisher에 구독하기
    MySubscriber sub = new MySubscriber();
    pub.subscribe(sub);
```


<br/><br/>

# WebFlux 구현하기


## Project 준비하기

1. Spring Reactive Web 설치

    ```gradle
    implementation 'org.springframework.boot:spring-boot-starter-webflux'

    testImplementation 'io.projectreactor:reactor-test' // 테스트를 위한 도구
    ```

2. R2DBC 설치하기

    Spring에서 일반적으로 사용하는 JDBC는 Blocking 방식으로 쿼리를 수행하고 그에 대한 결과를 받기까지 항상 Blocking이 되어 해당 쓰레드가 대기하는 문제가 있습니다.
    
    이러한 방식은 Flux에서 non-blocking 방식의 이점을 하나도 못얻기 때문에 non-blocking으로 사용이 가능한 R2DBC 모듈을 사용합니다.

    ```gradle
    implementation "org.springframework.boot:spring-boot-starter-data-r2dbc"
	implementation 'com.github.jasync-sql:jasync-r2dbc-mysql:2.1.8'
    ```

<br/>

## 데이터베이스 준비하기

### application.properties
```properties
spring.r2dbc.url=r2dbc:mysql://MY_MYSQL_URL
spring.r2dbc.username=username
spring.r2dbc.password=password

spring.r2dbc.pool.max-size=100
spring.r2dbc.pool.validation-query=SELECT 1

```

### Entity

```java
@Data
@Getter
@Table("user")
@RequiredArgsConstructor
public class User {
    @Id
    private Long id;
    private final String firstName;
    private final String lastName;
}
```

### Repository

```java
public interface UserRepository extends ReativCrudRepository<User, Long> {

}
```

R2DBC Repository는 JPA에서 findAll()을 했을 때 List\<User\>와 같이 반환하는 것이 아니라 Flux\<User\>를 반환한다. 그래서 아래 코드와 같이 바로 WebFlux로 보내줄 수 있다.

<br/>

## RestController 구현하기

```java

@RestController
public class MyController {

    private final UserRepository userRepository;

    // 어떤 A 요청과 B 요청이 있을 때 두 요청 다 Flux Stream인데 두 스트림을 하나의 스트림으로 합쳐주는 기능
    private final Sinks.Many<Customer> sink; 
    

    public CustomerController(UserRepository userRepository) {
        this.userRepository = userRepository;
        this.sink = Sinks.many().multicast().onBackpressureBuffer();
    }



    @GetMapping("/user1")
    public Flux<User> findAll1() {
        // 데이터를 전부 받은 경우 반환
        return customerRepository.findAll();
    }

    @GetMapping("/user/mono")
    public Mono<User> find(String id) {
        // 하나의 데이터만 처리
        return customerRepository.findById(id);
    }

    @GetMapping(value="/user2", produces = MediaType.APPLICATION_STREAM_JSON_VALUE)
    public Flux<User> findAll2() {
        // 데이터를 받을 때마다 반환 (데이터를 전부 전송한 경우 연결 종료)
        // 이때 위의 구독 패턴을 이용
        return customerRepository.findAll();
    }



    @GetMapping(value="/user/sse") // ServerSentEvent 리턴시 produces=MediaType.TEXT_ENVENT_STREAM_VALUE 생략 가능
    public Flux<ServerSentEvent<User>> findAllSSE() {

        // sink에 합쳐진 데이터가 있는 경우 데이터를 전송
        return sink.asFlux().map(u -> ServerSentEvent.builder(u).build())
        .doOnCancel(() -> { // 연결이 취소됐을 때 핸들링
            // 마지막 데이터라고 알려주며 구독자가 onComplete()를 호출하도록 한다.
            // 이작업을 수행해야 다시 연결했을 때 다시 데이터를 받을 수 있다.
            sink.asFlux().blockLast(); 
        });
    }

    @PostMapping("/user")
    public Mono<User> save() {
        return userRepository.save(new User("val1", "val2")).doOnNext(u -> {
            sink.tryEmitNext(u); // sink에 데이터 추가
        });
    }

}

```

# 출처
- https://www.youtube.com/playlist?list=PL93mKxaRDidFH5gRwkDX5pQxtp0iv3guf