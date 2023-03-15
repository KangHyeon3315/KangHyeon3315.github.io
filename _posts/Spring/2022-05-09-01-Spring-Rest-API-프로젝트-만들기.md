---
title:  "Spring Rest API 프로젝트 만들기" 
excerpt: ""

categories:
  - Spring
tags:
  - [Java, Rest]

toc: true
toc_sticky: true
 
date: 2022-05-09
last_modified_at: 2022-05-09
---

이번에는 Java에서 Rest 서버를 구현하기 위해 Spring Boot 환경에서 프로젝트를 만드는 방법을 정리해 보겠습니다.

개발 환경은 다음과 같습니다.

IDE : IntelliJ

JDK : adoptOpenJDK 1.8

Type : Maven

우선 새로운 프로젝트를 생성합니다.

![](../../assets/images/spring/SpringRestAPI-프로젝트-만들기/스크린샷_2022-05-06_오후_5.22.44.png)

![](../../assets/images/spring/SpringRestAPI-프로젝트-만들기/스크린샷_2022-05-06_오후_5.25.34.png)

이제 기본적인 Dependency를 설치해야 하는데 우선 기본적인 것만 설치하겠습니다.

- Spring Boot devTools :개발하면서 자동 빌드를 지원해 줍니다. (옵션)

- Lombok : 자바에서 기본이 되는 getter, setter, constructor들을 자동으로 관리해 줍니다.

- Spring Web : Spring의 Rest Service를 제공할 수 있게 도와줍니다.

![](../../assets/images/spring/SpringRestAPI-프로젝트-만들기/스크린샷_2022-05-06_오후_5.28.30.png)

이제 위에서 설치한 Lombok을 위한 프로젝트 활성화를 진행해 주겠습니다.

Preferences -> Build, Execution, Deployment -> Compiler -> Annotation Processor 순으로 이동해 줍니다.

그리고 Enable annotation processing을 체크해 줍니다.

![](../../assets/images/spring/SpringRestAPI-프로젝트-만들기/스크린샷_2022-05-06_오후_5.29.53.png)

이제 실행하면 다음과 같이 Application을 실행하면 다음과 같이 실행이 됩니다.

![](../../assets/images/spring/SpringRestAPI-프로젝트-만들기/스크린샷_2022-05-06_오후_5.34.15.png)

이제 Rest Controller를 구현해 보겠습니다.

프로젝트에 controller 패키지를 만들어 이곳에 다음과 같이 controller를 만들어보겠습니다.

```java
// ControllerTest.java
package com.example.backend.controller;

import com.example.backend.service.ServiceTest;
import org.springframework.http.*;
import org.springframework.web.bind.annotation.*;

@RestController // 해당 클래스가 RestController라는 의미이다.
@RequestMapping("/test") // http://localhost:8080/test와 같이 경로를 맵핑시킨다.
public class ControllerTest {
// Get 요청 처리기능
    @GetMapping("/get")
    public ResponseEntity<?> getReq() {
        return new ResponseEntity<>("Get Response", HttpStatus.OK);
    }

    // Post 요청 처리 기능
    @PostMapping("/post")
    ResponseEntity<?> postReq() {
        return new ResponseEntity<>("Post Response", HttpStatus.OK);
    }
}
```

그리고 Service를 구현해 보겠습니다.

```java
package com.example.backend.service;

import org.springframework.stereotype.Service;

@Service // 컨트롤러에 반환되는 비즈니스 로직임을 알려준다.
public class ServiceTest {
    public String test(String str) {
        return "Test " + str;
    }
}
컨트롤러에서 다음과 같이 가져와서 비즈니스 로직을 구현하면 됩니다.

public class ControllerTest {
    private final ServiceTest serviceTest;

    public ControllerTest(ServiceTest serviceTest) {
        this.serviceTest = serviceTest;
    }
}
```


마지막으로 controller에서 파라미터를 받는 방법을 정리하고 끝내겠습니다.

```java
@GetMapping("/required")
ResponseEntity<?> getReq(@RequestParam(required=false) String key) {  // param의 필수 여부를 정할 수 있습니다.
    return new ResponseEntity<>("Get Response param : " + key, HttpStatus.OK);
}

@GetMapping("/path_var/{Pathnum}")
ResponseEntity<?> getReq1(@PathVariable("Pathnum") String key) { // @GetMapping의 PathVariable을 Param으로 받습니다.
    return new ResponseEntity<>("Get Response Path Variable : " + key, HttpStatus.OK);
}

@GetMapping("/body")
ResponseEntity<?> getReq2(@RequestBody String key) { // Body에서 해당 파라미터를 가져옵니다.
    return new ResponseEntity<>("Get Response Request Body : " + key, HttpStatus.OK);
}

@GetMapping("/header")
ResponseEntity<?> getReq3(@RequestHeader String key) { // Header에서 해당 파라미터를 가져옵니다.
    return new ResponseEntity<>("Get Response Header : " + key, HttpStatus.OK);
}
```