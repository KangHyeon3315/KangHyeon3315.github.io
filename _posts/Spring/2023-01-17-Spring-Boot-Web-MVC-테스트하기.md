---
title:  "Spring Boot Web MVC 테스트하기"
excerpt: ""

categories:

- Spring
tags:
- [TDD]

toc: true
toc_sticky: true

date: 2023-01-17
last_modified_at: 2023-01-17
---

# Spring Web MVC Test하기
```java
@SpringBootTest
@AutoConfigureMockMvc
class MyControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    void myText() throws Exception {
        MyDTO sample = MyDTO.sample();
        String contents = objectMapper.writeValueAsString(sample);

        mockMvc.perform(post("/users") // post 요청
            .content(contents) // parameter
            .contentType(MediaType.APPLICATION_JSON) // contents type
            .accept(MediaType.APPLICATION_JSON)) // accept type
            .andExpect(status().isOk()) // status 검사
            .andExpect(content().string(sample.getId())) // 반환 결과가 sample의 id인지 검사
            .andDo(print()); // console에 출력
            
    }

}
```

