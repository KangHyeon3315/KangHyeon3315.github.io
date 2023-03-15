---
title:  "Spring OAuth2.0 구현하기"
excerpt: ""

categories:

- Spring
tags:
- [Security, OAuth2.0]

toc: true
toc_sticky: true

date: 2023-01-17
last_modified_at: 2023-01-17
---

# OAuth 2.0이란?

OAuth 2.0은 구글, 페이스북, 깃헙, 카카오 등 다양한 기업의 로그인을 내 어플리케이션에 붙여 사용자의 정보를 가져올 수 있는 기능입니다.

이 기능을 사용하는 이유는 직접 구연해야하는 경우 구현해야 하는 기능도 많고 보안또한 신경써야 하기 때문입니다.

# 1. 구글 서비스 등록하기

우선 [Google Cloud](https://console.cloud.google.com/apis/dashboard?pli=1&project=teammatch)에 접속해 프로젝트를 등록해야 합니다.

<img src='../../assets/images/spring/oauth/project_select.png'>

<img src='../../assets/images/spring/oauth/new_project.png'>

그리고 `API 및 서비스`를 선택해 OAuth 동의 화면으로 넘어가 아래 사진과 같이 입력합니다.

<img src='../../assets/images/spring/oauth/OAuth-동의화면1.png'>

<img src='../../assets/images/spring/oauth/OAuth-동의화면2.png'>

`범위 추가 또는 삭제`를 클릭해 어떤 데이터에 액세스할지 범위를 설정합니다.

<img src='../../assets/images/spring/oauth/select-data1.png'>

<img src='../../assets/images/spring/oauth/select-data2.png'>

그 다음 사용자 인증 정보를 만들기 위해 `사용자 인증 정보` > `사용자 인증 정보 만들기` > `OAuth 클라이언트 ID`를 선택합니다.

<img src='../../assets/images/spring/oauth/OAuth-client-id1.png'>

애플리케이션 유형, 이름, 리다이렉션 URI를 설정해줍니다.

<img src='../../assets/images/spring/oauth/OAuth-client-id2.png'>

그러면 다음과 같이 클라이언트 ID, 클라이언트 비밀번호를 얻을 수 있습니다.

<img src='../../assets/images/spring/oauth/client-id-generated.png'>

이제 해당 값들을 이용해 인증하는 기능을 구현할 수 있습니다.

# 2. 인증 기능 구현하기

## 의존성 추가하기

```gradle
implementation 'org.springframework.boot:spring-boot-starter-web'

// OAuth
implementation 'org.springframework.boot:spring-boot-starter-security'
implementation 'org.springframework.boot:spring-boot-starter-oauth2-client'

// Database
implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
runtimeOnly 'mysql:mysql-connector-java'
```

## properties 추가하기

### application-oauth.properties 작성하기
```properties

spring.security.oauth2.client.registration.google.client-id=클라이언트id
spring.security.oauth2.client.registration.google.client-secret=클라이언트pw
spring.security.oauth2.client.registration.google.scope=profile,email
# scope의 기본값은 email, profile, openid
# 이 경우 네이버나 카카오와 같은 서비스에서 제공하는 OAuth의 경우 openID를 제공하기 않기 때문에 위와 같이 설정해 동일하게 적용 가능하도록 구현

```

### application-oauth.properties를 application.properties에 추가하기

```properties
spring.profiles.include=oauth
```

### gitignore에 application-oauth.properties 추가
```text
application-oauth.properties
```

## Role 만들기

우선 User의 역할 정보를 가지는 Enum을 만들어줍니다.

```java
@Getter
@RequiredArgsConstructor
public enum Role {
    USER("ROLE_USER"),
    ADMIN("ROLE_ADMIN");

    private final String value;
}
```

## Time Entity 만들기

그리고 해당 유저의 생성 시간과 수정 시간을 관리하는 Entity를 만들어줍니다.

```java
@Getter
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public abstract class TimeEntity {
 
    @Column(name = "created_date")
    @CreatedDate
    private String createdDate;
 
    @Column(name = "modified_date")
    @LastModifiedDate
    private String modifiedDate;
 
    /* 해당 엔티티를 저장하기 이전에 실행 */
    @PrePersist
    public void onPrePersist(){
        this.createdDate = LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy.MM.dd"));
        this.modifiedDate = this.createdDate;
    }
 
    /* 해당 엔티티를 업데이트 하기 이전에 실행*/
    @PreUpdate
    public void onPreUpdate(){
        this.modifiedDate = LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy.MM.dd HH:mm"));
    }
}
```

## User Entity 만들기

Time Entity를 상속받아 유저 정보를 가지고 있는 Entity를 만들어줍니다.

```java
@Builder
@Getter
@Entity
@AllArgsConstructor
@NoArgsConstructor
public class User extends TimeEntity {
 
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
 
    @Column(nullable = false, length = 30, unique = true)
    private String username;
 
    @Column(nullable = false, unique = true)
    private String nickname;
 
    @Column(length = 100)
    private String password;
 
    @Column(nullable = false, length = 50)
    private String email;
 
    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private Role role;
 
    public void modify(String nickname, String password) {
        this.nickname = nickname;
        this.password = password;
    }
 
    public User updateModifiedDate() {
        onPreUpdate();
        return this;
    }
 
    public String getRoleValue() {
        return this.role.getValue();
    }
}

```

## User Repository 만들기


```java

public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByUsername(String username);
}

```

## OAuthAttributes
OAuth2UserService를 통해 가져온 OAuth2User의 attribute를 담을 클래스를 만들어줍니다.

```java
@AllArgsConstructor
@NoArgsConstructor
@Builder
@Getter
public class OAuthAttributes {
    private Map<String, Object> attributes;
    private String nameAttributeKey;
    private String username;
    private String nickname;
    private String email;
    private Role role;
    
    public static OAuthAttributes of(String registrationId,
                                     String userNameAttributeName,
                                     Map<String, Object> attributes) {
         
         /* OAuth 서비스(구글 (ofGoogle)), 네이버(ofNaver), 카카오(ofKakao)) 구분하기 위한 메소드 */
        return ofGoogle(userNameAttributeName, attributes); 
    }
 
    private static OAuthAttributes ofGoogle(String userNameAttributeName,
                                            Map<String, Object> attributes) {
        return OAuthAttributes.builder()
                .username((String) attributes.get("email"))
                .email((String) attributes.get("email"))
                .nickname((String) attributes.get("name"))
                .attributes(attributes)
                .nameAttributeKey(userNameAttributeName)
                .build();
    }
    
    public User toEntity() {
        return User.builder()
                .username(email)
                .email(email)
                .nickname(nickname)
                .role(Role.SOCIAL)
                .build();
    }
}
```

## SessionUser

세션에 사용자 정보를 저장하기 위한 Dto 클래스를 만들어줍니다.

```java
@Getter
public class SessionUser implements Serializable {
    private String name;
    private String email;

    public SessionUser(User user) {
        this.name = user.getNickname();
        this.email = user.getEmail();
    }
}

```


## CustomOAuth2UserService

OAuth2의 처리 기능을 구현할 CustomOAuth2UserService를 만들어줍니다.

```java
@RequiredArgsConstructor
@Service
public class CustomOAuth2UserService implements OAuth2UserService<OAuth2UserRequest, OAuth2User> {

    private final UserRepository userRepository;
    private final HttpSession session;

    @Override
    public OAuth2User loadUser(OAuth2UserRequest userRequest) throws OAuth2AuthenticationException {
        OAuth2UserService<OAuth2UserRequest, OAuth2User> delegate = new DefaultOAuth2UserService();
        OAuth2User oAuth2User = delegate.loadUser(userRequest);

        /* 서비스 id 구분코드 ( 구글, 카카오, 네이버 등 ) */
        String registrationId = userRequest.getClientRegistration().getRegistrationId();

        /* OAuth2 로그인 진행 키값 */
        String userNameAttributeName = userRequest.getClientRegistration().getProviderDetails()
                .getUserInfoEndpoint().getUserNameAttributeName();

        /* OAuth2UserService */
        OAuthAttributes attributes = OAuthAttributes.of(registrationId, userNameAttributeName, oAuth2User.getAttributes());

        User user = saveOrUpdate(attributes);

        /* 세션 정보를 저장하는 직렬화된 dto 클래스*/
        session.setAttribute("user", new SessionUser(user));

        return new DefaultOAuth2User(
                Collections.singleton(new SimpleGrantedAuthority(user.getRoleValue())),
                attributes.getAttributes(),
                attributes.getNameAttributeKey());
    }

    /* 기존 회원이 존재하면 수정날짜 정보만 업데이트 */
    private User saveOrUpdate(OAuthAttributes attributes) {
        User user = userRepository.findByUsername(attributes.getEmail())
                .map(User::updateModifiedDate)
                .orElse(attributes.toEntity());

        return userRepository.save(user);
    }
}
```

## SecurityConfig 설정하기

WebSecurityConfigurerAdapter는 버전이 업그레이되면서 Deprecated 되었습니다. 따라서 아래 코드와 같이 filterChain을 이용해 Configuration을 설정합니다.

```java
@RequiredArgsConstructor
@Configuration
@EnableWebSecurity
public class SecurityConfig{

    private final CustomOAuth2UserService customOAuth2UserService;

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
                .csrf().disable()
                .headers().frameOptions().disable()
                .and()
                .authorizeRequests()
                .antMatchers("/", "/css/**", "/images/spring/**", "/js/**").permitAll()
                .antMatchers("/api/**").hasRole(Role.USER.name())
                .anyRequest().authenticated()
                .and()
                .logout()
                .logoutSuccessUrl("/")
                .and()
                .oauth2Login()
                .userInfoEndpoint()
                .userService(customOAuth2UserService);
        return http.build();
    }
}

```

# 프론트엔드 구현하기

테스트용으로 간단하게 구현하기 위해 Mustache로 간단하게 로그인 기능만 구현해보겠습니다.

## 의존성 추가하기

```gradle
compileOnly 'org.springframework.boot:spring-boot-starter-mustache'
```

<br/>

## Web Page 구현하기 (Mustache)

{%raw%}
```html
<!DOCTYPE HTML>
<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />

        <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.6.0/font/bootstrap-icons.css">
        <link rel="stylesheet" href="/css/app.css">
        <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css">
    </head>
    <body>
        {{#user}}
            <span class="mx-3">Hello {{user}}</span>
            <a href="/logout" class="btn btn-outline-dark">Logout</a>
            <a href="/modify" class="btn btn-outline-dark bi bi-gear"></a>
        {{/user}}
        {{^user}}
            <a href="/oauth2/authorization/google" role="button" class="btn btn-outline-danger bi bi-google"> Google Login</a>
        {{/user}}
    </body>
</html>
```
{%endraw%}

## Controller 구현하기

```java
@Controller
@RequiredArgsConstructor
public class IndexController {

    private final HttpSession httpSession;

    @GetMapping("/")
    public String index(Model model) {
        SessionUser user = (SessionUser)httpSession.getAttribute("user");

        if(user != null) {
            model.addAttribute("user", user.getName());
        }

        return "index";
    }
}

```

# 결과

이제 [http://localhost:8080](http://localhost:8080)에 접속하면 다음과 같이 Google Login 버튼이 나타나고 클릭합니다.

<img src='../../assets/images/spring/oauth/LoginBtn.png'>

클릭시 구글 로그인 화면으로 이동하고 로그인시 아래 화면처럼 로그인 성공 화면으로 이동합니다.

<img src='../../assets/images/spring/oauth/LoginedBtn.png'>

<br/>

# 참고 및 출처

- [Spring Boot 게시판 OAuth 2.0 구글 로그인 구현](https://dev-coco.tistory.com/128#--%--Mustache)
- [Spring Security와 OAuth2](https://velog.io/@shawnhansh/Spring-Security%EC%99%80-OAuth2-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0)
- [스프링 시큐리티와 OAuth 2.0으로 로그인 기능 구현](https://velog.io/@swchoi0329/%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0%EC%99%80-OAuth-2.0%EC%9C%BC%EB%A1%9C-%EB%A1%9C%EA%B7%B8%EC%9D%B8-%EA%B8%B0%EB%8A%A5-%EA%B5%AC%ED%98%84)
- [[SpringBoot] OAuth2 Google Login](https://seokr.tistory.com/811)
- [WebSecurityConfigurerAdapter deprecated 관련 이슈 및 권장방식 적용](https://blog.naver.com/PostView.naver?blogId=h850415&logNo=222755455272&parentCategoryNo=&categoryNo=37&viewDate=&isShowPopularPosts=true&from=search)
