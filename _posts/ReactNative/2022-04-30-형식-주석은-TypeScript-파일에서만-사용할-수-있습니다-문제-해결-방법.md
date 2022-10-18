---
title:  "형식 주석은 TypeScript 파일에서만 사용할 수 있습니다. 문제 해결 방법" 
excerpt: ""

categories:
  - ReactNative
tags:
  - []

toc: false
toc_sticky: false
 
date: 2022-04-30
last_modified_at: 2022-04-30
---

React Native 프로젝트를 생성하면 app.js에서 바로 빨간 줄이 나타나는 경우가 있습니다.

그래서 자세한 정보를 확인하면 다음과 같이 형식 주석은 TypeScript 파일에서만 사용이 가능하는 문장을 표시합니다.

![](../../assets/images/형식주석은-TypeScript-파일에서만-사용할-수-있습니다/스크린샷_2022-04-30_오후_9.59.35.png)

해결 방법은 간단합니다.

다음과 같이 setting.json을 오픈하고 아래 코드를 추가해 주고 저장하면 해결됩니다.

![](../../assets/images/형식주석은-TypeScript-파일에서만-사용할-수-있습니다/스크린샷_2022-04-30_오후_9.59.51.png)

    "javascript.validate.enable" : false

![](../../assets/images/형식주석은-TypeScript-파일에서만-사용할-수-있습니다/스크린샷_2022-04-30_오후_10.00.04.png)

![](../../assets/images/형식주석은-TypeScript-파일에서만-사용할-수-있습니다/스크린샷_2022-04-30_오후_10.00.14.png)