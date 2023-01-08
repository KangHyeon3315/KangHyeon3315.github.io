---
title:  "java에서 selenium 사용하기"
excerpt: ""

categories:

- Java

tags:

- [gradle, selenium, parsing]

toc: true
toc_sticky: true

date: 2023-01-08
last_modified_at: 2023-01-08
---

# build.gradle에 selenium 추가하기
```java
implementation 'org.seleniumhq.selenium:selenium-java:4.7.1'
```

# chrome webdriver 설치하기
selenium을 이용하기 위해서는 브라우저의 webdriver를 설치해야 합니다.

## browser에서 다운로드 받기

부라우저에서 직접 다운로드하는 경우 여기([chromedriver 설치](https://chromedriver.chromium.org/downloads))에서 설치 가능합니다.

## brew에서 설치하기

터미널에서 다음 명령어를 이용해 설치가 가능합니다. 다운로드가 왼료되면 '/usr/local/bin/chromedriver'에 webdriver가 위치홥니다.

```text
brew install --cask chromedriver
```

# 크롤링하기

## Browser 열기

```java
System.setProperty("webdriver.chrome.driver", DRIVER_PATH);
    
WebDriver driver = new ChromeDriver();
driver.get(url); // 해당 url로 이동한다.
```

## 로딩 기다리기

```java

// Thread.sleep로 기다리기
Thread.sleep(1000);

// ImplicitlyWait으로 페이지가 도딩될때까지 기다린다. 최대 주어진 시간만큼 기다린다.
driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10));

// ExplicitlyWait으로 일부분이 나타날때까지 기다린다. 최대 주어진 시간만큼 기다린다.
WebDriverWait driverWait = new WebDriverWait(driver, Duration.ofSeconds(5));
driverWait.until(ExpectedConditions.presenceOfElementLocated(By.cssSelector("찾으려는 element")));

```

## elements 찾기

```java

// 여러 Element
List<WebElement> elements = driver.findElements(By.cssSelector("찾으려는 element"));
 
// 단일 Element
WebElement element = driver.findElement(By.cssSelector("찾으려는 element"));
 
```

## element 값 가져오기 및 조작하기

```java

// text 가져오기
String text = element.getText();

// attribute 가져오기 
String attr = element.getAttribute("이름");

// 클릭하기
element.click()

```

## browser 종료하기

```java
driver.close(); // 탭 닫기
driver.quit(); // 브라우저 닫기
```