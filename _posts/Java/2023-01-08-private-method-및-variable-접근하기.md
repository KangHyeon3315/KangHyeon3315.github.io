---
title:  "private method 및 variable 접근하기"
excerpt: ""

categories:

- Java

tags:

- []

toc: true
toc_sticky: true

date: 2023-01-08
last_modified_at: 2023-01-08
---

기능들에 대한 유닛 테스트를 하다보면 private로 캡슐화된 메서드나 변수를 테스트 해야하는 필요성을 느낄때가 많다. 하지만 테스트를 위해 public으로 바꾸는 것은
캡슐화를 망치는 방법이다.

그럼에도 private 메서드와 변수를 접근하는 방법이 있습니다.

# Private Method 접근하기

```java

/*
private int func(int a, int b){
  return a + b;
}
*/

MyClass myClass = new MyClass();

// Method method = myClass.getClass().getDeclaredMethod("함수이름", 매개변수타입.class, 매개변수타입.class);
Method method = myClass.getClass().getDeclaredMethod("func", int.class, int.class);
method.setAccessible(true); // 접근이 가능하도록 수정

int result = (int) method.invoke(myClass, 1,2); // 메서드 실행

```


# Private Variable 접근하기

```java

/*
private int myVar = 10;
*/

Field field = myClass.getClass().getDeclaredField("myVar");
field.setAccessible(true);

int value = (int)field.get(myClass);
```