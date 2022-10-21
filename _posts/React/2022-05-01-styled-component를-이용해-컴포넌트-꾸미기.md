---
title:  "styled-component를 이용해 컴포넌트 꾸미기" 
excerpt: ""

categories:
  - React
tags:
  - []

toc: true
toc_sticky: true
 
date: 2022-05-01
last_modified_at: 2022-05-01

---

저는 기존에 React 프로젝트를 하면서 컴포넌트로 꾸밀 때는 CSS 파일을 만들어 CSS를 정의하고 import 하는 방식으로 꾸며왔습니다. 이 경우 동적으로 변경하는 것이 불편한 경우가 있어 styled component라는 라이브러리를 사용해 컴포넌트를 꾸미는 방법을 정리해 보겠습니다.

우선 패키지 설치 명령어는 다음과 같습니다.

```text
npm install styled-components
```

# 기본 문법

1. html 엘리먼트 스타일링 하기

```javascript
import styled from "styled-components";

const StyledButton = styled.button`
    해당 버튼에 대한 css 내용
`;
```

2. React 컴포넌트 스타일링 하기

```javascript
import styled from "styled-components";
import MyComponent from "./MyComponent";

const StyledMyComponent = styled(MyComponent)`
    MyComponent에 대한 css 내용
`;
```

# 가변 스타일링

스타일이 적용된 컴포넌트도 컴포넌트이기 때문에 다음과 같이 props를 통해 값 전달이 가능합니다. 이 경우 그 아래 코드와 같이 props를 이용해 css 값을 동적으로 적용 가능합니다.

```javascript
import styled from "styled-components";

const StyledButton = styled.button`
    width: ${(props)=>props.width || "100px"};
`;
```

```javascript
<StyledButton width="200px"> MyButton </StyledButton>
```