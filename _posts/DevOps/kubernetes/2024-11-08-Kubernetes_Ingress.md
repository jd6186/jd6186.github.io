---
layout: post
title: "[Kubernetes] Kubernetes Ingress란?"
tags: [Kubernetes, Ingress, DevOps]
---

# Intro
안녕하세요, Noah입니다.<br/>
이번 글에서는 Ingress가 무엇인지에 대해 알아보고, Kubernetes에서 Ingress를 사용하는 방법을 알아보겠습니다.<br/>
Kubernetes에서 도메인별 요청 라우팅 및 특정 노드로 트래픽을 보내는 로드 밸런싱을 구현하려면 **Ingress**와 **Node Affinity** 기능을 조합해서 사용할 수 있습니다.

그럼 시작해 보겠습니다.
<br/><br/><br/><br/>

# 목차
<br/><br/><br/><br/>

## **Ingress란 무엇인가요?**
Ingress Controller는 일반적으로 로드 밸런서 역할로 Ingress 규칙을 이행합니다.<br/>
또한, Ingress는 클러스터 외부에서 클러스터 내부의 서비스 로 HTTP 및 HTTPS 경로를 노출합니다.<br/>
트래픽 라우팅은 Ingress 리소스에 정의된 규칙에 의해 제어됩니다.<br/>
Ingress는 서비스에 외부에서 도달 가능한 URL을 제공하고, 트래픽을 로드 밸런싱하고, SSL/TLS를 종료하고, 이름 기반 가상 호스팅을 제공하도록 구성될 수 있습니다.<br/>
Ingress는 임의의 포트나 프로토콜을 노출하지 않습니다. HTTP 및 HTTPS 이외의 서비스를 인터넷에 노출하는 것은 일반적으로 `Service.Type=NodePort` 또는 `Service.Type=LoadBalancer` 유형의 서비스를 사용합니다 .<br/>

다음은 Ingress가 모든 트래픽을 하나의 서비스로 보내는 간단한 예<br/>
https://kubernetes.io/docs/images/ingress.svg

---

## **1. 구현 전략**
1. **Ingress 리소스**:
    - **`test.com`** 도메인에 대한 경로별(**`/game`**, **`/user`**) 라우팅을 설정합니다.
    - 각각의 경로는 특정 서비스(Service)로 트래픽을 전달합니다.
2. **Node Affinity**:
    - 특정 서비스가 **Node A** 또는 **Node B**에서만 실행되도록 **Pod 스케줄링**을 설정합니다.
3. **Ingress Controller**:
    - Ingress 규칙을 처리하기 위해 NGINX Ingress Controller 또는 Traefik과 같은 컨트롤러를 설치해야 합니다.

---

## **2. 설정 단계**
### **Step 1: Ingress Controller 설치**
먼저 클러스터에 Ingress Controller를 설치해야 합니다. 여기서는 NGINX Ingress Controller를 예로 들겠습니다.

#### **NGINX Ingress 설치 (Helm 사용)**
```bash
# helm 설치
brew install helm

# Ingress Controller 설치
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install nginx-ingress ingress-nginx/ingress-nginx

# 상태확인
helm status nginx-ingress
```

---

### **Step 2: 서비스 및 Deployment 작성**
#### **`/game`과 `/user` 경로에 맞는 서비스 정의**
```yaml
# game-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: game-service
spec:
  selector:
    app: game-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```

```yaml
# user-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: user-service
spec:
  selector:
    app: user-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```

#### **각 서비스의 Deployment 작성**
Node Affinity를 통해 특정 노드에서 실행되도록 설정합니다.

```yaml
# game-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: game-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: game-app
  template:
    metadata:
      labels:
        app: game-app
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: kubernetes.io/hostname
                operator: In
                values:
                - node-a  # Node A의 이름
      containers:
      - name: game-app
        image: your-game-app-image
        ports:
        - containerPort: 8080
```

```yaml
# user-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: user-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: user-app
  template:
    metadata:
      labels:
        app: user-app
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: kubernetes.io/hostname
                operator: In
                values:
                - node-b  # Node B의 이름
      containers:
      - name: user-app
        image: your-user-app-image
        ports:
        - containerPort: 8080
```

---

### **Step 3: Ingress 리소스 작성**
Ingress 리소스를 사용하여 요청을 특정 서비스로 라우팅합니다.

https://kubernetes.io/docs/images/ingressFanOut.svg

```yaml
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: test-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: test.com
    http:
      paths:
      - path: /game
        pathType: Prefix
        backend:
          service:
            name: game-service
            port:
              number: 80
      - path: /user
        pathType: Prefix
        backend:
          service:
            name: user-service
            port:
              number: 80
```

---

## **3. 테스트**
1. **`test.com/game`**으로 요청을 보내면 **Node A**에 배포된 **`game-app`**으로 전달됩니다.
2. **`test.com/user`**로 요청을 보내면 **Node B**에 배포된 **`user-app`**으로 전달됩니다.

---

## **4. 문서 관리**
Kubernetes YAML 파일을 효율적으로 관리하기 위해 아래 방식을 추천합니다

1. **디렉토리 구조**<br/>
    ```jsx
    k8s/
    ├── ingress/
    │   └── ingress.yaml
    ├── services/
    │   ├── game-service.yaml
    │   └── user-service.yaml
    ├── deployments/
    │   ├── game-deployment.yaml
    │   └── user-deployment.yaml
    ├── configs/
    └── README.md
    
    ```

2. **README 작성** 디렉토리 구조와 각 파일의 역할을 설명하는 **`README.md`** 파일을 작성합니다.
3. **Git 관리**
    - 모든 YAML 파일을 Git에 저장하고, 변경 사항을 기록하세요.
    - 브랜치를 사용해 실수로 프로덕션 환경을 손상시키지 않도록 관리하세요.

---

## 복잡한 구조의 ingress yaml 작성

### **1. Host 기반 Ingress 리소스 작성**

호스트는 정확한 일치(예: " `foo.bar.com`") 또는 와일드카드(예: " `*.foo.com`")일 수 있습니다. 정확한 일치는 HTTP `host`헤더가 필드와 일치 해야 합니다 `host`. 와일드카드 일치는 HTTP `host`헤더가 와일드카드 규칙의 접미사와 동일해야 합니다.

| Host | Host Header | Match 여부 |
| --- | --- | --- |
| `*.foo.com` | `bar.foo.com` | 공유 접미사를 기반으로 한 일치 |
| `*.foo.com` | `baz.bar.foo.com` | *일치하지 않음*, 와일드카드는 단일 DNS 레이블만 포함합니다. |
| `*.foo.com` | `foo.com` | *일치하지 않음*, 와일드카드는 단일 DNS 레이블만 포함합니다. |

https://kubernetes.io/docs/images/ingressNameBased.svg

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-wildcard-host
spec:
  rules:
  - host: "foo.bar.com"
    http:
      paths:
      - pathType: Prefix
        path: "/bar"
        backend:
          service:
            name: service1
            port:
              number: 80
  - host: "*.foo.com"
    http:
      paths:
      - pathType: Prefix
        path: "/foo"
        backend:
          service:
            name: service2
            port:
              number: 80
```

---

### **2. Ingress 리소스 내 기본 요청 수신 Path설정**
규칙에 정의된 호스트 없이 Ingress 리소스를 생성하면 이름 기반 가상 호스트가 필요하지 않고도 Ingress 컨트롤러의 IP 주소로 들어오는 모든 웹 트래픽을 매칭할 수 있습니다.<br/>
예를 들어, 다음 Ingress는 요청된 트래픽 중 Host가 `first.bar.com`, `second.bar.com`인 것들은 그에 맞는 서비스로 라우팅하고, 요청 호스트 헤더가 일치하지 않는 모든 트래픽을 service3로 라우팅 합니다.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: name-virtual-host-ingress-no-third-host
spec:
  rules:
  - host: first.bar.com
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: service1
            port:
              number: 80
  - host: second.bar.com
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: service2
            port:
              number: 80
  - http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: service3
            port:
              number: 80
```

---

# Outro
이상으로 Kubernetes Ingress에 대해 알아보았습니다.<br/>
Ingress를 사용하면 클러스터 외부에서 클러스터 내부의 서비스로 HTTP 및 HTTPS 경로를 노출할 수 있습니다.<br/>
이렇게 설정하면 원하는 대로 트래픽을 라우팅하고 노드별로 서비스 배포를 제어할 수 있습니다. <br/>
추가적인 도움이나 세부 사항이 필요하면 언제든 문의해주세요! 😊

긴 글 읽어주셔서 감사합니다! 🙏