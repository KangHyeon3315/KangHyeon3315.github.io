---
title:  "문자열 수학 표현식을 계산하는 법" 
excerpt: ""

categories:
  - Spring
tags:
  - []

toc: true
toc_sticky: true
 
date: 2022-05-20
last_modified_at: 2022-05-20
---

프로젝트를 하다가 문자열에 대한 수학 계산을 해야하는 일이 있었습니다.

예를들어 다음과 같이 "13 * 3 - 2"와 같은 문자열 표현식이 있을 때 연산자 우선순위를 고려하며 계산하려고 하면 구현해야 하는 내용이 많습니다.

대신 자바에서는 내장 자바스크립트 엔진이 있어 해당 엔진을 이용해 다음과 같이 문자열 표현식을 계산할 수 있습니다.

```java
import javax.script.ScriptEngineManager;
import javax.script.ScriptEngine;
import javax.script.ScriptException;

public class Test {
    public static void main(String[] args) throws ScriptException {
        ScriptEngineManager mgr = new ScriptEngineManager();
        ScriptEngine engine = mgr.getEngineByName("JavaScript");
        String foo = "40+2";
        System.out.println(engine.eval(foo));
    }
}
```