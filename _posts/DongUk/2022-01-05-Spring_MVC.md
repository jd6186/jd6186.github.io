---
layout: post
title: "Spring Framework Legacy - MVC"
tags: [Java, Spring, Basic]
author: "DongUk"
---

# Spring MVC 작동원리
## Spring을 활용한 Web MVC 작동원리

<img src="../../assets/img/2021-12-09-Spring_Framework_Legacy/Client_to_DB.jpg"  width="900"/>

1. Web Server(Apache)에서 request 수신
2. tcp/udp 패킷 내 정보를 확인해 Web Application Server로 전달
3. tomcat 코어인 Catalina에서 관리하는 접근 권한을 확인
4. 통과 시 Catalina가 관리하는 작업 대기열(Queue)에 대기 상태로 Loading
5.  Catalina가 Thread Pool을 확인하여 여분의 Thread가 존재하는지 확인하여 작업 대기열 내 작업들에게 배분
6. Web Server에서 전달한 내용을 확인하여 httpServletRequest 생성
7. Spring에서 작성한 web.xml을 확인하여 맵핑된 Dispatcher Servlet이 있는지 확인, 없다면 맵핑된 다른 Servelt이 있는지 확인 후 요청 전달
    - Dispatcher Servlet
        - 여러 요청들이 들어올 때 어느 Controller에게 해당 요청을 보내야하는지 결정해주는 역할
        - Spring MVC 안에 내장되어 있는 객체이며 XML을 활용해 관리가 가능 (요청 분기 및 Controller로 사용될 Bean 등록 등)
        - XML을 이용해 다른 POJO형식의 Controller들을 Dispatcher Servlet과 연결된 Bean으로 설정해주면 Controller로서 동작 시킬 수 있음
        - Spring에서는 위 작업 없이 POJO형식의 Class에서 @Controller라는 Annotation을 활용해 해당 Class를 Spring Bean으로 추가하는 기능도 지원함
8. Dispatcher Servlet에서 Spring에 등록된 Handler(Controller)들을 확인하여 각 요청에 맞게 Mapping
9. Handler(Controller) 호출 전 등록된 Interceptor가 존재하는 지 확인 후 Before 조건으로 등록된 작업을 먼저 실행
10. Handler Adapter를 활용해 들어온 요청에 Mapping된 Controller 호출
11. @Controller 진입 시 @AutoWired로 연결되어 있는 Spring Bean(Service)들을 DI(Dependency Injection)을 활용해 주입
12. @RequestMapping 어노테이션들을 확인하면서 요청들어온 URI 및 HTTP Method에 맞는 @RequestMapping된 Method들을 조회
13. @RequestMapping으로 연결된 Method 실행
14. Method 실행
15. 실행 중 @Service(Spring Bean)을 활용하거나 @Repository(Spring Bean)를 활용해 DB 통신을 해야할 경우 아래 순서로 작동
    1. Mybatis 사용 시
        1. 필요한 기능이 정의된 Method가 구현된 Spring Bean(@Service)를 찾음
        2. 그 후 DB활용이 필요 시 추상화된 DAO 인터페이스를 조회
        3. 해당 데이터 조회 기능을 가진 실제 Spring Bean(@Repository)을 조회
        4. Spring의 SqlSessionTemplate을 활용해 Mybatis의 SqlSessionFactoryBean을 조회
        5. Mybatis에 Setting한 xml을 활용해 DB와 Pooling 진행
        6. 획득한 데이터를 DTO에 담아 SqlSessionTemplate으로 다시 전달
        7. DTO에 담긴 데이터를 활용해 DAO 작업 처리
        8. DAO에서 처리된 데이터를 활용해 Service 구현
        9. Service 작동에 대한 결과값을 Controller로 전달
    2. JPA사용 시
        1. Spring Legacy와 JPA 사용 시 DB와 pooling을 하는 방법에 대한 공부 필요
16. Controller에서는 Service에서 MAV형태로 제작된 결과값을 String형태로 Dispatcher Servlet으로 전달
    
    Map 형태로 된 것들은 View Resolver로 다이렉트로 전달 가능
    
17. DispatcherServlet에서 Resolver 호출 시
    1. DispatcherServlet에서 Default(JSP용)로 제공되는 ViewResolver(Internal Resource View Resolver) 사용 시
        1. 기본적으로 ViewResolver가 ViewObejct 들을 캐싱해두었다가 사용됌
        2. Controller에서 return 값으로 ViewObject의 이름을 반환할 시 해당 ViewObject를 Mapping해줌
        3. Mapping된 ViewObject는 Controller를 통해 넘어온 데이터를 확인 후 화면(HTML) 제작
        4. 제작된 HTML을 사용자에게 전달(Response)하고 필요 시 2차, 3차에 거쳐 JS, CSS등을 주고 받는 통신 과정을 거치게 됌
    2. JSON Resolver 사용 시
        1. 주로 ajax, axios 통신과 같이 비동기 통신 요청이 들어올 때 많이 사용(Front단이 SPA일 경우 자주 사용됌)
        2. return값으로 html을 제작해 return하는 것이 아닌 json데이터만을 넘기기 때문에 통신속도가 매우 빠름
18. 위 과정을 MVC 부분만 도식화하면 아래와 같다

<img src="../../assets/img/2021-12-09-Spring_Framework_Legacy/SpringMVC.jpg"  width="900"/>