---
title:  "Spring Mockito 사용하기"
excerpt: ""

categories:

- Spring
tags:
- [JUnit, Mockito]

toc: true
toc_sticky: true

date: 2023-01-12
last_modified_at: 2023-01-12
---

# Mockito란?

Mockito는 가짜 객체를 만들어 프로그래머가 직접 행동을 관리하는 Mock 객체를 만들고 관리 및 검증을 할 수 있는 방법을 제공하는 프레임워크 입니다.

테스트를 구현할 때 데이터베이스나 외부 API같은 개발자가 컨트롤하기 어려운 객체를 테스트에 사용할 때 많은 불편함이 있습니다. 이때 Mock 객체를 만들어 작성하면 편리하게 테스트가 가능합니다.


# Mock 등록하기

Test 코드에서 Mock 객체를 사용하기 위해서는 @Mock Annotation을 이용해 해당 객체가 Mock 객체라고 정의할 수 있습니다.

또는 Mockito.mock 메서드를 이용해 Mock 객체를 생성할 수 있습니다.

```java

@ExtendWith(MockitoExtension.class)
class MyTest {
    static MyService myService;

    @Mock
    UserRepository userRepository;

    @BeforeAll
    static void setUp() {
        myService = Mockito.mock(MyService.class);
    }
}

```

# Mock 객체 행동 정의하기

기본적으로 모든 Mock 객체의 행동은 null 값을 반환하도록 정의되어 있습니다. Collection과 같은 객체는 비어있는 값을 리턴합니다.

따라서 Mock 객체의 행동을 정의해 개발자가 원하는 결과를 반환하도록 수정해야 합니다.

```java
@Test
void myTest() {
    // 값 반환하기
    when(userRepository.findById(1L)).thenReturn(Optional.of(new User()));
    userRepository.findById(1L); // User 객체 반환
}

@Test
void myThrowTest() {
    // 예외 발생시키기
    when(userRepository.findById(1L)).thenThrow(new RuntimeException());
    userRepository.findById(1L); // RuntimeException 발생
}

@Test
void myThrowTest2() {
    // 예외 발생시키기
    doThrow(new RuntimeException()).when(userRepository).findById(1L);
    userRepository.findById(1L); // RuntimeException 발생
}

@Test
void myAnyTest() {
    // 입력 매개변수에 상관없이 값 반환하기
    when(userRepository.findById(any())).thenReturn(Optional.of(new User()));

    userRepository.findById(1L); // User 객체 반환
}

@Test
void myMethodChainingTest() {
    // 메소드를 여러번 호출 할 때 각각 다른 행동을 하도록 하기
    when(userRepository.findById(any()))
        .thenReturn(Optional.of(new User()));
        .thenThrow(new RuntimeException());

    userRepository.findById(1L); // User 객체 반환
    userRepository.findById(1L); // RuntimeException 발생
}
```

# Mock 객체 확인하기

테스트를 하다보면 Mock 객체가 어떻게 사용되었는지 확인해야 할 때가 있습니다.

```java

@Test
void myTest() {
    // userRepository.findById(2L);가 2번 호출됐는지 확인
    verify(userRepository, times(2)).findById(2L); 
}

@Test
void myTest() {
    // userRepository.findById(2L);가 한번도 호출이 안됐는지 확인
    verify(userRepository, never()).findById(2L); 
}

@Test
void myTest() {
    // 메서드 호출 순서 확인하기
    InOrder inOrder = inOrder(userRepository);
    inOrder.verify(userRepository).findById(1L); 
    inOrder.verify(userRepository).findById(2L); 
}

@Test
void myTest() {
    // 메서드가 특정 시간 이내에 호출 되었는지 확인
    verify(userRepository, timeout(100),times(1)).findById(1);
}


```


# 출처
[Mockito란? Mockito 사용하기 - 책 읽는 개발자 - 티스토리](https://scshim.tistory.com/439)