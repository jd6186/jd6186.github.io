---
layout: post
title: "[AWS] ECS와 Kubernetes 비교에 대한 간단한 글(EKS 인프라 구축편)"
tags: [AWS, ECS, EKS, Kubernetes, DevOps]
---

# Intro
안녕하세요, Noah입니다.<br/>
오늘은 **Kubernetes(쿠버네티스)를 사용해 환경을 구축하기 위한 세팅 방법**에 대해 설명드리겠습니다. 쿠버네티스는 클라우드와 온프레미스 환경 모두에서 사용할 수 있는 유연한 오케스트레이션 도구로, 다양한 애플리케이션을 안정적으로 배포하고 관리할 수 있습니다.<br/>
이번 글에서는 인프라 구조부터 네트워킹 및 보안 설정, 관리 및 모니터링 도구, 그리고 코드형 인프라(IaC)를 통해 쿠버네티스 환경을 설정하는 방법까지 차근차근 알아보겠습니다.

그럼 시작해 보겠습니다.
<br/><br/><br/><br/>

# 목차
- [Intro](https://www.notion.so/2024-11-1-134c7f0b0a68806091c0db5506444982?pvs=21)
- [쿠버네티스를 사용해 환경을 구축하기 위해 필요한 환경 세팅](https://www.notion.so/2024-11-1-134c7f0b0a68806091c0db5506444982?pvs=21)
    1. [인프라 구조도 작성](https://www.notion.so/2024-11-1-134c7f0b0a68806091c0db5506444982?pvs=21)
    2. [관리 및 모니터링 도구 설정](https://www.notion.so/2024-11-1-134c7f0b0a68806091c0db5506444982?pvs=21)
    3. [네트워킹 및 보안 설정 방법](https://www.notion.so/2024-11-1-134c7f0b0a68806091c0db5506444982?pvs=21)
    4. [코드형 인프라 (IaC)로 클라우드 포메이션을 활용한 구축](https://www.notion.so/2024-11-1-134c7f0b0a68806091c0db5506444982?pvs=21)
- [Outro](https://www.notion.so/2024-11-1-134c7f0b0a68806091c0db5506444982?pvs=21)<br/><br/><br/><br/>
<br/><br/><br/><br/>

# 쿠버네티스를 사용해 환경을 구축하기 위해 필요한 환경 세팅
## 1. 인프라 구조도 작성
쿠버네티스를 사용하여 인프라를 설정할 때는 전체 클러스터의 구조를 쉽게 파악할 수 있는 **아키텍처 구조도**를 작성하는 것이 중요합니다. 
아래와 같은 **쿠버네티스 기반 아키텍처 구조도**를 통해 각 구성 요소의 상호 관계를 쉽게 파악할 수 있습니다.

[아래는 기본적인 쿠버네티스 아키텍처 예시]
- **Kubernetes Cluster**: 쿠버네티스 클러스터는 컨트롤 플레인(Control Plane)과 워커 노드(Worker Node)로 구성됩니다. 컨트롤 플레인은 클러스터 전체를 관리하며, 워커 노드는 실제 애플리케이션 컨테이너가 실행되는 곳입니다.
- **Pods**: 애플리케이션 컨테이너가 배포되는 최소 단위입니다. 하나 이상의 컨테이너로 구성될 수 있고, 네트워크 및 스토리지 리소스를 공유합니다.
- **Namespaces**: 클러스터 내에서 환경을 격리하기 위해 사용됩니다. 여러 애플리케이션이 같은 클러스터 내에 있더라도 네임스페이스를 통해 독립적인 환경을 구성할 수 있습니다.
- **Service와 Ingress**: Pods와 외부 간의 트래픽을 제어합니다. **Service**는 특정 포드 집합에 트래픽을 전달하며, **Ingress**는 외부에서 클러스터 내부로의 HTTP/HTTPS 접근을 관리합니다.<br/>(이걸 보면서 왜 ECS 클러스터 내 Service 항목이 존재하는지 알게 되었습니다... ㅋㅋㅋㅋ)
- **Persistent Volumes**: 클러스터 내에서 데이터 영속성을 보장하는 스토리지 리소스입니다.
<br/><br/>

## 2. 관리 및 모니터링 도구 설정
쿠버네티스는 분산 환경에서 여러 애플리케이션을 안정적으로 운영하기 위해 다양한 관리 및 모니터링 도구를 제공합니다.<br/>
이와 같은 관리 및 모니터링 도구를 통해 쿠버네티스 클러스터 상태를 효과적으로 관리하고, 서비스 장애 발생 시 빠르게 문제를 파악할 수 있습니다.

- **Prometheus와 Grafana**<br/>
  Prometheus는 쿠버네티스 클러스터의 성능을 모니터링하는 오픈소스 도구로, CPU 및 메모리 사용량, 네트워크 트래픽, 에러 등을 수집합니다. 수집된 데이터를 **Grafana**와 통합하여 **시각화**할 수 있습니다. 특히, 알림 규칙을 설정해 특정 지표가 임계치를 초과하면 알림을 받을 수 있습니다.
- **Kubernetes Dashboard**<br/>
  쿠버네티스 클러스터의 상태를 시각적으로 모니터링할 수 있는 웹 UI입니다. Pods, Deployments, 서비스, 네임스페이스 등 주요 리소스 상태를 실시간으로 확인할 수 있어 클러스터의 상태를 쉽게 파악할 수 있습니다.
- **Fluentd 및 Elasticsearch**<br/>
  쿠버네티스 환경에서 로깅을 효율적으로 관리하기 위해 자주 사용됩니다. **Fluentd**는 각 포드의 로그 데이터를 수집하고 **Elasticsearch**로 전송하여 저장합니다. 이를 통해 클러스터의 전체 로그 데이터를 중앙에서 관리하고, 특정 이벤트나 에러를 빠르게 검색할 수 있습니다.
- **Jaeger**<br/>
  Jaeger는 분산 트랜잭션을 추적하는 도구로, 애플리케이션 간의 호출 관계와 병목 지점을 파악하는 데 유용합니다. 트랜잭션의 흐름을 시각화하여 장애 분석과 성능 최적화에 도움을 줍니다.
<br/><br/>

## 3. 네트워킹 및 보안 설정 방법
쿠버네티스의 네트워크 및 보안 설정은 외부 트래픽 제어, 클러스터 간 통신, 접근 제어를 위해 매우 중요합니다. 쿠버네티스 환경에서 주로 사용하는 네트워크와 보안 설정 방법은 다음과 같습니다.<br/>
네트워크와 보안 설정은 쿠버네티스 클러스터의 안정성과 보안을 유지하는 데 필수적인 요소이므로, 각 구성 요소와 리소스에 맞는 보안 설정을 적용하는 것이 중요합니다.

- **네트워크 구성 (CNI Plugin)**<br/>
  쿠버네티스는 기본적으로 CNI(Container Network Interface) 플러그인을 통해 네트워크를 관리합니다. **Calico**, **Flannel**과 같은 CNI 플러그인을 선택하여 클러스터 내에서 네트워크 정책을 적용할 수 있습니다. 특히, **Calico**는 네트워크 정책을 통한 포드 간의 트래픽 제어를 지원하여 보안을 강화할 수 있습니다.
- **네임스페이스(Namespace) 보안**<br/>
  각 애플리케이션을 별도의 네임스페이스에 배치하면 권한을 격리할 수 있습니다. 이를 통해 클러스터 내에서 권한 충돌을 방지하고, 네임스페이스별 접근 제어를 통해 보안을 강화할 수 있습니다.
- **Network Policy**<br/>
  쿠버네티스는 **Network Policy** 리소스를 통해 포드 간 트래픽 제어가 가능합니다. Network Policy는 어떤 포드에서 어떤 포드로 트래픽을 허용할지 지정하며, 특정 네임스페이스나 라벨 기반으로 정책을 설정할 수 있어 세밀한 보안 제어가 가능합니다.
- **RBAC(Role-Based Access Control)**<br/>
  쿠버네티스는 사용자와 서비스 계정의 역할을 설정할 수 있는 RBAC 기능을 제공합니다. 클러스터 리소스에 대한 접근 권한을 Role과 RoleBinding을 통해 관리할 수 있으며, 사용자마다 적절한 권한을 부여하여 보안을 유지할 수 있습니다.
<br/><br/>

## 4. CloudFormation을 이용해 EKS 클러스터 코드형 인프라(IaC) 구축 예시
CloudFormation을 활용하여 쿠버네티스를 관리하기 위해서는 EKS라는 쿠버네티스 완전 관리형 서비스를 사용해 관리하는 것이 좋습니다.<br/>
클러스터와 관련 리소스를 코드로 관리하면 일관된 환경을 유지할 수 있고, 필요 시 자동화된 방식으로 클러스터를 손쉽게 재구성할 수 있습니다.

- **RBAC 및 네트워크 정책 설정**<br/>EKS의 경우 네트워크 정책과 RBAC 설정을 통해 애플리케이션 간의 보안 정책을 적용할 수 있습니다. 이 설정은 클러스터가 생성된 후 `kubectl`을 통해 관리할 수 있습니다.
- **EKS 클러스터 생성**<br/>아래는 CloudFormation 템플릿에서 EKS 클러스터를 정의하는 기본 예시입니다.<br/>
    ```yaml
    Resources:
      MyEKSRole:
        Type: AWS::IAM::Role
        Properties:
          AssumeRolePolicyDocument:
            Version: '2012-10-17'
            Statement:
              - Effect: Allow
                Principal:
                  Service: eks.amazonaws.com
                Action: sts:AssumeRole
          ManagedPolicyArns:
            - arn:aws:iam::aws:policy/AmazonEKSClusterPolicy
    
      MyEKSCluster:
        Type: AWS::EKS::Cluster
        Properties:
          Name: my-eks-cluster
          RoleArn: !GetAtt MyEKSRole.Arn
          ResourcesVpcConfig:
            SubnetIds:
              - !Ref Subnet1
              - !Ref Subnet2
    ```
- **VPC와 서브넷 설정**<br/>VPC와 서브넷을 설정하여 EKS 클러스터가 배포될 네트워크를 정의합니다.<br/>
    ```yaml
    Resources:
      MyVPC:
        Type: AWS::EC2::VPC
        Properties:
          CidrBlock: 10.0.0.0/16
          EnableDnsSupport: true
          EnableDnsHostnames: true
    
      PublicSubnet1:
        Type: AWS::EC2::Subnet
        Properties:
          VpcId: !Ref MyVPC
          CidrBlock: 10.0.1.0/24
          MapPublicIpOnLaunch: true
    
      PublicSubnet2:
        Type: AWS::EC2::Subnet
        Properties:
          VpcId: !Ref MyVPC
          CidrBlock: 10.0.2.0/24
          MapPublicIpOnLaunch: true
    ```
- **Node Group 설정**<br/>EKS 클러스터에 연결할 워커 노드 그룹을 설정합니다. Node Group은 EC2 인스턴스에 의해 구성되며, 클러스터에서 애플리케이션을 실행하는 역할을 합니다.<br/>
    ```yaml
    Resources:
      MyNodeGroup:
        Type: AWS::EKS::Nodegroup
        Properties:
          ClusterName: !Ref MyEKSCluster
          NodeRole: !Ref MyEKSNodeRole
          Subnets:
            - !Ref PublicSubnet1
            - !Ref PublicSubnet2
          ScalingConfig:
            MinSize: 1
            MaxSize: 3
            DesiredSize: 2
    ```
  

## 5. EC2에서 별도로 쿠버네티스를 운용할 시 Teraform을 사용해 코드형 인프라(IaC) 구축 예시
EC2에서 쿠버네티스를 직접 설치하고 운영할 때, **Terraform**을 사용하면 설정을 자동화하고 효율적으로 관리할 수 있습니다.<br/>
Terraform은 **다양한 클라우드 리소스를 코드로 관리**할 수 있도록 지원하며, 특히 EC2 인스턴스 생성, 보안 그룹 설정, IAM 역할 부여와 같은 AWS 리소스 관리를 쉽게 자동화할 수 있습니다.

### CloudFormation을 사용하지 않는 이유
EC2 인스턴스에 직접 쿠버네티스를 설치하고 관리하려면 CloudFormation보다는 다른 관리 방식을 사용하는 것이 일반적입니다.<br/>
CloudFormation은 주로 AWS 리소스의 설정 및 배포에 특화되어 있습니다. 하지만 쿠버네티스를 EC2에 직접 설치하려면 kubeadm이나 kops 등의 도구를 사용하여 클러스터를 구축해야 하는데, 이 과정은 상당히 복잡하며 CloudFormation으로 구현하기에는 많은 커스텀이 필요해 권장되지 않습니다.

### EKS 대신 EC2에서 쿠버네티스를 사용할 때 권장되는 대안 방법
1. **kubeadm 사용**: `kubeadm`은 쿠버네티스를 설치하고 클러스터를 초기화하는 데 사용되는 대표적인 CLI 도구입니다. EC2 인스턴스에 `kubeadm`을 이용해 쿠버네티스를 설치하면, 기본적으로 마스터 노드와 워커 노드를 설정할 수 있고, AWS CLI로 필요한 IAM 역할과 보안 그룹 등을 설정해 운영할 수 있습니다.
2. **Terraform**: CloudFormation 대신 Terraform을 사용하여 쿠버네티스 인프라를 정의하는 것도 좋은 대안입니다. Terraform은 AWS 외에도 다양한 클라우드 프로바이더와 호환되며, kubeadm과 연동하여 초기 설정을 자동화하는 데 유리합니다.
3. **kops 사용**: `kops`는 쿠버네티스 클러스터를 자동으로 설치하고 설정할 수 있는 도구로, 특히 EC2 인프라에 쿠버네티스를 설정하는 데 최적화되어 있습니다. `kops`는 S3 버킷을 통해 클러스터 상태를 관리하고, 추가적인 노드 확장이나 설정 변경도 쉽게 관리할 수 있습니다.

### Terraform으로 EC2에서 쿠버네티스 설정하는 단계
1. **EC2 인스턴스 생성**<br/>
   Terraform을 통해 필요한 인스턴스를 정의하고 프로비저닝할 수 있습니다. Terraform 코드에서 인스턴스 타입, AMI, 보안 그룹 등을 설정하여 쿠버네티스를 설치할 마스터 및 워커 노드를 생성합니다.<br/>
    ```hcl
    hcl
    코드 복사
    provider "aws" {
      region = "us-west-2"
    }
    
    resource "aws_instance" "k8s_master" {
      ami           = "ami-0c55b159cbfafe1f0" # Ubuntu AMI
      instance_type = "t2.medium"
      key_name      = "my-key"
    
      tags = {
        Name = "K8sMaster"
      }
    }
    
    resource "aws_instance" "k8s_worker" {
      count         = 2
      ami           = "ami-0c55b159cbfafe1f0"
      instance_type = "t2.medium"
      key_name      = "my-key"
    
      tags = {
        Name = "K8sWorker-${count.index}"
      }
    }
    
    ```

2. **보안 그룹 및 IAM 역할 설정**<br/>
   보안 그룹을 설정하여 쿠버네티스가 사용하는 포트를 열고, IAM 역할을 추가해 EC2 인스턴스에서 AWS 리소스에 접근할 수 있도록 권한을 부여합니다.<br/>
    ```hcl
    resource "aws_security_group" "k8s_sg" {
      name = "k8s-security-group"
    
      ingress {
        from_port   = 6443
        to_port     = 6443
        protocol    = "tcp"
        cidr_blocks = ["0.0.0.0/0"]
      }
    
      egress {
        from_port   = 0
        to_port     = 0
        protocol    = "-1"
        cidr_blocks = ["0.0.0.0/0"]
      }
    }
    ```

3. **kubeadm 설치 및 초기화 스크립트 실행**<br/>
   Terraform에서는 EC2 인스턴스가 프로비저닝되면, `user_data`를 사용하여 초기화 스크립트를 실행할 수 있습니다. 이를 통해 인스턴스 시작 시 자동으로 `kubeadm` 설치 및 초기화를 진행합니다.<br/>
    ```hcl
    resource "aws_instance" "k8s_master" {
      # other properties...
    
      user_data = <<-EOF
        #!/bin/bash
        sudo apt-get update -y
        sudo apt-get install -y docker.io apt-transport-https curl
        curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
        sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
        sudo apt-get install -y kubelet kubeadm kubectl
        sudo kubeadm init --pod-network-cidr=10.244.0.0/16
        EOF
    }
    ```

4. **Terraform에서 출력 변수 사용하기**<br/>
   초기화 과정이 끝나면, Terraform을 통해 마스터 노드의 IP 주소와 연결 정보를 출력해 워커 노드에서 마스터 노드에 쉽게 연결할 수 있도록 합니다.<br/>
    ```hcl
    output "master_ip" {
      value = aws_instance.k8s_master.public_ip
    }
    ```

### Terraform을 통한 EC2에서 쿠버네티스 관리 시 모니터링 설정 방법
EC2 기반의 쿠버네티스 환경에서는 쿠버네티스 클러스터와 각 노드, 애플리케이션 상태를 모니터링하는 것이 매우 중요합니다. EC2 인스턴스에서 쿠버네티스를 직접 관리할 때는 주로 오픈소스 모니터링 도구를 사용해 상태를 추적하고 성능을 관리합니다.
다음은 쿠버네티스 모니터링에 자주 사용되는 대표적인 도구들과 Terraform을 통해 모니터링을 자동화할 수 있는 방법들입니다.

#### 1. Prometheus & Grafana: 클러스터와 애플리케이션 모니터링
- **Prometheus**는 쿠버네티스 환경에서 가장 널리 쓰이는 모니터링 솔루션으로, 쿠버네티스 클러스터와 포드(Pod) 내 컨테이너 상태를 실시간으로 수집합니다.
- **Grafana**는 Prometheus에서 수집한 데이터를 시각화하여 대시보드로 제공하며, 주요 지표(CPU, 메모리, 네트워크 트래픽 등)를 한눈에 파악할 수 있습니다.

**[설치 및 설정 방법]**<br/>
Terraform을 통해 Prometheus와 Grafana를 설정하려면, EC2 인스턴스의 `user_data` 스크립트에 Prometheus와 Grafana 설치 명령을 포함하거나 Helm Chart를 사용하여 쿠버네티스 클러스터에 배포할 수 있습니다.

```hcl
resource "aws_instance" "monitoring_instance" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.medium"
  key_name      = "my-key"

  user_data = <<-EOF
    #!/bin/bash
    # Docker 설치
    sudo apt-get update -y
    sudo apt-get install -y docker.io

    # Prometheus와 Grafana 설치 (Docker 컨테이너로 실행)
    docker run -d --name prometheus -p 9090:9090 prom/prometheus
    docker run -d --name grafana -p 3000:3000 grafana/grafana

    # Prometheus와 Grafana의 포트 열기
    sudo ufw allow 9090/tcp
    sudo ufw allow 3000/tcp
  EOF

  tags = {
    Name = "MonitoringInstance"
  }
}

```

**[Terraform 출력 변수 설정]**<br/>
설치 후에는 Grafana 대시보드나 Prometheus 대시보드의 접속 정보를 출력 변수로 설정해 접속 정보를 손쉽게 확인할 수 있습니다.

```hcl
output "prometheus_url" {
  value = "http://${aws_instance.monitoring_instance.public_ip}:9090"
}

output "grafana_url" {
  value = "http://${aws_instance.monitoring_instance.public_ip}:3000"
}
```

### 2. Fluentd & Elasticsearch: 로그 관리 및 분석
- **Fluentd**는 각 포드의 로그 데이터를 수집하여 중앙의 로그 관리 시스템으로 전송합니다. EC2 기반의 쿠버네티스 환경에서는 Fluentd가 애플리케이션 로그를 **Elasticsearch**로 전송해 관리하는 경우가 많습니다.
- **Elasticsearch**는 대규모 로그 데이터를 저장하고 검색할 수 있는 분산형 검색 엔진입니다. Fluentd에서 전송된 로그 데이터를 시각화할 때 **Kibana**를 함께 사용하여 로그 분석을 더욱 효율적으로 진행할 수 있습니다.

**[설치 및 설정 방법]**<br/>
Terraform을 이용해 Elasticsearch 인스턴스와 Fluentd를 설치하고, Fluentd가 각 포드의 로그 데이터를 Elasticsearch로 전송하도록 설정할 수 있습니다.<br/>
이렇게 설정하면 Fluentd가 각 포드의 로그 데이터를 수집해 Elasticsearch로 전송하고, Kibana를 통해 이를 시각화할 수 있습니다.

```hcl
resource "aws_instance" "logging_instance" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.medium"
  key_name      = "my-key"

  user_data = <<-EOF
    #!/bin/bash
    # Docker 설치
    sudo apt-get update -y
    sudo apt-get install -y docker.io

    # Elasticsearch 설치
    docker run -d --name elasticsearch -p 9200:9200 -p 9300:9300 elasticsearch:7.10.1

    # Fluentd 설치 및 설정
    docker run -d --name fluentd -p 24224:24224 -v /var/log:/fluentd/log fluent/fluentd

    # Elasticsearch와 Fluentd의 포트 열기
    sudo ufw allow 9200/tcp
    sudo ufw allow 9300/tcp
    sudo ufw allow 24224/tcp
  EOF

  tags = {
    Name = "LoggingInstance"
  }
}
```

### 3. Jaeger: 트랜잭션 추적 및 장애 진단
- **Jaeger**는 마이크로서비스 환경에서 분산 트랜잭션을 추적하고, 서비스 간의 호출 관계와 병목 지점을 시각적으로 확인하는 데 사용됩니다. Jaeger는 애플리케이션 간의 호출 데이터를 추적하여 성능 병목을 분석하고, 장애 원인을 진단하는 데 큰 도움을 줍니다.

**[설치 및 설정 방법]**<br/>
Jaeger를 Terraform을 통해 설치하려면 Jaeger 서버를 EC2 인스턴스에 띄우고, 각 애플리케이션이 트랜잭션 정보를 Jaeger에 전송하도록 설정합니다.<br/>
이 설정으로 Jaeger가 EC2 인스턴스에 설치되고, Jaeger의 대시보드에서 애플리케이션의 분산 트랜잭션을 추적할 수 있습니다.

```hcl
resource "aws_instance" "tracing_instance" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.medium"
  key_name      = "my-key"

  user_data = <<-EOF
    #!/bin/bash
    # Docker 설치
    sudo apt-get update -y
    sudo apt-get install -y docker.io

    # Jaeger 설치
    docker run -d --name jaeger -e COLLECTOR_ZIPKIN_HTTP_PORT=9411 -p 5775:5775/udp -p 6831:6831/udp -p 6832:6832/udp -p 5778:5778 -p 16686:16686 -p 14268:14268 -p 14250:14250 -p 9411:9411 jaegertracing/all-in-one:1.21

    # Jaeger의 포트 열기
    sudo ufw allow 16686/tcp
    sudo ufw allow 14268/tcp
    sudo ufw allow 9411/tcp
  EOF

  tags = {
    Name = "TracingInstance"
  }
}
```


# Outro
이번 글에서는 Kubernetes 환경을 구축하기 위한 필수적인 설정 방법에 대해 다루었습니다.<br/>
**인프라 구조**, **관리 및 모니터링 도구 설정**, **네트워크와 보안 구성**, 그리고 **CloudFormation** 및 **Terraform**을 이용한 코드형 인프라 구현까지 폭넓게 설명드렸습니다.
특히 EC2에서 쿠버네티스를 직접 관리할 경우, Prometheus, Grafana, Fluentd, Jaeger 등의 오픈소스 도구를 활용한 모니터링 및 로그 관리 방법에 대해서도 상세히 소개했는데요.<br/>
여러분들이 이를 통해 쿠버네티스 인프라의 가시성과 안정성을 높이는 방법을 이해하는 데 도움이 되었기를 바랍니다.

다음 글에서는 쿠버네티스로 오케스트레이션을 구현하는 상세한 방법에 대해 다루겠습니다.

긴 글 읽어주셔서 감사합니다. 질문이 있으시면 언제든지 댓글로 남겨주세요!
