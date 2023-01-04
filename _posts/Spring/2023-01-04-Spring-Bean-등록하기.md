---
title:  "Spring Bean 등록하기"
excerpt: ""

categories:
- Spring
  tags:
- []

toc: true
toc_sticky: true

date: 2023-01-04
last_modified_at: 2023-01-04
---

# Spring Bean이란?

기본적인 자바 프로그래밍에서는 클래스를 작성 후 객체를 직접 생성해 사용했습니다. 하지만 Spring에서는 직접 객체 생성하는 것이 아니라 Spring이
객체를 생성해 관리할 수 있습니다. 이렇게 Spring으로 생성해 관리되는 인스턴스를 Bean이라고 합니다.   


# Bean 등록하기

## 1. Annotation으로 등록하기

아래 코드와 같이 @Component Annotation을 이용해 Bean으로 등록 가능합니다.

그리고 Spring에서 주로 쓰는 @Controller Annotation과 @Service Annotation또한 내부에 @Component Annotation이 등록되어 있어 Bean으로 등록됩니다.

```java
@Component
public class MyBeanComponent {
    
}
```

## 2. Bean Configuration에 직접 등록하기

우선 아래의 코드와 같이 @Configuration Annotation을 이용해 Spring에 Configuration Class를 지정합니다.

그리고 Bean으로 등록하고자 하는 객체를 반환하는 함수를 만들어 @Bean Annotation을 추가하면 Bean을 등록 가능합니다. 

```java
@Configuration
public class MyBeanConfiguration {
    
    @Bean
    public MyBeanComponent getMyBeanController() {
        return new MyBeanComponent();
    }
}
```

# Bean 객체 가져오기

Spring에 등록한 객체는 아래 코드와 같이 가져올 수 있습니다.

```java

@Controller
public class MyController {
    
    private MyBeanComponent myBeanComponent;
    
    public MyController(MyBeanComponent myBeanComponent) {
        this.myBeanComponent = myBeanComponent;
    }
}

```