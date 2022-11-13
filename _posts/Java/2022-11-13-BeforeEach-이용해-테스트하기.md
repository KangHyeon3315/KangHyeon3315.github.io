---
title:  "BeforeAll, BeforeEach, AfterAll, AfterEach 등을 이용해 테스트하기"

excerpt: ""

categories:

- Java

tags:

- [TDD, JUnit5, AssertJ]

toc: true
toc_sticky: true

date: 2022-11-13
last_modified_at: 2022-11-13
---

유닛 테스트를 작성하보면 같은 데이터를 준비해야 하는 경우가 있습니다.

이때 각 테스트에서 준비하면 코드가 복잡해지면서 코드의 양이 복잡해지고, 가독성이 떯어지는 문제가 있습니다.

이번 우테코 프리코스에서 피어리뷰를 하며 이러한 문제가 있는 경우 BeforeEach와 같은 방법을 이용해 코드의 재사용성을 향상시킬 수 있다는 것을
알게 되었습니다.

그래서 이번에 공부 겸 정리해보고자 합니다

# fixture란?

fixture : 테스트를 수행하는 데 필요한 정보나 오브젝트를 픽스처 라고 합니다.

# BeforeAll

BeforeAll은 테스트 클래스에 있는 테스트를 실행하기 전 한번만 실행되는 함수입니다.

리턴타입이 void인 static 함수로 구현해야합니다.

# BeforeEach

Test, RepeatedTest, ParameterizedTest 등을 호출하기 전에 먼저 실행되는 함수입니다.

로직이 복잡해지고 테스트에 여러 초기화가 필요한 경우 BeforeEach로 간결하게 구현할 수 있습니다.

단, 초기화 메소드들 사이의 실행 순서는 보장되지 않습니다. 순서가 보장되어야 하는 경우 Order 어노테이션을 사용해 순서를 지정해야합니다.

# AfterAll

테스트하는 클래스의 테스트를 마친 이후에 메소드가 실행되는 함수입니다.

BeforeAll처럼 리턴타입이 void인 static 함수로 구현해야합니다.

# AfterEach

Test, RepeatedTest, ParameterizedTest 등을 호출한 이후 실행되는 함수입니다.

테스트를 실패해도 호출됩니다.

# 예시

```java

class MyTest {
    
    private MyClass myClass;
    
    @BeforeEach
    void setUp() {
      this.myClass = new MyClass();
    }
    
    @Test
    void test() {
        assertThat(myClass.test()).isEqualTo(true); // 테스트 수행
    }
}

```

