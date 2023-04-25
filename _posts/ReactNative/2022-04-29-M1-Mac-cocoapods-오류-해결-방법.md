---
title:  "M1 Mac cocoapods 오류 해결 방법" 
excerpt: ""

categories:
  - ReactNative
tags:
  - [M1 Mac]

toc: false
toc_sticky: false
 
date: 2022-04-29
last_modified_at: 2022-04-29
---

최근 M1 맥에서 다시 React Native를 이용한 프로젝트를 진행해 보려고 하니 아래 사진과 같이 오류가 발생합니다.
![](../../assets/images/reactnative/M1MacCocoapods/80704.png)

오류 내용을 차분히 읽어보면 CocoaPods의 의존성 때문에 생성 실패를 하는 상황입니다.

그래서 오류에서 말하는 대로 pod repo update를 실행해 보면 다음과 같이 오류가 발생합니다

![](../../assets/images/reactnative/M1MacCocoapods/91229.png)

이유는 M1의 arm 아키텍처에서는 pod repo update가 지원되지 않습니다.

그래서 해결 방법을 검색해 보니 공식 문서는 다음과 같이 x86_64로 명령어를 실행하면 됩니다.

![](../../assets/images/reactnative/M1MacCocoapods/React_Native_docs.png)

그래서 한번 명령어를 따라 해보면 다음과 같이 오류가 발생합니다.

```text
cd 생성중인 프로젝트/ios
sudo arch -x86_64 gem install ffi
arch -x86_64 pod install
```

![](../../assets/images/reactnative/M1MacCocoapods/91859.png)

다시 pod repo update를 실행하라고 합니다. 그래서 몇 시간 동안 구글링 해봤지만 해결 방법을 찾지 못했었습니다.

결국 마지막에 해결했는데 해결 방법은 pod repo update 또한 x86_64로 실행하는 방법입니다.

```text
cd 생성중인 프로젝트/ios
sudo arch -x86_64 gem install ffi
arch -x86_64 pod repo update
arch -x86_64 pod install
```

![](../../assets/images/reactnative/M1MacCocoapods/92037.png)

이제 정상적으로 CocoaPods에 대한 의존성 문제를 해결했습니다. 다시 프로젝트 생성을 진행하면 됩니다.

다시 프로젝트 부모 디렉터리로 이동해서 프로젝트 생성하면 정상적으로 생성이 됩니다.

![](../../assets/images/reactnative/M1MacCocoapods/ProjectGenerate.png)








