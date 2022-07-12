---
layout: post
title: "ECS Build 시 Docker Build Limit에 걸리는 이슈 해결"
tags: [Back-End AWS]
author: "Noah"
---

## Intro
안녕하세요 **Noah**입니다.

AWS를 실무에서 사용 시 주로 ECS Fargate 및 CodePipeline을 기반으로 인프라를 설계하는 프로젝트가 많아 해당 아키텍처 구성 시 도움이 될 만한 내용을 공유드리고자 합니다.
Docker는 유료 서비스가 생기면서 2020년 11월부로 무료 회원에 한해서는 Docker Build 시 회수 제한이 생기게 되었습니다.

이 때문에 주로 무중단배포환경을 이용 중이면서 많은 배포가 일어나는 환경(Ex_개발서버)에서 Docker Build Limit 이슈가 발생하여 작업 진행이 되지 않았던 이슈가 있었습니다.

해당 이슈를 해결한 방법을 공유드리겠습니다.
<br/><br/><br/><br/>


## Build Limit은 어떤 상황에 걸리게 되는가?
Docker를 사용해 본 분들은 알겠지만 일반적으로 최초에 기본 이미지를 FROM절을 통해 가져오고 
가져온 이미지에 원하는 작업을 추가해 컴퓨팅 환경을 구성하게 됩니다.
> EX) Maven을 사용해 Build 후 서버 실행하는 샘플
>    ```dockerfile
>       FROM maven:3.8.2-amazoncorretto-11
>       
>       USER root
>       
>       ARG SRC=/src
>       COPY ${SRC} /src
>       ARG POM_FILE=/pom.xml
>       COPY ${POM_FILE} pom.xml
>       
>       RUN ["mvn","clean","package"]
>       
>       RUN useradd -ms /bin/bash my_server
>       USER my_server
>       
>       EXPOSE 8080
>       EXPOSE 443
>       ENTRYPOINT java -Xms256m -Xmx4g -jar ./target/my-api-server-0.0.1.war
>   ```
<br/><br/>
여기서 FROM절을 보게 되면 "maven:3.8.2-amazoncorretto-11"이라는 이미지를 Docker Hub로부터 불러와 사용하는 것을 볼 수 있습니다.
<br/>
Docker Limit은 이렇게 Hub에서 기본 이미지를 호출하는 회수를 제한하는 것입니다.
<br/>
따라서 이미지를 Docker Hub로부터 호출하지 않는다면 해당 이슈를 해결할 수 있습니다.
<br/><br/><br/><br/>

## 해결 방법
해결 방법은 **<strong style="color: #bb4177;">'이미지를 다른 레포지토리에 Build 후 호출하여 사용하라.'</strong>**입니다.
<br/>
필자의 경우 AWS를 사용하고 있기 때문에 ECR에 기본 이미지를 Build하여 사용하고 있습니다.
<br/>
이때 여러 프로젝트에서 사용할 수 있는 공용 이미지를 ECR 내 Build하고 해당 이미지 주소를 호출하여 해당 이슈를 해결하였습니다.

아래 예시를 살펴보겠습니다.
1. ECR에 이미지 등록
> ECR 내 등록된 기본 이미지 정보
> <img src="../../assets/img/2022-07-12-AWS_ECS_Dockerbuild_Limit_Issue/ECR.png"  width="700"/>
> 
> 내부 버전정보
> <img src="../../assets/img/2022-07-12-AWS_ECS_Dockerbuild_Limit_Issue/MAVEN.png"  width="700"/>

2. Dockerfile FROM절 구성
> ECR 내 이미지지를 Build 해두었다면 해당 이미지의 URL 정보를 활용해 Dockerfile FROM절 구성
>   ```dockerfile
>       FROM 3**********.dkr.ecr.ap-northeast-2.amazonaws.com/maven:3.8.2-amazoncorretto-11
>       
>       USER root
>       
>       ARG SRC=/src
>       COPY ${SRC} /src
>       ARG POM_FILE=/pom.xml
>       COPY ${POM_FILE} pom.xml
>       
>       RUN ["mvn","clean","package"]
>       
>       RUN useradd -ms /bin/bash my_server
>       USER my_server
>       
>       EXPOSE 8080
>       EXPOSE 443
>       ENTRYPOINT java -Xms256m -Xmx4g -jar ./target/my-api-server-0.0.1.war
>   ```

위와 같이 AWS를 사용하고 있으신 경우 ECR 내 Docker Image를 배포하시고 해당 이미지 URL을 FROM절에 입력하게 되면 
Docker Hub가 아닌 ECR에서 Image를 호출하기 때문에 해당 이슈가 해결되는 것을 확인할 수 있습니다.

이때 기본 이미지를 Build하실 때는 여러 프로젝트에서 공용으로 사용할 수 있도록 가장 기본적인 컴퓨팅 환경으로 세팅하시는 걸 추천드립니다.
 
위 글과 관련하여 더 궁금하신 내용이 있다면 이메일로 메일 주시면 답변드리도록 하겠습니다~

긴 글 읽어주셔서 감사합니다.