---
layout: post
title: "Spring Framework Legacy - IoC"
tags: [Back-End JAVA]
author: "Noah"
---

## 상위 글
#### ["Spring Framework Legacy"](https://jd6186.github.io/Spring_Framework_Legacy/) 정리

## Spring 3대 특징 중 "IoC"
### 개념
IoC는 Inversion of Control의 약자로 말 그대로 제어의 역전로써 기존 JAVA 프로그램 개발과는 반대로 작업이 진행되는 것을 의미합니다.
<br/>
일반적으로 지금까지 JAVA 프로그램은
<br/>
객체 결정 및 생성 -> 의존성 객체 생성 -> 객채 내의 메소드 호출 하는 작업을 반복
<br/><br/>
이는 각 객체들이 프로그램의 흐름을 결정하고 각 객체를 구성하는 작업에 직접적으로 참여한 것으로 쉽게 말해 모든 작업을 사용자가 제어하는 구조를 의미합니다.
<br/>
하지만 IOC 특성을 가진 Spring에서는 객체는 자기가 사용할 객체를 선택하거나 생성하지 않습니다. 또한 자신이 어디서 만들어지고 어떻게 사용되는지 또한 모르도록 추상화하여 관리합니다.
<br/>
다만! 자신의 모든 권한을 다른 대상에 위임함으로 써 제어권한을 위임받은 특별한 객체에 의해 롤이 결정되고 일정 규칙에 따라 생성되며 또한 그 롤과 규칙을 결정하는 것은 Spring Framework의 Container를 관리하는 Spring Core입니다.

### DL(Dependency Lookup) - 의존성 검색
컨테이너에서는 객체들을 관리하기 위해 별도의 저장소에 빈을 저장하는데 저장소에 저장되어 있는 개발자들이 컨테이너에서 제공하는 API 를 이용하여 사용하고자 하는 Spring Bean을 검색하는 방법입니다.

### DI(Dependency Injection) - 의존성 주입
의존성 주입이란 객체가 서로 의존하는 관계가 되게 의존성을 주입하는 것입니다.
<br/>
객체지향 프로그램에서 의존성 이란 **하나의 객체가 어떠한 다른 객체를 사용하고 있음을 의미**

### IoC에서의 DI 요약
1. 빈 설정 정보에 원하는 설정 조건을 작성
2. 각 클래스 사이에 필요로 하는 의존관계를 설정
3. 빈 설정 정보를 바탕으로 컨테이너가 자동으로 의존관계에 놓인 객체들을 연결