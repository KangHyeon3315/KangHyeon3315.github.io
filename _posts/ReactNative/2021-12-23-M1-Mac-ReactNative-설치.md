---
title:  "mac os에서 React Native 개발 환경 구축하기" 
excerpt: ""

categories:
  - ReactNative
tags:
  - [M1 Mac]

toc: true
toc_sticky: true
 
date: 2021-12-23
last_modified_at: 2021-12-23
---

Mac에서 React Native를 이용하기 위해서는 2가지 방법이 있습니다. 첫 번째로 Expo CLI를 이용하는 방법이고, 두 번째로 React Native CLI가 있습니다.


Expo CLI의 경우 자주 상용되는 네이티브 기능을 패키지로 묶어 제공하는데 이로 인해 사용하지 않는 모듈까지 있어 앱 사이즈가 커지는 문제가 있어 주로  React Native CLI를 이용한다고 합니다. 그래서 저도  React Native CLIf를 이용해 개발해 보겠습니다.


우선 React Native를 설치하기 위해서는 Nodejs, Watchman, Xcode 등을 설치해야 합니다.

# 1. Node js 설치

homebrew를 이용해 nodejs를 설치하고 node와 npm이 정상적으로 설치가 되었는지 확인해 보면 됩니다.

```text
brew install node

node --version
npm --version
```

# 2. Watchman 설치

Watchman은 특정 폴더나 파일을 감시하다가 변화가 생기면 특정 동작을 실행하도록 하는 역할을 하는 툴입니다. 이번 프로젝트에서는 소스코드를 변경했을 때 자동으로 다시 빌드 하는 기능을 수행하기 위해 Watchman을 사용하도록 하겠습니다.

```text
brew install watchman
watchman --version
```

3. Xcode 설치

React Native로 IOS 앱을 개발하기 위해서는 xcode가 반드시 필요합니다. 따라서 앱스토어에서 xcode를 검색해 다운을 받습니다.

그다음 Preferences - Locations - Command Line Tools 에서 Xcode를 선택해줍니다.

![](../../assets/images/reactnative/MacOS-ReactNativeInstall/스크린샷_2021-12-23_오후_10.10.07.png)

![](../../assets/images/reactnative/MacOS-ReactNativeInstall/스크린샷_2021-12-23_오후_10.10.34.png)

그리고 cocoapods이라는 의존성 관리자를 추가적으로 설치해 줍니다.

```text
sudo gem install cocoapods
pod --version
```

# 4. React Native CLI 설치

이제 react native cli를 설치하고 잘 설치되었는지 버전을 확인해 보겠습니다.

```text
npm install -g react-native-cli
npx react-native --version
```

# 5. JDK 설치

이제 안드로이드 앱을 개발하기 위한 JDK를 설치하고 버전을 확인해 보겠습니다.

```text
brew install --cask adoptopenjdk8

java -version
Javac -version
```

# 6. 안드로이드 스튜디오 설치

이제 추가적으로 안드로이드 스튜디오를 다음 링크에서 설치하면 됩니다.

[Android Studio 설치](https://developer.android.com/studio)

안드로이드 스튜디오를 설치한 뒤에는 환경 변수를 설정해야합니다.

터미널 종류에 따라 .bash_profile이나 .zshrc에 다음과 같이 정보를 추가해줍니다.

```text
export ANDROID_HOME=자신의 안드로이드SDK 위치/Android/sdk
export PATH=$PATH:$ANDROID_HOME/emulator
export PATH=$PATH:$ANDROID_HOME/tools
export PATH=$PATH:$ANDROID_HOME/tools/bin
export PATH=$PATH:$ANDROID_HOME/platform-tools
```

이후 다음 터미널 종류에 따라 다음 명령어를 실행해줍니다.

```text
# bash
source ~/.bash_profile
# zsh
source ~/.zshrc
```

만약 정상적으로 설치되면 다음 명령어를 실행했을때 다음 사진과 같이 버전 정보가 출력됩니다.

```text
adb --version
```

![](../../assets/images/reactnative/MacOS-ReactNativeInstall/스크린샷_2021-12-23_오후_9.59.00.png)

# 7. 프로젝트 생성

이제 모든 설치가 끝났습니다. 이제 다음 명령어를 이용해 프로젝트를 생성하기만 하면 됩니다.

```text
npx react-native init [프로젝트 이름]
```

그다음 IOS와 안드로이드 에뮬레이터를 실행해 어플리케이션을 실행 할 수 있습니다.

```text
cd [프로젝트 이름]

npm run android
npm run ios
```

|                                               IOS                                                | Android |
|:------------------------------------------------------------------------------------------------:| :---: |
| <img src="../../assets/images/reactnative/MacOS-ReactNativeInstall/스크린샷_2021-12-23_오후_10.22.51.png" alt=""/> | <img src="../../assets/images/reactnative/MacOS-ReactNativeInstall/스크린샷_2021-12-23_오후_10.45.25.png" alt="" /> |
