---
title:  "Spring Cloud Config 알아보기"
excerpt: ""

categories:

- SpringCloud
tags:
- [MicroService, Cloud, Config]

toc: true
toc_sticky: true

date: 2023-01-19
last_modified_at: 2023-01-19
comments: true
---

# Spring Cloud Config란?

Spring Cloud Config는 분산 시스템에서 외부화된 설정 정보를 서버와 클라이언트에 제공하는 시스템입니다. 기본적으로 Git에 설정 정보를 저장하도록 되어있어 레이블이 지정된 버전의 구성 환경을 쉽게 지원할 뿐만 아니라 콘텐츠 관리를 위한 다양한 도구에 액세스할 수 있습니다.

## Spring Cloud Config의 장단점

### 장점

- 여러 서버의 성정 파일을 중앙 서버에서 관리할 수 있습니다.
- 서버를 재배포 하지 않고 설정 파일의 변경사항을 반영할 수 있습니다.

### 단점

- Git 서버 또는 설정 서버에 의해 장애가 전파될 수 있습니다.
- 우선 순위에 의해 설정 정보가 덮어쒸어질 수 있다.

## Config의 읽는 순서

Config의 경우 아래 순서대로 정보를 읽어오기 때문에 읽는 순서가 빠른 정보의 경우 덮어씌어질 수 있습니다.

1. 프로젝트의 applicaiton.yml

2. 설정 저장소의 application.yml

3. 프로젝트의 application-{profile}.yml

4. 설정 저장소의 application-{profile}.yml


# Config Server 구축하기

## Git Repository 구현하기

Git Repository를 하나 생성해주고 Repsotiry에 설정 정보를 저장할 때는 다음과 같은 구조로 이름을 작성하면 됩니다.

- {application}-{profile}.properties
- {application}-{profile}.yml

예시) foo-db.yml

## 의존성 추가하기

```gradle
ext {
	set('springCloudVersion', "2021.0.5")
}

dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-web'
	implementation 'org.springframework.cloud:spring-cloud-config-server'
	
	testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

dependencyManagement {
	imports {
		mavenBom "org.springframework.cloud:spring-cloud-dependencies:${springCloudVersion}"
	}
}
```

## application.yml 설정하기

```properties

server:
  port: 9900

spring:
  cloud:
    config:
      server:
        git:
            # Git Repository URI
            uri: https://github.com/spring-cloud-samples/config-repo
            
            # Branch 이름
            default-label: main

            # 설정 파일을 찾을 경로
            search-paths: config-file/**


spring.cloud.config.server.git.uri=

spring.cloud.config.server.git.default-label=master

spring.cloud.config.server.git.
```

## application 구축하기

Spring Boot Application에 다음과 같이 `@EnableConfigServer` Annotation을 추가해줍니다.

```java
@SpringBootApplication
@EnableConfigServer
public class ConfigServer {
  public static void main(String[] args) {
    SpringApplication.run(ConfigServer.class, args);
  }
}
```

## 테스트하기

Git Repository에 다음과 같이 application-example.properties를 만들어 다음과 같이 테스트용 정보를 작성합니다.

```properties
zuul.route.samplebackendservice=/hello
zuul.route.samplefrontendservice=/
zuul.route.hystrixdashboard=/hystrix-dashboard
```

그리고 Config 서버를 실행하고 [http://localhost:9900/application/example](http://localhost:9900/application/example)에 접속하면 다음과 같이 설정 정보에 대한 JSON 정보를 반환합니다.

```json
{
    "name": "application",
    "profiles": [
        "example"
    ],
    "label": null,
    "version": "424473e5bc35db7fb714cc052f04dc11713941b7",
    "state": null,
    "propertySources": [
        {
            "name": "https://github.com/KangHyeon3315/ConfigStudy.git/application-example.properties",
            "source": {
                "zuul.route.samplebackendservice": "/hello",
                "zuul.route.samplefrontendservice": "/",
                "zuul.route.hystrixdashboard": "/hystrix-dashboard"
            }
        }
    ]
}
```

## Private Repository의 경우

Git의 Repository가 

1. 키 생성하기

먼저, ssh-keygen 명령어를 이용해 ssh 공개키와 개인키를 생성해줍니다.

```text
$ sudo ssh-keygen -C "son2786w@gmail.com" -f myKey # 공개키 / 개인키 생성
$ cat myKey.pub # 공개키 확인
$ cat myKey # 개인키 확인
```
 
2. SSH 키 업로드

    Github에서 Setting > SSH and GPG Keys > New SSH Key를 눌러 공개키를 등록해줍니다.


    <img src='../../assets/images/SpringCloud/MicroService-Config-Server/SSH키등록.png'>


3. application.yml 수정하기

    이제 application.yml에 개인키를 추가해주고 uri를 HTTP 주소에서 SSH 주소로 수정해줍니다.

    <img src='../../assets/images/SpringCloud/MicroService-Config-Server/SSH주소복사.png' style='width:400px'>


    ```yml
    server:
        port: 9900

    spring:
        cloud:
            config:
                server:
                    git:
                        uri: git@github.com:KangHyeon3315/ConfigStudy.git # SSH 주소
                        default-label: main
                        ignore-local-ssh-settings: true
                        private-key: |
                            -----BEGIN OPENSSH PRIVATE KEY-----
                            ...
                            -----END OPENSSH PRIVATE KEY-----
    ```


# Config Client 구축하기

## 의존성 추가하기
```gradle
ext {
    set('springCloudVersion', "2021.0.5")
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.cloud:spring-cloud-starter-config'

    annotationProcessor 'org.projectlombok:lombok'
    compileOnly 'org.projectlombok:lombok'

    annotationProcessor "org.springframework.boot:spring-boot-configuration-processor"

    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

dependencyManagement {
    imports {
        mavenBom "org.springframework.cloud:spring-cloud-dependencies:${springCloudVersion}"
    }
}
```

## application.yml 설정하기

application.yml에 다음 항목들을 작성해줍니다.
- `spring.config.import` :  config server의 uri를 추가해줍니다.

    - `optional:configserver:` Config Server와 통신이 실패하더라도 서버는 정상적으로 동작하도록 설정
    
    - `configserver:` Config Server와 통신을 실패했을 때 예외를 던지고 서버를 멈추도록 설정   

- `spring.cloud.config.name` : 참고하고자 하는 config의 application 이름

- `spring.cloud.config.profile` : 참고하고자 하는 config의 profile 이름


```yml
spring:
  config:
    import: "optional:configserver:http://localhost:9900/"
  cloud:
    config:
      name: application
      profile: example
```

## 설정파일을 가져오는 클래스 구현하기

```java
@Getter
@Setter
@RefreshScope // 설정 정보가 바뀌면 다시 불러오도록 설정
@Configuration
@ConfigurationProperties("zuul.route") // config 값들의 prefix 설정
@EnableConfigurationProperties
@ToString
public class MyConfig {

    private String samplebackendservice;

    private String samplefrontendservice;
    
    private String hystrixdashboard;

}
```

## 테스트하기위한 Rest Controller

```java
@RestController
@RequiredArgsConstructor
public class ConfigController {

    private final MyConfig myConfig;

    @GetMapping("/config")
    public String test() {
        return myConfig.toString();
    }

}
```

## 결과
이제 [http://localhost:8080/config](http://localhost:8080/config)에 접속하면 다음과 같이 Config를 불러오는 것을 확인 할 수 있습니다.

| 위치 | 결과 |
| :-----: | :---: |
| Git <br/> 원본 | <img src='../../assets/images/SpringCloud/MicroService-Config-Server/Config원본.png'> |
| Client  | <img src='../../assets/images/SpringCloud/MicroService-Config-Server/CloudConfigTest.png'> |

# 출처

- [Spring Cloud Config Docs](https://docs.spring.io/spring-cloud-config/docs/current/reference/html/#_spring_cloud_config_server)
- [Spring Cloud Config 도입하기 및 private 레포지토리 SSL로 연결 설정 예시](https://mangkyu.tistory.com/253)
- [Spring Cloud Config Server를 Private Repository와 연동](https://velog.io/@zayson/%EB%82%98-%ED%98%BC%EC%9E%90-%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8-4-Spring-Cloud-Config-Server%EB%A5%BC-Private-Repository%EC%99%80-%EC%97%B0%EB%8F%99)
- [Github SSH Key 등록하기](https://velog.io/@skyepodium/Github-SSH-Key-%EB%93%B1%EB%A1%9D%ED%95%98%EA%B8%B0)
- [Spring: @Configuration 및 @ConfigurationProperties 주석을 결합할 때 잘못된 오류 발생](https://youtrack.jetbrains.com/issue/IDEA-204153)