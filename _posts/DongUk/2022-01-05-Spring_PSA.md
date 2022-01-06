---
layout: post
title: "Spring Framework Legacy - PSA"
tags: [Java, Spring, Basic]
author: "DongUk"
---

# 상위 글
** ["Spring Framework Legacy"](https://jd6186.github.io/Spring_Framework_Legacy/) **

# Spring 3대 특징 중 "PSA"
이해하기 어려울 수도 있는 개념이지만 간단한 예시를 통해 알아보면
우선 PSA는 Spring을 통해 프로젝트를 만들면서 **개발자가 직접적으로 컨트롤하거나 구현할 수 있는 개념이 아닙니다.**

예를 들어
@Controller Annotation이 붙어있는 클래스에서 @GetMapping, @PostMapping과 같은 
@RequestMapping Annotation을 사용해서 요청을 매핑해 사용하는 경우를 살펴보면
실제로는 내부적으로 서블릿 기반으로 코드가 동작하지만 서블릿 기술은 추상화 계층에 의해 숨겨져 있어 확인이 되지 않습니다.

- PSA 중 SA(Service Abstraction)란?
    1. Service Abstraction 이란 추상화 계층을 사용해서 어떤 기술을 내부에 숨기고 개발자에게 편의성을 제공해 주는 것을 의미
    2. DB에 Transaction 처리를 할 때 Commit , RollBack같은 low level의 작업이 필요할 경우<br/>
        스프링이 제공하는 `@Transactional`이라는 추상화 계층을 사용하면 Try, Catch, AutoCommit, Rollback 등과 같은 처리를 직접 코드로 작성할 필요 없이 Transaction을 적용할 수 있게 됌
    3. 이제 다시 P를 붙여 PSA의 개념에서 위 예시를 적용 시<br/>
        우리가 개발 초기에는 JDBC를 사용하다가 이후 JPA로 변경한다고 해서 `@Transactional` 사용할 수 없게 되는 것은 아님
        심지어 트랜잭션 처리에 대해서는 변경해야 하는 코드도 존재X
        즉, JPA를 사용하던 MyBatis또는 JDBC를 사용하던 상관 없이 트랜잭션 처리를 하는데 있어서는 단순히 `@Transactional` 만 알고 있다면 따로 신경쓸것이 없다는 것

# PSA 요약
1. 추상화 계층을 사용해서 어떠한 기술을 내부로 숨김
2. 개발자에게 편의성을 제공해주는 Service Abstraction 코드를의 수정 없이 유연하게 선택
3. 선택된 코드를 원한 곳에서 사용할 수 있도록 해주는 Spring 자체 편의 기술
