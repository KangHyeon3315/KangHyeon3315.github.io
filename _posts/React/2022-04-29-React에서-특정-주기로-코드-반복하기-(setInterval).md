---
title:  "React에서 특정 주기로 코드 반복하기 (setInterval)" 
excerpt: ""

categories:
  - React
tags:
  - []

toc: true
toc_sticky: true
 
date: 2022-04-29
last_modified_at: 2022-04-29
---

```javascript
class MyComponent extends Component {
    componentDidMount() {
        const interverId = setInterval(() => {
            // 반복할 코드 작성
        }, 1000)  // ms 단위
    }
    ...
}
```