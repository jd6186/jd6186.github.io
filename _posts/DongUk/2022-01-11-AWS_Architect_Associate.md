---
layout: post
title: "AWS Solution Architect Associate 자격증 준비"
tags: [Java, Spring, Basic]
author: "DongUk"
---


## 목차
#### [1. Intro](#intro)
<br/><br/>

## Intro
AWS 관련 자격증을 준비하며 관련 내용을 공유드리면 좋을 것 같아서 블로그 포스팅을 시작합니다.
<br/>
저도 자격증 취득을 위해 공부한 내용을 정리해서 올릴 예정이라 취득 전 까지 계속 글을 갱신할 예정입니다.
<br/>
저는 실무에서 이미 Lambda기반 프로젝트, ECS Fagate기반 프로젝트를 설계 및 진행해본 사람으로
<br/>
현재 완전 처음 AWS를 접하신 분들과는 차이가 있다는 점 미리 말씀드리고 시작하겠습니다.

준비 과정은 크게 두가지로 나뉩니다.
1. 용어 이해
2. 실제 문제 풀이

### 용어 이해
용어 이해를 위해서는 먼저 기본적인 레거시 인프라에 대한 간단한 이해가 있으시면 편하실 것 같습니다.
<br/>
(위 내용을 알아야 추후 AWS의 각 서비스가 어떤 기능을 담당하는지 아실 수 있습니다.)
<br/>
웹 통신을 하기 위해서는 현재 국제 표준인 OSI 7계층을 이해하시는 것이 중요합니다.

IDC와 같은 내용도 존재하지만 여기서는 OSI 7계층을 보며 간단하게만 살펴보겠습니다.
<img src="../../assets/img/2022-01-11-AWS_Architect_Associate/OSI 7계층과 스위치.jpg"  width="900"/>

어플리케이션과 Web이 통신하기 위해서는 위와 같은 7가지 과정을 거쳐야 하며 레거시 인프라에서는 이런 기능을 하는 각 스위치들을 마련해 줘야 합니다.


그렇다면 AWS에서는 어떻게 변경되었을까요?
<img src="../../assets/img/2022-01-11-AWS_Architect_Associate/AWS Architecture.jpg"  width="900"/>
필자의 경험으로 판단해보면 크게 3분류로 변경되었습니다.
1. 요청 수신(L2, L3 기능 담당)
> CloudFront(CDN), Route53(DNS), VPC(네트워킹) ...
2. 요청 판별(L4)
> ELB(요청에 맞는 Application 연결)
3. Application 관리(L5, L6, L7)
> ECS, Lambda
