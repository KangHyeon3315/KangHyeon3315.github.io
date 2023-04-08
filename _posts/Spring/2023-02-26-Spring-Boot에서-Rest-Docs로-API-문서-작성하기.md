---
title:  "Spring Boot에서 Rest Docs로 API 문서 작성하기"
excerpt: ""

categories:

- Spring

tags:
- [API Docs, Spring, Rest Docs]

toc: true
toc_sticky: true

date: 2023-02-26
last_modified_at: 2023-02-26
---

# Spring Rest Docs란?

## API 문서 자동화가 필요한 이유

API 문서를 직접 작성한다는 것은 매우 힘든 일입니다. API 문서를 작성 후 요구사항이 변하며 요청 데이터나 응답 결과가 바뀐다면 개발자가 수동으로 업데이트해야 합니다.

하지만 실제로는 변경사항을 실시간 업데이트가 안돼 API 문서의 정보가 실제 코드와는 다른 경우가 생기기 쉽습니다. 이러한 문제는 API 문서의 신뢰성을 떨어트리고 협업에 많은 지장을 주게 됩니다.

이러한 문제를 해결하기위해 Swagger나 Spring Rest Docs 같은 문서 자동화 툴을 이용해 API 문서를 자동으로 작성하도록 구현해 개선이 가능합니다.

<br/>

## Spring Rest Docs란?

Spring Rest Docs는 REST API 문서를 자동으로 만들어주는 툴입니다. 이때 테스트를 기반으로 문서를 작성하는데 테스트가 실패한다면 문서가 작성되지 않아 신뢰성을 더 높일 수 있다는 특징이 있습니다.


<br/>

## Swagger와 Rest Docs의 차이

**Swagger**

특징
- Annotation 기반으로 문서를 작성합니다.

장점

- 사용하기 쉽습니다.
- 화면에서 API 테스트가 가능합니다.

단점

- 코드에 문서와 정보가 필요합니다.
- Rest Docs에 비해 신뢰성이 떨어집니다.

**Rest Docs**

특징
 - 테스트 기반으로 문서를 작성합니다.

장점
- api 명세 최신화가 강제이기 떄문에 신뢰성이 높습니다.
- 비즈니스 코드에 문서 정보가 필요 없습니다.

단점
- 어렵습니다.

<br/>

# 테스트 코드 작성하기

이제 Rest Docs를 작성하기 전에 MockMvc를 이용해 요청을 테스트하는 코드부터 추가해주겠습니다.

개발 환경은 다음과 같습니다.

- Java : OpenJDK 17

- Gradle : 7.6

- SpringBoot : 3.0.2

기존 코드의 경우 [Spring으로 마이크로서비스 구축하기 (2) - 간단한 Microservice Project 만들기](../../springcloud/Spring으로-마이크로서비스-구축하기-(2)/)에서 확인이 가능합니다.

<br/>

## RestTemplate를 Bean으로 만들기

Composite 서버의 경우 RestTemplate를 이용해 다른 마이크로서비스에 요청을 보내고 데이터를 수신받도록 구현이 되어 있습니다.

하지만 테스트하는 경우 다른 마이크로서비스가 실행중이지 않은 상태이기 때문에 RestTemplate를 Mock 객체로 만들 필요가 있습니다.

따라서 RestTemplate를 Bean 객체로 등록을 하고 테스트코드에서 MockBean으로 정의를 해주겠습니다.

```java
@Configuration
public class MyConfig {

    @Bean
    RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

```java
public CompositeControllerImpl(
        @Value("${app.post.host}") String postHost,
        @Value("${app.post.port}") String postPort,
        @Value("${app.comment.host}") String commentHost,
        @Value("${app.comment.port}") String commentPort,
        RestTemplate restTemplate
) {
    POST_URL = String.format("http://%s:%s/post", postHost, postPort);
    COMMENT_URL = String.format("http://%s:%s/comment", commentHost, commentPort);

    this.restTemplate = restTemplate; // 수정
    mapper = new ObjectMapper();
    mapper.registerModule(new JavaTimeModule());
}
```

<br/>

## 테스트 구현

우선 다음 코드와 같이 RestTemplate를 MockBean으로 정의하고 MockMvc를 가져오고 요청에 대한 테스트 코드를 작성합니다.

```java
@WebMvcTest(CompositeControllerImpl.class)
public class CompositeControllerImplTest {

    @MockBean
    RestTemplate restTemplate;

    @Autowired
    @SuppressWarnings("SpringJavaInjectionPointsAutowiringInspection") // IntelliJ에서 mockMvc를 Autowired 할 수 없다는 에러 억제 (실제 코드 실행시 문제없이 Autowired됨)
    MockMvc mockMvc;

    @Test
    public void getPostTest() throws Exception {
        int postId = 12;
        
        // Mock 객체에서의 행동 정의
        when(restTemplate.getForObject(ArgumentMatchers.anyString(), ArgumentMatchers.eq(Post.class)))
                .thenReturn(new Post(postId, "post title " + postId, "post author " + postId, "contents " + postId));

        when(
                restTemplate.exchange(
                        ArgumentMatchers.anyString(),
                        ArgumentMatchers.eq(HttpMethod.GET),
                        ArgumentMatchers.isNull(),
                        ArgumentMatchers.eq(new ParameterizedTypeReference<List<Comment>>() {
                        })
                )
        ).thenReturn(
                ResponseEntity.of(Optional.of(
                        List.of(
                                new Comment(postId, 1, "작성자1", "내용1"),
                                new Comment(postId, 2, "작성자2", "내용2")
                        )
                ))
        );

        mockMvc.perform(get("/composite/" + postId)))
                .andExpect(status().isOk())
                // 기타 테스트 코드 생략
    }

}
```

<br/>

# Spring Rest Docs 적용하기

Rest Docs를 작성하기 위해서는 다음과 같이 build.gradle 파일을 설정해줍니다.

<br/>

## build.gradle 작성하기

```gradle
plugins {
	id 'java'
	id 'org.springframework.boot' version '3.0.2'
	id 'io.spring.dependency-management' version '1.1.0'

	id "org.asciidoctor.jvm.convert" version "3.3.2" // Asciidoctor 플러그인 추가
}

group = 'com.microblog'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '17'

configurations {
	asciidoctorExt // Asciidoctor를 확장하는 의존성에 대한 asciidoctorExt 구성을 선언

	compileOnly {
		extendsFrom annotationProcessor
	}
}

repositories {
	mavenCentral()
}

dependencies {
	implementation project(':api')
	implementation project(':util')

	implementation 'org.springframework.boot:spring-boot-starter-actuator'
	implementation 'org.springframework.boot:spring-boot-starter-webflux'
	implementation 'com.fasterxml.jackson.datatype:jackson-datatype-jsr310'

	compileOnly 'org.projectlombok:lombok'
	annotationProcessor 'org.projectlombok:lombok'

	testImplementation 'org.springframework.boot:spring-boot-starter-test'
	testImplementation 'io.projectreactor:reactor-test'

	// 의존성 추가
	asciidoctorExt 'org.springframework.restdocs:spring-restdocs-asciidoctor'
	testImplementation 'org.springframework.restdocs:spring-restdocs-mockmvc'
}

ext {
	set('snippetsDir', file("build/generated-snippets")) // 생성된 스니펫의 출력 위치
}

test {
	outputs.dir snippetsDir // 스니펫 디렉터리를 출력으로 추가
	useJUnitPlatform()
}

asciidoctor { // asciidoctor 작업을 구성
	inputs.dir snippetsDir // 스니펫 디렉터리를 입력으로 설정
	configurations 'asciidoctorExt' // asciidoctorExt 구성 사용을 구성
	baseDirFollowsSourceDir() // .adoc 파일에서 다른 .adoc 파일을 포함하는 경우 경로를 동일한 경로로 설정
	dependsOn test // 문서가 생성되기 전에 테스트가 실행되도록 설정
}

asciidoctor.doFirst {
	delete file('src/main/resources/static/docs') // asciidoctor가 실행될 때 처음으로 해당 경로에 있는 파일 제거
}

task createDocument(type: Copy) { // 복사 타입의 createDocument Task 추가
	dependsOn asciidoctor // createDocument Task가 실행되기 전에 asciidoctor Task가 먼저 수행하도록 설정
	from file("build/docs/asciidoc")
	into file("src/main/resources/static") // build/docs/asciidoc -> src/main/resources/static 복사
}

bootJar {
	dependsOn createDocument // createDocument 이후 실행되도록 설정
	from("${asciidoctor.outputDir}") { // asciidoctor.outputDir 폴더를 static/docs 폴더에 복사
		into 'static/docs'
	}
}
```

<br/>

## Test Code에 Api 정보 추가하기

이제 테스트코드에 API에 대한 정보를 추개해줘야 합니다.

```json
{
  "postId" : 12,
  "title" : "post title 12",
  "author" : "post author 12",
  "contents" : "contents 12",
  "comments" : [ {
    "postId" : 12,
    "commentId" : 1,
    "author" : "작성자1",
    "comment" : "내용1"
  }, {
    "postId" : 12,
    "commentId" : 2,
    "author" : "작성자2",
    "comment" : "내용2"
  } ]
}
```

위와 같은 응답 결과가 있을 때 테스트 코드를 다음과 같이 andDo(document())를 추가해주고 내부에 문서정보를 추가해 구현하면 됩니다.

이떄, `pathParameters`, `requestFields`, `responseFields`에 다음과 같이 데이터 정보를 추가해 각 데이터에 대한 정보를 작성할 수 있습니다.

자세한 정보는 [Documenting your API](https://godekdls.github.io/Spring%20REST%20Docs/documentingyourapi/#32-request-and-response-payloads)에서 확인하시면 됩니다.

```java
@WebMvcTest(CompositeControllerImpl.class)
@AutoConfigureRestDocs // 추가
public class CompositeControllerImplTest {

    @MockBean
    RestTemplate restTemplate;

    @Autowired
    @SuppressWarnings("SpringJavaInjectionPointsAutowiringInspection")
    MockMvc mockMvc;

    @Test
    public void getPostTest() throws Exception {
        // 생략

        mockMvc.perform(RestDocumentationRequestBuilders.get("/composite/{postId}", postId)) // 수정
                .andExpect(status().isOk())
                .andDo( // 추가
                        document("composite",
                                Preprocessors.preprocessResponse(Preprocessors.prettyPrint()), // 보기좋게 출력 
                                pathParameters(
                                        parameterWithName("postId").description("post id")
                                ),
                                responseFields(
                                        fieldWithPath("postId").type(JsonFieldType.NUMBER).description("post id"),
                                        fieldWithPath("title").type(JsonFieldType.STRING).description("post title"),
                                        fieldWithPath("author").type(JsonFieldType.STRING).description("post author"),
                                        fieldWithPath("contents").type(JsonFieldType.STRING).description("post contents"),
                                        fieldWithPath("comments").type(JsonFieldType.ARRAY).description("post comments"),
                                        fieldWithPath("comments[].postId").type(JsonFieldType.NUMBER).description("comment's post id"),
                                        fieldWithPath("comments[].commentId").type(JsonFieldType.NUMBER).description("comment id"),
                                        fieldWithPath("comments[].author").type(JsonFieldType.STRING).description("comment author"),
                                        fieldWithPath("comments[].comment").type(JsonFieldType.STRING).description("comment")
                                )

                        )
                );
    }

}

```

<br/>

### 테스트 결과가 다른 경우

이때, 테스트 결과가 다르다면 다음과같이 에러가 발생하게 됩니다. 따라서 API 최신화가 강제됩니다.

<img src='../../assets/images/Spring-cloud/docs/Test결과가-다른경우.png'>

<br/>

### 성공한 경우

테스트 결과가 같다면 다음과 같이 문제없이 테스트가 통과되는 것을 알 수 있습니다.

<img src='../../assets/images/Spring-cloud/docs/Test결과가-같은경우.png'>

<br/>

### 스니펫 결과

그리고 다음과 같이 adoc snippets가 생기는 것을 확인 가능합니다.

<img src='../../assets/images/Spring-cloud/docs/snippets-result.png'>

<br/>

## Document HTML 파일 만들기

snippets는 요청 정보에 대한 조각 파일이라고 보시면 됩니다. 따라서 해당 정보를 보여줄 페이지 양식을 지정해줘야 합니다. 

따라서 `src/docs/asciidocs`에 adoc 파일을 만들어 어떤 형태로 API 정보를 보여줄지 작성해줍니다.

### adoc 파일 작성하기

예를들어 다음과같이 `src/docs/asciidocs/index.adoc`을 다음과 같이 작성하면 아래 이미지와 같은 결과가 나타납니다.

자세한 Asciidoc 문법은 [Asciidoc 기본 사용법](https://narusas.github.io/2018/03/21/Asciidoc-basic.html)에서 확인이 가능합니다.

```
= Getting Started With Spring REST Docs

This is an example output for a service running at http://localhost:8080:

.request
include::{snippets}/composite/http-request.adoc[]

.response
include::{snippets}/composite/http-response.adoc[]

include::{snippets}/composite/response-fields.adoc[]

As you can see the format is very simple, and in fact you always get the same message.
```

<img src='../../assets/images/Spring-cloud/docs/adoc-result.png'>

<br/>

### html 파일 생성

adoc 파일을 전부 작성했다면 `./gradlew build` 명령어를 통해 다시 빌드해줍니다.

그러면 다음과 같이 `resources/static`에 html 파일이 생긴 것을 확인할 수 있습니다.

<img src='../../assets/images/Spring-cloud/docs/document-html-result.png'>

<br/>

### 결과물 확인하기

이제 `http://localhost:8000/index.html`를 통해 document를 확인할 수 있습니다.

<img src='../../assets/images/Spring-cloud/docs/restdocs-result.png'>

<br/>

# 참고 및 출처

- [Spring REST Docs를 사용한 API 문서 자동화](https://hudi.blog/spring-rest-docs/)
- [[10분 테코톡] 승팡, 케이의 REST Docs](https://www.youtube.com/watch?v=BoVpTSsTuVQ)
- [Spring REST Docs](https://docs.spring.io/spring-restdocs/docs/current/reference/htmlsingle/#documenting-your-api)
- [Restdocs로 API 문서 만들기](https://spring.io/guides/gs/testing-restdocs/)
- [[Spring] MockMVC Test](https://doongjun.tistory.com/71)
- [Spring Rest Docs 적용](https://techblog.woowahan.com/2597/)
- [Documenting your API](https://godekdls.github.io/Spring%20REST%20Docs/documentingyourapi/)
- [Asciidoc 기본 사용법](https://narusas.github.io/2018/03/21/Asciidoc-basic.html)