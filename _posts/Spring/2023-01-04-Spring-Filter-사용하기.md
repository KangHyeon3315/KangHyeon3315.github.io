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

# Spring Filter란?
필터(Filter)는 요청이 전달되기 전/후에 url 패턴에 맞는 모든 요청에 대해 부가적인 작업을 처리할 수 있는 기능을 제공합니다.

# Filter 구현하기
구현하는 방법은 아래 코드와 같이 Filter 인터페이스를 구현하는 방식으로 구현합니다. 해당 필터에서 부가적인 작업을 처리합니다.


```java
// MyFilter.java
public class MyFilter implements Filter {
  
  @Override
  public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
    System.out.println("필터 실행됨");

    HttpServletResponse servletResponse = (HttpServletResponse) response;
    servletResponse.setContentType("text/plain; charset=utf-8");

    PrintWriter out = servletResponse.getWriter();
    out.print("응답" + i);
    out.flush();
  }
} 
```

# Filter 등록하기
이제 구현한 Filter를 다음과 같이 @Configuration Annotation을 추가한 Config 클래스에서 filter를 등록합니다.

```java
// MyFilterConfig.java

@Configuration
public class MyFilterConfig {

  public FilterRegistrationBean<Filter> addFilter() {
    System.out.println("필터 등록됨")
    FilterRegistrationBean<Filter> bean = new FilterRegistrationBean<>(new MyFilter(eventNotify));
    bean.addUrlPatterns("/*");

    return bean;
  }
}
```
