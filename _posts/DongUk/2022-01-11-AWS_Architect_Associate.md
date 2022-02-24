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
#### Amazon Simple Workflow Service(SWF)
1. SWF란?

<br/><br/>

### Billing & Cost Management
#### AWS Cost Explorer
1. Cost Explorer(비용 탐색기)란?
    * 비용 및 사용량을 분석할 수 있는 UI 도구
    * 사용자에게 그래프 지원
    * CostExplorer의 비용 및 사용 보고서 및 CostExplorer RI 보고서 지원
    * Billing 또는 CostManagement에서 접근 가능
2. 분석을 위한 기본 리포트(보고서) 제공
    * 보고서를 생성하기 위한 일부 필터 및 제약 조건이 있는 보고서
    * bookmark 또는 CSV 파일로 저장 가능
3. 기본 리포트 종류
    1. Cost and usage reports(비용 및 사용 보고서)
        * 산출된 비용을 이해하기 위한 데이터 제공
            1. AWS Marketplace
            2. Daily 비용
            3. Monthly 계정 비용
            4. Monthly 서비스 비용
            5. Monthly EC2 사용 시간에 따른 비용 및 사용 내역
    2. RI(Reserved Instance) reports(예약 인스턴스 보고서)
        * 예약된 내역을 이해하기 위한 데이터 제공
            1. RI utilization reports(RI 활용 보고서)
                * 예약 인스턴스를 사용함으로써 절감 또는 초과된 **비용**에 대한 내역 제공
            2. RI coverage reports(RI 적용 범위 보고서)
                * 예약 인스턴스를 사용함으로써 얼마나 많은 인스턴스 사용 **시간**을 절감 또는 초과하였는지 내역 제공
3. 기본 리포트 특징
    * 유저가 CostExplorer 최초 가입 시 console의 main part로 이동
        그 후 준비된 비용 및 사용 내역에 관한 12달 간의 과거 히스토리 데이터들, 현재 달, 다음 12달의 데이터를 예측해서 보여줌
    * AWS Cost, Usage Reports, billing reports와 동일한 데이터 세트를 사용
    * 유저의 기호에 따라 월별, 일별 데이터를 조회할 수 있는 기능 제공
    * AWS 컴퓨팅 사용량을 최대 72%까지 절약 가능한 Saving Plan 제공
    * Cost Explorer API를 제공하여 개발 시 활용 가능
4. 가격 정보
    * 사용한 비용 및 사용량 분석 조회는 무료
    * API 사용 비용은 API Request(요청)당 $0.01

#### AWS Budgets
1. AWS Budgets(예산)란?
    * 유저에게 예산을 자유롭게 설정하게 함으로써 가장 단순한 사용사례부터 복잡한 사용사례까지 비용과 사용량을 track(추적) 가능한 서비스
2. AWS Budgets 특징
    * AWS Budgets는 예약 사용률 또는 적용 대상 지정하고 metrics reach the threshold(지표가 임계값에 도달했을 때)일 때 당신에게 email 또는 SNS로 알림을 보내도록 허락하는데 사용 가능.  
    * AWS Budgets은 EC2, RDS, Redshift, ElasticCache, Elasticsearch에 예약 알림 기능을 제공함
    * AWS Budgets은 speific dimensions(특정 수치들)로 필터링하여 알림 수신 가능<br/>
    Service(서비스별), Linked Account(연결된 계정별), Tags(태그별), Availability Zone(AZ, 즉, 가용 영역별), API Operation(API 작동별), API Purchase Option(API 구매 옵션별)
    * AWS Budgets는 AWS Management Console's service links와 AWS Billing Console을 내에서 접근 가능<br/>
    또한 Budgets API 또는 CLI는 각 지불 계정별로 최대 20,000개 예산들을 create, edit, delete, view up(조회)하는데 사용 가능
    * AWS Budgets는 다른 AWS Services와 통합될 수 있다.<br/>
    AWS Cost Explorer, AWS Chatbot, AWS Amazon Chime room, AWS Service Catalog
    * AWS Budgets는 AWS 리소스 사용 내역 또는 AWS 비용들을 Monthly(달별), quarterly(분기별), annual(연별) 예산들로 구분해 생성 가능
3. AWS Budgets를 활용해 만들 수 있는 예산의 타입들
    1. 비용 예산
    2. 사용 이력 예산
    3. RI 활용 예산
    4. RI 적용 예산
    5. Saving Plans 활용 예산
    6. Saving plans 적용 예산
4. 모범 사례
    1. 각 예산별로 5가지의 알림을 세팅 가능
        * 주의 사항
            1. 알림들은 이번달 비용이 예산액을 초과할 때 알림
            2. 알림들은 이번달 비용이 예산액의 80%를 넘어설 때 알림
            3. 알림들은 월별 예상 비용이 예산액을 초과할 때 알림
    2. Budget API를 활용해 예산들을 생성 시, 만약 여러 유저가 Budgets API에 접근해야 한다면 각 유저별들의 액세스 또는 IAM 역할을 허용하기 위해 따로 구분된 IAM 유저를 만들어야 함
    3. organization 내 통합 결제 사용을 마스터 계정을 통해 관리하는 경우, IAM policies(정책)는 member 계정별로 예산 접근에 대한 액세스 컨트롤이 가능<br/>
    각 Member 계정 주인들은 그들의 예산들을 생성을 할 수는 있지만 마스터 계정의 예산들을 변경하거나 수정하는 것은 불가능
    4. 관련 관리 정책들 중 두가지는 예산 조치를 위해 공급<br/>
    하나의 정책은 유저가 예산 서비스에 역할을 전달할 수 있도록 함<br/>
    다른 하나의 정책은 유저가 예산으로 해당 조치를 취할 수 있도록 허용함
    5. 예산 조치들은 Auto Scaling Groups으로 비용을 제어하는 것 만큼 효과적이진 않음(즉, Auto Scaling groups로 비용 제어하는 것이 더 효과적)
5. 가격 정책
    * 예산을 모니터링하는 것과 알림을 받는 것은 무료로 제공
    * 후속 조치 가능한 각 예산들은 무료 할당량이 종료된 후 부터 일일 $0.1가 부과됨

#### AWS Cost & Usage
1. Cost와 Usage란?
    * AWS Cost & usage Report(AWS CUR)는 유저들이 AWS 리소스, 가격, Reserved Instances(예약 인스턴스들), Savings Plans들에 대한 메타데이터를 포함한 디테일한 AWS 비용, 사용 데이터 세트에 접근하도록 승인한다.
2. AWS Cost & Usage Report는 AWS Cost Explorer의 일부분
    * AWS Cost & Usage Report function의 종류
        1. Amazon S3 bucket으로 report file 전송
        2. 하루에 최대 3번 report를 update 진행
        3. AWS CUR API Reference를 활용해 report를 생성, retrieves(검색), 삭제 진행
    * 상세한 비용 및 사용 내역 분석을 손쉽게 리포트 안에 컬럼 형태로 추가하고 나열할 수 있는 데이터 사전의 특징을 가짐.
    * 조회 시에는, reports는 Amazon S3 console으로 부터 다운로드가 가능, Amazon Athena를 사용해 분석 가능하며 또는 Amazon Redshift, Amazon QuickSight를 통해 리포트 upload 가능
    * IAM 권한들 또는 IAM 역할이 있는 유저들은 리포트에 액세스 및 조회 가능
    * 조직의 구성원 계정이 비용 및 사용량 보고서를 소유하거나 생성하는 경우 해당 계정이 조직의 구성원이었던 기간 동안의 청구한 데이터에만 액세스할 수 있음
    * 조직의 마스터 계정이 Cost & Usage Report에 Member 계정들의 접근을 block하고 싶다면 Service Control Policy를 사용할 수 있음.

#### Reserved Instance Reporting
1. Reserved Instance(RI) Reporting이란?
    * Reserved Instance Utilization and Coverage reports라고도 불리는 RI Reporting은 AWS Cost Explorer에서 이용 가능
2. RI Reporting 특징
    * 이는 특정 기간 동안에 AWS Resource에 의해 발생한 RI 사용률 또는 초과 사용된 내역이 존재하는지 확인 할 때 사용할 수 있다.<br/>
    (Amazon EC2, Amazon Redshift, Amazon RDS, Amazon Elasticsearch Service, Amazon ElastiCache)
    * AWS Cost Explorer는 과거 사용 내역을 기반으로 RI 구매에 대한 권장 사항을 제공.<br/> 
    이를 통해 온디멘드 대비 비용을 더 많이 절약할 수 있는 기회 제공
    * RI Utilization report로 부터 정보에 접근하기 위해서는 AWS Cost Explorer를 사용 필수
    * Reserved Instance Utilization과 Coverage report는 둘 다 PDF나 CSV 형식으로 내보내기 할 수 있음
3. Reporting 종류
    1. RI utilization reports
        * RI Reporting은 계정에서 사용한 총 RI 시간을 표시하고 모든 RI 및 서비스에 걸쳐 결합된 활용률을 이해하고 모니터링하는데 도움이 됨
        * AWS는 예약 저축액에서 사용하지 않는 예약 비용을 빼서 총 절감액을 계산함
    2. RI coverage reports
        * RI Reporting 은 계정의 instance 사용 시간을 보여주고 당신의 모든 RI들의 적용 내역을 통합하여 이해하기 쉽고 모니터링하기 쉽게 도와줌
        * RI coverage reports는 구매한 계정들, 인스턴스 형식, 기타 등등을 분석하기 위해 RI Utilization 필터들 대신 Cost Explorer 필터들을 사용한다.
    3. Reserved Instance Utilization and Coverage reports
        * RI utilization reports의 목표 이용량과 RI coverage reports의 목표 적용 범위는 차트 안에서 점선 또는 차트 아래 표에 컬러가 들어간 상태 막대 그래프로 표현 가능함.
            * Red status bar - 인스턴스가 적용은 되어 있으나 사용되지 않은 RI들
            * Yellow status bar - 목표 이용량보다 아래인 것들
            * Green status bar - 목표 이용량에 도달한 것들
            * Gray status bar - 예약에 적용조차 되지 않은 인스턴스들
        * RI reports는 RI-specific과 Cost Explorer의 필터를 조합해 사용하여 제작
        * 일간 RI Utilization chart는 이전 3개월 분의 데이터를 활용하여 RI 이용 내역을 보여줌
        * 월간 RI Utilization chart는 이전 12개월 분의 데이터를 활용하여 RI 이용 내역을 보여줌
4. 가격 정책
    * 데이터 조회 요청당 $0.01 USD
<br/><br/>

### Monitoring
#### AWS Personal Health
1. AWS Personal Health Dashboard란?
    * AWS Personal Health Dashboard는 AWS Health API를 활용 가능<br/>
    이를 통해 AWS 리소스 및 인프라와 관련된 문제를 진단하고 해결하기 위한 방법을 제시하거나 알림들을 전달 할 수 있음
    * AWS Health는 지속적으로 AWS 서비스 성능과 가용성을 시각화하여 제공함
    * AWS Personal Health Dashboard는 Amazon CloudWatch 이벤트와 통합되어 사용자 지정 규칙을 생성하고 업데이트 적용 작업을 사용하도록 AWS Lambda functions와 같은 대상을 지정할 수 있음
2. AWS에서 제공하는 두가지 대쉬보드
    1. AWS Service Health Dashboard
        * 현재 상태에 대한 액세스를 제공하고 전체 지전안에 있는 전체 서비스들의 완전한 health check를 지원
    2. The Personal Health Dashboard
        * 계정에 연결된 리소스들에 영향을 끼칠 수 도 있는 서비스 중단에 대한 알림을 제공
3. Personal Health Dashboard의 3가지 카테고리
    1. Open issues
        * 최근 7일간의 이슈들을 보여줌
    2. Scheduled changes
        * 향후 변경 사항이 있는 항복을 보여줌
    3. Other notifications
        * 기타 알림
<br/><br/>

### AWS Management Console
#### AWS Management Console
1. Managment Console이란?
    * Managment Console은 Web Application<br/>
    이는 Amazon Web Service들을 관리하기 위해 많은 서비스 콘솔들로 이루어져 있음
2. Managment Console 특징
    * Managment Console은 유저가 처음으로 로그인 했을 때 보여질 수 있음
    * AWS 탐색을 위해 사용자 인터페이스를 제공하고 다른 서비스 콘솔들에 대한 액세스를 제공
    * AWS Management Console은 네비게이션바 위에 가장 최근에 조회했던 목록 또는 전체 서비스 목록으로 부터 선택할 수 있는 서비스 옵션을 제공함
    * 네비게이션 바에는 리전을 선택할 수 있는 옵션 존재
    * 네비게이션 바에는 전체 또는 일부 Service 명을 입력함으로써 AWS 내 모든 Service들을 검색할 수 있는 검색창이 존재
    * Management Console 이용 시 더 나은 터치 경험을 제공하기 위해 최대된 수평, 수직 공간 그리고 큰 버튼 등을 안드로이드나 iOS와 같은 app을 통해 이용할 수 있음. 
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



