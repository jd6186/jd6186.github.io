---
layout: post
title: "AWS Solution Architect Associate 자격증 준비"
tags: [Java, Spring, Basic]
author: "DongUk"
---


## 목차
#### [1. Intro](#intro)
#### [2. 용어 이해](#용어-이해)
#### [3. AWS 용어 정리](#AWS-용어-정리)
#### [4. 실제 문제 풀이](#실제-문제-풀이)
#### [5. 마무리](#마무리)
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
> 아래 정리한 내용 확인
2. 실제 문제 풀이
> https://www.whizlabs.com/learn/course/aws-solutions-architect-associate/153 

## 용어 이해
### 인프라 네크워크 관련
용어 이해를 위해서는 먼저 기본적인 레거시 인프라에 대한 간단한 이해가 있으시면 편하실 것 같습니다.
<br/>
(위 내용을 알아야 추후 AWS의 각 서비스가 어떤 기능을 담당하는지 아실 수 있습니다.)
<br/>
웹 통신을 하기 위해서는 현재 국제 표준인 OSI 7계층을 이해하시는 것이 중요합니다.

IDC와 같은 내용도 존재하지만 여기서는 OSI 7계층을 보며 간단하게만 살펴보겠습니다.
<img src="../../assets/img/2022-01-11-AWS_Architect_Associate/OSI 7계층과 스위치.jpg"  width="900"/>

어플리케이션과 Web이 통신하기 위해서는 위와 같은 7가지 과정을 거쳐야 하며 레거시 인프라에서는 이런 기능을 하는 각 장비들을 마련해 줘야 합니다.

이해하기 쉽게 간단히 살펴보면

집에서 노트북 또는 데스크탑과 인터넷을 연결하는데 사용하는 장비가 무엇일까요?
여러분이 아는 **<strong style="color: #bb4177;">'랜선'</strong>**입니다. 랜선은 컴퓨터와 인터넷 연결 장비를 이어주는 기능을 하는 것이 L1 입니다.

간단하죠? 위에서 살펴보았듯이 무언가를 연결해주는 기능을 하는 것이 L1, L2, L3... 입니다.
자 그럼 L1끼리 통신을 뭘로 할까요? 

인터넷 연결 시 가장 기본이 되는 통신수단은 **<strong style="color: #bb4177;">'Mac Address'</strong>**이며 이것을 연결해주는 장비를 **L2 스위치**라고 부릅니다.
이 때 Mac Address는 현재 인터넷과 이어져 있는 다른 인터넷 연결장비가 어떤 것이 있는지 알려주는 기능을 합니다.

그럼 그 다음에 필요한 것은 무엇일까요?
각 인터넷별 고유한 주소 즉, **<strong style="color: #bb4177;">'IP'</strong>**이며 이것을 연결해주는 장비를 **L3 라우터**라고 부릅니다.
IP는 집주소와 비슷합니다. 배달을 시키더라도 정확한 집주소를 입력해야 배달이 가능한 것 처럼 인터넷 통신도 특정 서버의 정확한 IP주소가 있어야 통신이 가능합니다.

그럼 스위치와 라우터의 차이가 무엇일까요?
스위치는 장비간 통신을 위해 사용되며 라우터는 네트워크간의 통신을 하는 용도로 사용됩니다. 
때문에 외부의 다른 네트워크와 통신할 때에는 라우터를 사용해야 합니다.

예를 들어 설명드리면, 스위치는 자신에게 어떤 신호가 들어오면 그 신호를 다른 장비에게 전달해주기 위해 내부망을 스캔합니다.
반면, 라우터는 자신에게 들어온 신호가 내부망으로 보낼 신호인지 외부로 전달이 필요한 신호인지 판단하고 외부로 전달해야 하는 상황에서는 인터넷을 활용해 전달합니다.
<br/>
(하지만 요새는 L2장비에 L3장비의 기능을 혼합해 사용하는 추세이기 때문에 서로의 경계가 많이 모호해졌습니다.)

* L4는 **<strong style="color: #bb4177;">'Port'</strong>**를 제어하는 것입니다.
여기서 Port란 특정 IP주소와 통신하기 위한 항구입니다.
부산항에도 여러 항구가 있듯이 하나의 IP주소(컴퓨터) 안에는 동작할 수 있는 여러 서버가 있을 수 있습니다.
이들을 각각 구분하여 내가 원하는 서버에 요청을 전달할 수 있도록 관리하는 것이 Port라고 생각하시면 됩니다.<br/>
(그래서 보통 HTTP 요청은 80 포트로 수신하고 HTTPS 요청은 443포트로 수신하게 됩니다.)

L4를 관리하는 장비는 **로드벨런서**라고 불리며 Port번호를 보고 요청을 서버와 연결지어주는 역할을 합니다. 
보통 L6까지 연결하는 기능을 모두 가지고 있습니다.<br/>
(L2가 L3의 기능을 가지고 있던 것 처럼) <br/>

또한 L5~L7은 논리적인 부분이라 별도의 장비를 두지 않는 것이 현실입니다. 간단하게만 살펴보면
* L5는 유저 관리시 많이 사용됐던 **<strong style="color: #bb4177;">'세션'</strong>**을 담당하는 계층
* L6는 **<strong style="color: #bb4177;">'표현계층'</strong>**이라고 하는데 웹 기반 프로그램의 공통된 표준<br/>
(어렵게 생각하지 마세요. HTML, 아스키 코드 ...)
* L7은  Application Layer라고 부르며 대표적인 장비가 PC<br/>
입니다.

그렇다면 AWS에서는 어떻게 변경되었을까요?
<img src="../../assets/img/2022-01-11-AWS_Architect_Associate/AWS Architecture.jpg"  width="900"/>

1. 요청 수신(L2, L3 기능 담당)
> CloudFront(CDN), Route53(DNS), VPC(네트워킹) ...
2. 요청 판별(L4)
> ELB(요청에 맞는 Application 연결), ~ Gateway, ...
3. Application 관리(L7)
> EC2, Lambda, S3, ...

이 외에도 AWS에서는 보안, 역할, 정책 등과 관련된 수 많은 부분이 모두 서비스로 제공되고 있습니다.<br/>
아래 URL은 제가 AWS를 공부하면서 몰랐던 내용들에 대해 정리하던 내용들입니다.<br/> 
문제 풀면서 나왔던 내용들을 오답노트 정리하듯이 제작하고 있는 내용이라 AWS에 어떤 인프라 서비스들이 있는지 확인하시는 용도로 살펴보시면 좋을 것 같습니다.<br/>
https://www.notion.so/AWS-SAA-ef4c542e84094e1c88909ea1c9024cfb
<br/><br/><br/><br/>

## AWS 용어 정리
### Compute
#### EC2
#### Batch
#### Elastic Beanstalk
#### AWS Lambda
#### SAR
#### Fargate

### Container
#### Amazon EKS
#### Amazon ECS
#### Amazon ECR

### Storage
#### Amazon S3
#### S3 Glacier
#### AWS Backup
#### Amazon EBS
#### Amazon EFS
#### Amazon FSx(Window File Server)
#### Amazon FSx(Lustre)
#### AWS Snowball
#### AWS Storage Gateway

### Database
#### Amazon Aurora
#### Amazon RDS
#### Amazon DocumentDB
#### Amazon DynamoDB
#### Amazon ElastiCache
#### Amazon KeySpace
#### Amazon Neptune
#### Amazon Redshift

### Security, Identity, Compliance
#### AWS IAM
#### Amazon Cognito
#### AWS Directory Service
#### AWS RAM
#### AWS Secrets Managers
#### AWS Security Hub

### Cryptography & PKI
#### AWS KMS
#### AWS Certificate Manager

### Management & Govermance
#### AWS Auto Scaling
#### AWS CloudFormation
#### AWS CloudTrail
#### Amazon CloudWatch
#### AWS Config
#### AWS License Manager
#### AWS Organizations
#### AWS Systems Manager

### Development Tools
#### AWS CodeBuild
#### AWS CodeCommit
#### AWS CodeDeploy
#### AWS X-Ray

### Migration & Transfer
#### AWS DMS

### Networking & Content Delivery
#### AWS VPC
#### AWS PrivateLink
#### AWS Direct Connect
#### Amazon CloudFront
#### AWS Route 53
#### Amazon API Gateway
#### AWS Trasit Gateway
#### AWS ELB
#### AWS Cloud Map

### Front End Web & Mobile
#### AWS AppSync

### Application Integration
#### Amazon EventBridge
#### Amazon SNS
#### Amazon SQS
#### AWS Step Functions
#### Amazon SWF

### Billing & Cost Management
#### AWS Cost Explorer
#### AWS Budgets
#### AWS Cost & Usage
#### Reports
#### Reserved Instance
#### Reporting

### Monitoring
#### AWS Personal Health
#### Dashboard

### AWS Management Console
#### AWS Management
#### AWS Console
<br/><br/><br/><br/>

## 실제 문제 풀이
저는 문제를 whizlabs에서 구매하여 공부하였습니다. 가장 합리적인 가격에 많은 문제를 접할 수 있다고 생각해서 선택하게 되었구요.<br/>
AWS에 대한 기본적인 내용을 알고 있다고 생각하고 문제를 풀기 시작하였는데 생각보다 너무 많은 서비스들이 존재해서 당황했었습니다.<br/>
whizlabs 외에도 다양한 예상 문제를 제공하는 외국 사이트들이 많은데 반드시 예상 문제는 풀어보시고 시험보시기를 권장드립니다.<br/>
안다고 생각하는 것과 그것을 문제로 푸는 것은 또 다른 문제인 것 같습니다. ^^

아래는 whizlabs 링크입니다.<br/>
https://www.whizlabs.com/
<br/><br/><br/><br/>


## 마무리
#### 준비 기간 
22.01.11 ~ 22.04.15 (약 3달)<br/>

//TODO
자격증 취득하고 여기에 붙이기
<br/><br/>


#### 준비를 하며 느낀 점
학원을 다닐 때 부터 개발하는게 재밌어서 공부하다보니 인프라까지 공부를 하게 되었는데요.<br/>
실무에서 일하다보니 서버 에러가 아닌 인프라에서 장애가 발생 시 어디서 어떤 에러가 발생하는지 발견하시기가 어려웠고 개발한 서비스를 배포하기 전 부하 테스트 시 어떤 부분에서 트래픽이 몰려 서비스 품질을 떨어뜨리는지 알기 어려웠습니다.

위 내용들에 관심을 가지고 사내에서 진행한 프로젝트에 사용된 AWS 인프라들를 분해해 공부를 진행했고 현재는 자격증을 준비하며 개발팀 투입 시 AWS 인프라세팅을 직접 구현하다보니 자연스럽게 어떤 이슈들이 존재할 수 있는지 알게 되었습니다. 

또한 운영 업무를 겸업하고 있는 입장에서 장애 대응속도가 더 빨라지고 있어 인프라에 대한 이해가 전무하신 분들은 저처럼 한번은 공부할 가치가 있을 것 같아 여러분도 꼭 공부해보시기를 추천드립니다.<br/>

긴 글 읽어주셔서 감사합니다. 



