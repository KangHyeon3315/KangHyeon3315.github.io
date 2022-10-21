---
title:  "Please accept all necessary Android SDK licenses 문제 해결하기" 
excerpt: ""

categories:
  - ReactNative
tags:
  - []

toc: false
toc_sticky: false
 
date: 2021-12-23
last_modified_at: 2021-12-23
---

React Native를 설치하다 보니 Android SDK licenses 때문에 안드로이드 실행이 안 되는 문제가 있었습니다.

해당 문제를 구글링해 보니  Android SDK licenses를 accept 하면 바로 해결되는 문제입니다.

윈도우에서는 cmd 창에 다음과 같이 입력하면 되고

```text
%ANDROID_HOME%\tools\bin\sdkmanager.bat --licenses
```

mac에서는 terminal에 다음과 같이 입력하면 됩니다.

```text
yes | $ANDROID_HOME/tools/bin/sdkmanager --licenses
```

<br/>
출처 : 
[https://www.kindacode.com/article/react-native-please-accept-all-necessary-android-sdk-licenses/](https://www.kindacode.com/article/react-native-please-accept-all-necessary-android-sdk-licenses/)