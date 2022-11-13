---
title:  "ParameterizedTest를 이용해 테스트하기"
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

안녕하세요. 이번 우테코 프리코스에서 피어리뷰를 하면서 Unit Test를 할 때 더 효율적으로 테스트 하는 방법을 피드백 받았던점을 공부 및 기록하기 위해 정리해보고자 합니다!

# ParameterizedTest

유닛 테스트를 진행하다 보면 다양한 값으로 테스트 하는 경우가 있습니다. 이때 일반 Test 어노테이션을 이용해 테스트하는 경우

테스트 함수 내부에서 에서 파라미터를 각각 초기화하고 테스트 하는 코드의 반복이 이루어집니다.

```java
@Test
void test(){
  String param1="TestParam1";
  assertThat(myClass.func(param1)).isEqualTo(expected);

  String param2="TestParam2";
  assertThat(myClass.func(param2)).isEqualTo(expected);

  String param3="TestParam3";
  assertThat(myClass.func(param3)).isEqualTo(expected);
}
```

하지만 ParameterizedTest를 이용하는 경우 다음과 같이 파라미터들을 미리 설정하고 반복해서 테스트를 진행하며 단순화가 가능합니다.

```java
@ParameterizedTest
@ValueSource(strings = {"TestParam1", "TestParam2", "TestParam3"})
void test(String param){
  assertThat(myClass.func(param)).isEqualTo(expected);
}
```

# ValueSource

파라미터 값이 단순한 경우 ValueSource를 이용해 순서대로 파라미터를 넘겨줄 수 있습니다.

이때 테스트 함수는 하나의 기본 타입 파라미터만 전달할 때 사용 가능합니다.

```java
@ParameterizedTest
@ValueSource(ints = {1, 2, 3})
void test(int param){
  assertThat(myClass.func(param)).isEqualTo(expected);
}
```

# MethodSource

ValueSource 어노테이션의 경우 기본 타입의 파라미터 한 종류만 테스트가 가능하다는 단점이 있습니다.

대신 MethodSource를 사용해 Argument를 넘겨주는 메소드를 작성해

다양한 값으로 유닛 테스트가 가능하게 구현할 수 있습니다.

첫번째로, Argument 타입의 Stream 또는 Iterator를 반환하는 static 함수를 구현합니다.  
그다음 ParameterizeTest 함수에서 Parameter를 받아 다음과 같이 테스트를 하도록 구현하면 됩니다.

```java
static Stream<Arguments> getTestArg(){
  ...
  
  return Stream.of(
  Arguments.of(arg1,arg2,arg3),
  Arguments.of(arg4,arg5,arg6),
  Arguments.of(arg7,arg8,arg9)
  );
}

@ParameterizeTest
@MethodSource("getTestArg")
void test(int arg1,List<Integer> arg2,int expected){
    assertThat(myClass.func(arg1, arg2)).isEqualTo(expected);  
}
```