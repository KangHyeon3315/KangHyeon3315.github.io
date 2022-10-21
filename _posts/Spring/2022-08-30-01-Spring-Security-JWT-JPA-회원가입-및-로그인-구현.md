---
title:  "Spring Security + JWT + JPA 회원가입 및 로그인 구현" 
excerpt: ""

categories:
  - Spring
tags:
  - [JWT, Spring Security, JPA]

toc: true
toc_sticky: true
 
date: 2022-08-30
last_modified_at: 2022-08-30
---

안녕하세요 학교 연구실에서 React + Spring Boot를 이용해 웹 서비스를 만드는 프로젝트를 진행하고 있습니다.

로그인 기능을 구현하기 위해 Spring Security를 사용하려고 했는데 난이도가 어렵고, 일부 기능이 Deprecated 되어 다른 방법으로 구현해야 하는데 해당 자료를 찾는데 어려움을 겪어 복습 겸 Deprecated된 방법이 아닌 다른 방법으로 구현하는 방법을 정리해 보고자 합니다.

문제점 첫 번째로, WebSecurityConfig를 설정하는 데 WebSecurityConfigurerAdapter를 상속해야 하는데
WebSecurityConfigurerAdapter가 deprecated 되어있습니다. 따라서 Bean을 사용한 방법으로 정리했습니다.

두 번째로, 저는 React에서 Axios를 이용해 Spring 서버와 통신하기 때문에 일반적인 Session-Cookie 기반 로그인이 아닌 다중 서버 환경에 유용한 토큰 기반 로그인을 구현해야 합니다. 따라서 단순 Spring Security만을 이용한 게 아닌 JWT를 같이 이용해 구현했습니다.

기본 개념은 아래 링크에서 잘 설명되어 있어 해당 링크에서 공부하시는 것을 추천드립니다.

[https://webfirewood.tistory.com/115](https://webfirewood.tistory.com/115)

# 1. Dependency 추가

build.gradle에 다음 Dependency를 추가해 줍니다.

```properties
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
    implementation 'org.springframework.boot:spring-boot-starter-security'
    implementation 'io.jsonwebtoken:jjwt:0.9.1'
    compileOnly 'org.projectlombok:lombok'
    developmentOnly 'org.springframework.boot:spring-boot-devtools'
    runtimeOnly 'mysql:mysql-connector-java'
    annotationProcessor 'org.projectlombok:lombok'
}
```

# 2. Entity 객체와 Repository 만들기

프로젝트에 entity 패키지와, repository 패키지를 만들어 주고 아래 클래스들을 만들어줍니다.

```java
@Getter
@AllArgsConstructor
@NoArgsConstructor
@Builder
@Entity(name = "user")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private long id;

    @Column(nullable = false, length = 30, unique = true)
    private String username;

    @Column(nullable = false, length = 40)
    private String nickname;

    @Column(nullable = false, length = 100)
    private String password;

    @Column(nullable = false, length = 50)
    private String email;

    @Enumerated(EnumType.STRING)
    private Role role;
}
```

```java
public interface UserRepository extends JpaRepository<User, Long> {
    
}
```

# 3. UserDTO 추가하기

User Entity와 비슷하지만 데이터베이스의 테이블과는 무관한 DTO를 추가해 순수한 데이터 객체를 만들어줍니다.

```java
@Getter
@Setter
@NoArgsConstructor
public class UserDTO {
    private String username;
    private String nickname;
    private String password;
    private String email;
    private Role role;

    @Builder
    public UserDTO(String username, String nickname, String password, String email, Role role) {
        this.username = username;
        this.nickname = nickname;
        this.password = password;
        this.email = email;
        this.role = role;
    }

    public User toEntity() {
        return User.builder()
                .username(username)
                .nickname(nickname)
                .password(password)
                .email(email)
                .role(role)
                .build();
    }
}
```

# 4. WebSecurityConfig 추가하기

config 패키지를 추가로 만들어 아래 클래스를 추가해 줍니다.

해당 클래스에서 어떤 경로의 요청에 인증을 요청할지 설정이 가능합니다.

```java
@Configuration
@EnableWebSecurity
@RequiredArgsConstructor
public class WebSecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
                .authorizeRequests((requests) -> requests
                        .antMatchers( "/auth/signup", "/auth/login").permitAll()
                        .anyRequest().authenticated()
                );

        return http.build();
    }

}
```

# 5. JwtTokenProvider 추가하기

이번에는 Jwt를 생성하고 검증하는 클래스를 추가하겠습니다.

```java
@Component
@RequiredArgsConstructor
public class JwtTokenProvider {
    private String secretKey = "Secret Key";

    private final long tokenValidTime = 30 * 60 * 1000L;     // 토큰 유효시간 30분

    private final UserDetailsService userDetailsService;

    // 객체 초기화, secretKey를 Base64로 인코딩한다.
    @PostConstruct
    protected void init() {
        secretKey = Base64.getEncoder().encodeToString(secretKey.getBytes());
    }

    // JWT 토큰 생성
    public String createToken(String userPk, List<String> roles) {
        Claims claims = Jwts.claims().setSubject(userPk); // JWT payload 에 저장되는 정보단위
        claims.put("roles", roles); // 정보는 key / value 쌍으로 저장된다.
        Date now = new Date();
        return Jwts.builder()
                .setClaims(claims) // 정보 저장
                .setIssuedAt(now) // 토큰 발행 시간 정보
                .setExpiration(new Date(now.getTime() + tokenValidTime)) // set Expire Time
                .signWith(SignatureAlgorithm.HS256, secretKey)  // 사용할 암호화 알고리즘과
                // signature 에 들어갈 secret값 세팅
                .compact();
    }

    // JWT 토큰에서 인증 정보 조회
    public Authentication getAuthentication(String token) {
        UserDetails userDetails = userDetailsService.loadUserByUsername(this.getUserPk(token));
        return new UsernamePasswordAuthenticationToken(userDetails, "", userDetails.getAuthorities());
    }

    // 토큰에서 회원 정보 추출
    public String getUserPk(String token) {
        return Jwts.parser().setSigningKey(secretKey).parseClaimsJws(token).getBody().getSubject();
    }

    // Request의 Header에서 token 값을 가져옵니다. "X-AUTH-TOKEN" : "TOKEN값'
    public String resolveToken(HttpServletRequest request) {
        return request.getHeader("X-AUTH-TOKEN");
    }

    // 토큰의 유효성 + 만료일자 확인
    public boolean validateToken(String jwtToken) {
        try {
            Jws<Claims> claims = Jwts.parser().setSigningKey(secretKey).parseClaimsJws(jwtToken);
            return !claims.getBody().getExpiration().before(new Date());
        } catch (Exception e) {
            return false;
        }
    }
}
```

# 6. JwtAuthenticationFilter 추가하기

JwtAuthenticationFilter는 인증 작업을 진행하는 클래스입니다.

```java
@RequiredArgsConstructor
public class JwtAuthenticationFilter extends GenericFilterBean {
    private final JwtTokenProvider jwtTokenProvider;

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        // 헤더에서 JWT 를 받아옵니다.
        String token = jwtTokenProvider.resolveToken((HttpServletRequest) request);
        // 유효한 토큰인지 확인합니다.
        if (token != null && jwtTokenProvider.validateToken(token)) {
            // 토큰이 유효하면 토큰으로부터 유저 정보를 받아옵니다.
            Authentication authentication = jwtTokenProvider.getAuthentication(token);
            // SecurityContext 에 Authentication 객체를 저장합니다.
            SecurityContextHolder.getContext().setAuthentication(authentication);
        }
        chain.doFilter(request, response);
    }
}
```

# 7. WebSecurityConfig에 JwtAuthenticationFilter 추가하기

```java
@Configuration
@EnableWebSecurity
@RequiredArgsConstructor
public class WebSecurityConfig {
    private final JwtTokenProvider jwtTokenProvider;

    // 암호화에 필요한 PasswordEncoder 를 Bean 등록합니다.
    @Bean
    public PasswordEncoder passwordEncoder() {
        return PasswordEncoderFactories.createDelegatingPasswordEncoder();
    }

    // authenticationManager를 Bean 등록합니다.
    @Bean
    public AuthenticationManager authenticationManager(AuthenticationConfiguration authenticationConfiguration)
            throws Exception {
        return authenticationConfiguration.getAuthenticationManager();
    }

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
                .httpBasic().disable() // rest api 만을 고려해 기본 설정 해제
                .csrf().disable() // csrf 보안 토큰 disable
                .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS) // 토큰 기반 인증이므로 세션 사용 X
                .and()
                .authorizeRequests((requests) -> requests
                        .antMatchers( "/api/auth/signup", "/api/auth/signin").permitAll() // 모든 사용자 접근 가능
                        .anyRequest().authenticated() //다른 요청은 모두 권한 체크
                        .and()
                         // JwtAuthenticationFilter를 UsernamePasswordAuthenticationFilter 전에 넣는다
                        .addFilterBefore(new JwtAuthenticationFilter(jwtTokenProvider), UsernamePasswordAuthenticationFilter.class)
                );
        return http.build();
    }

}
```

# 8. CustomUserDetatilService 클래스 추가하기

```java
@RequiredArgsConstructor
@Service
public class CustomUserDetailService implements UserDetailsService {

    private final UserRepository userRepository;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        return userRepository.findByUsername(username)
                .orElseThrow(() -> new UsernameNotFoundException("사용자를 찾을 수 없습니다."));
    }
}
```

# 9. UserRepository에 Query 추가하기

이번에 username을 이용해 해당 유저 정보를 찾는 Query를 추가해 줍니다.

```java
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByUsername(String username);
}
```

# 10. User 클래스에 UserDetails 상속받아 권한 정보 관리 기능 추가하기

```java
@Getter
@AllArgsConstructor
@NoArgsConstructor
@Builder
@EqualsAndHashCode(of = "id")
@Entity(name = "user")
public class User implements UserDetails {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private long id;

    @Column(nullable = false, length = 30, unique = true)
    private String username;

    @Column(nullable = false, length = 40)
    private String nickname;

    @Column(nullable = false, length = 100)
    private String password;

    @Column(nullable = false, length = 50)
    private String email;

    @Enumerated(EnumType.STRING)
    private Role role;

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        List<GrantedAuthority> authorities = new ArrayList<>();
        authorities.add(new SimpleGrantedAuthority(role.getKey()));
        return authorities;

    }

    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled() {
        return true;
    }
}
```

# 11. 실제 로그인 및 회원가입 기능을 Controller에 구현하기

```java
@RestController
@RequestMapping("/auth")
@RequiredArgsConstructor
public class AuthController {
    private final PasswordEncoder passwordEncoder;
    private final JwtTokenProvider jwtTokenProvider;
    private final UserRepository userRepository;

    // 회원가입
    @PostMapping("/signup")
    public Long join(UserDTO userDTO) {
        userDTO.setPassword(passwordEncoder.encode(userDTO.getPassword()));
        return userRepository.save(userDTO.toEntity()).getId();
    }

    // 로그인
    @PostMapping("/login")
    public String login(String username, String password) {
        User user = userRepository.findByUsername(username)
                .orElseThrow(() -> new IllegalArgumentException("가입되지 않은 계정 입니다."));;

        if (!passwordEncoder.matches(password, user.getPassword())) {
            throw new IllegalArgumentException("잘못된 비밀번호입니다.");
        }

        List<String> roles = new ArrayList<>();
        roles.add(user.getRole().getKey());
        return jwtTokenProvider.createToken(user.getUsername(), roles);
    }
}
```

이제 Spring 프로젝트를 실행하고, /auth/login으로 로그인할 경우 성공하면 토큰을 반환하고, 실패하면 500 에러가 발생합니다.

성공하고 수신 받은 토큰을 header에 X-AUTH-TOKEN에 담아 요청하면 토큰을 통해 권한을 확인하고 리소스를 반환합니다.

# 출처

[SPRING SECURITY + JWT 회원가입, 로그인 기능 구현](https://webfirewood.tistory.com/115)

[[Spring] Security WebSecurityConfigurerAdapter Deprecated 해결하기](https://devlog-wjdrbs96.tistory.com/434)
