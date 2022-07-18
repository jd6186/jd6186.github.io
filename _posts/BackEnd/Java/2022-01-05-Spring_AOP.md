---
layout: post
title: "Spring Framework Legacy - AOP"
tags: [BackEnd JAVA]
author: "Noah"
categories: "back_end"
---

## 상위 글
#### ["Spring Framework Legacy"](https://jd6186.github.io/Spring_Framework_Legacy/) 정리

## Spring 3대 특징 중 "AOP"
### 개념
AOP(Aspect Oriented Programming)란 말 그대로 관점 지향 프로그래밍입니다.
Spring을 활용해 개발을 할 때는 Java자체가 OOP이기 때문에 결합도가 낮고 응집도가 높은 형태로 코딩이 가능합니다.
<br/>
때문에 여러 코드 내에 중복된 코드들이 많아지고 가독성, 확장성, 유지보수성을 떨어짐 이러한 문제를 보완하기 위해 나온 것이 AOP의 개념입니다.
<br/>
AOP에서는 OOP와는 다르게 핵심기능과 공통기능을 분리하여 개발이 가능하며 공통기능은 핵심로직에는 영향을 끼치지 않으며 개발 이후에는 작동 시점을 스스로 판단하고 알아서 작동하게 끔 제작하는 것이 일반적입니다.
<br/>
이로 인해 개발자는 핵심기능 제작에 집중할 수 있게 되고 공통기능을 수정할 때는 굳이 모든 핵심 기능들의 코드를 찾아다니면서 수정할 필요없이 AOP의 원칙으로 구분해 놓은 공통기능 하나만 수정하면 모든 핵심 기능들에 적용됩니다.
<br/>
이렇게 개발함에 따라 무분별하게 중복되는 코드를 한 곳에 모아 중복 되는 코드를 제거하며 이를 통해 효율적인 유지보수가 가능, 재활용성이 극대화됩니다.

### Spring AOP 작동 순서
<img src="../../assets/img/2021-12-09-Spring_Framework_Legacy/aop.jpg"  width="900"/>

위 그림을 분석해보자

1. Caller 실행
2. AOP Proxy클래스 호출
3. Advisor 중 Transaction을 관리하는 Advisor 호출
4. Transaction을 통해 caller가 실행된 시점에 맞는 Custom Advisor 호출
5. Custom Advisor와 target으로 지정된 Class(또는 메서드) 내에서 작동 시점에 맞춰 공통 기능 작업을 수행

왜 그럼 핵심 기능 내에서 공통 기능들은 Proxy를 거쳐 저렇게 작동될까요?
<br/>
이유는 Spring AOP 또한 결국 Spring Bean으로 등록 후 사용되기 때문입니다. 위에서 살펴본 IoC Container가 DL, DI의 작업을 거치는 것 처럼 AOP로 등록된 Bean들도 먼저 어떤 Bean이 현재 사용될지 판단되어야 하고 판단이 되었다면 그 내용을 target에 적용해야하기 때문에 저런식으로 작동됩니다.

### Spring AOP에서 사용되는 용어 정리
- Aspect : 위에서 설명한 흩어진 관심사를 핵심 기능과 공통 기능으로 모듈화 한 것. 주로 공통기능을 모듈화함.
- Target : Aspect를 적용하는 곳 (클래스, 메서드 ... )
- Advice : 실질적으로 어떤 일을 해야할 지에 대한 것, 실질적인 공통기능을 담은 구현체
- JointPoint : Advice가 적용될 위치, 끼어들 수 있는 지점. 메서드 진입 지점, 생성자 호출 시점, 필드에서 값을 꺼내올 때 등 다양한 시점에 적용가능
- PointCut : JointPoint의 상세한 스펙을 정의한 것. 'A란 메서드의 진입 시점에 호출할 것'과 같이 더욱 구체적으로 Advice가 실행될 지점을 정할 수 있음
- 예를 들어 Join Point로 지정된 메서드 실행 전, 중, 후 중 Advice의 동작 시점을 결정
