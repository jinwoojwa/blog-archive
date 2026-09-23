---
series: 그림으로 이해하는 AWS 구조와 기술
title: DevOps·컨테이너·서버리스(ECS, EKS, Lambda 등)
---

# DevOps·컨테이너·서버리스(ECS, EKS, Lambda 등)

> 『그림으로 이해하는 AWS 구조와 기술』를 읽고 정리한 내용

## DevOps

### DevOps란?

DevOps(Development + Operations)는 개발과 운영이 협력하여 **더 빠르고 안정적으로 서비스를 개발하고 배포하는 문화 및 방법론**

목표

- 개발 속도 향상
- 배포 자동화
- 서비스 안정성 향상
- 운영 효율성 증대

---

### CI (Continuous Integration)

지속적 통합(Continuous Integration)

- 변경된 코드를 저장소에 병합할 때마다 빌드와 테스트를 자동 수행
- 코드 품질을 빠르게 검증
- 문제를 조기에 발견 가능

대표적인 CI 도구

- GitHub Actions
- Jenkins
- GitLab CI
- AWS CodeBuild

---

### CD (Continuous Delivery / Continuous Deployment)

지속적 전달(Continuous Delivery) 또는 지속적 배포(Continuous Deployment)

- 테스트를 통과한 코드를 운영 환경까지 자동 배포
- 사람이 직접 서버에 접속하여 배포할 필요 없음

CI는 **코드 품질 검증**, CD는 **배포 자동화**에 초점

대표적인 AWS 서비스

- AWS CodeDeploy
- AWS CodePipeline

---

### 마이크로서비스(Microservice)

하나의 거대한 애플리케이션(Monolith)이 아닌 여러 개의 작은 서비스로 시스템을 구성하는 방식

특징

- 서비스별 독립 배포
- 장애가 전체 시스템으로 확산되지 않음
- 서비스마다 서로 다른 기술 스택 선택 가능
- API를 통해 서비스 간 통신

---

### Infrastructure as Code (IaC)

인프라를 코드로 정의하고 관리하는 방식

예를 들어

- EC2 생성
- VPC 생성
- RDS 생성

등을 콘솔에서 직접 생성하는 대신 코드로 관리

대표적인 도구

- AWS CloudFormation
- Terraform

---

### 모니터링과 로깅

서비스 상태를 지속적으로 감시하고 기록하는 과정

대표적인 AWS 서비스

- Amazon CloudWatch
- AWS CloudTrail

이를 통해

- 장애를 빠르게 탐지
- 시스템 성능 분석
- 운영 자동화

가능

---

## 컨테이너(Container)

### 컨테이너란?

프로그램과 실행에 필요한 환경을 하나로 묶어 실행하는 기술

대표적인 컨테이너 플랫폼

- Docker

장점

- 빠른 실행 속도
- 가벼운 실행 환경
- 동일한 실행 환경 보장

---

### 가상 머신(VM)과의 차이

| 항목      | 가상 머신(VM) | 컨테이너 |
| --------- | ------------- | -------- |
| 운영체제  | 포함          | 공유     |
| 실행 속도 | 느림          | 빠름     |
| 크기      | 큼            | 작음     |
| 자원 사용 | 많음          | 적음     |

가상 머신은 운영체제까지 포함하지만,

컨테이너는 **호스트 OS의 커널을 공유**하므로 훨씬 가벼운 구조

---

### Docker Image와 Container

EC2가 AMI에서 생성되는 것처럼 Docker는 **이미지(Image)** 로부터 **컨테이너(Container)** 생성

```text
Docker Image

↓

Docker Container
```

이미지는 레지스트리(Registry)에 저장하여 관리

대표적인 레지스트리

- Docker Hub
- Amazon ECR(Elastic Container Registry)

---

## 컨테이너 오케스트레이션

컨테이너 수가 많아질수록 사람이 직접 관리하기 어려움

이를 자동으로 관리하는 기술을 **컨테이너 오케스트레이션(Container Orchestration)** 이라고 함

대표적인 오케스트레이션 도구

- Kubernetes(k8s)

---

### Pod

쿠버네티스에서 컨테이너를 실행하는 최소 단위

하나 이상의

- 컨테이너
- 네트워크
- 볼륨

을 하나의 Pod로 관리

---

### Cluster

쿠버네티스 전체 시스템

구성

- Control Plane
- Worker Node

```text
Cluster

├── Control Plane
│
├── Worker Node
│     ├── Pod
│     └── Pod
│
└── Worker Node
      ├── Pod
      └── Pod
```

- Worker Node에서 실제 컨테이너 실행
- Control Plane에서 전체 클러스터 관리

---

### Self-Healing

쿠버네티스의 대표 기능

예를 들어

```text
Pod 종료

↓

자동 감지

↓

새로운 Pod 생성
```

사람이 개입하지 않아도 원하는 상태 유지

추가 기능

- Auto Scaling
- Rolling Update
- Service Discovery

지원

---

## Amazon ECS와 Amazon EKS

AWS에서 제공하는 관리형 컨테이너 오케스트레이션 서비스

### Amazon ECS

- AWS가 자체 개발한 컨테이너 오케스트레이션 서비스
- 비교적 간단한 설정
- AWS 서비스와 높은 통합성 제공

### Amazon EKS

- 관리형 Kubernetes 서비스
- Kubernetes 표준 환경 제공
- 기존 Kubernetes 경험 그대로 활용 가능

### ECS와 EKS 비교

| 항목            | ECS      | EKS        |
| --------------- | -------- | ---------- |
| 오케스트레이션  | AWS 자체 | Kubernetes |
| 학습 난이도     | 쉬움     | 어려움     |
| Kubernetes 호환 | X        | O          |
| AWS 의존성      | 높음     | 낮음       |

---

### AWS Fargate

컨테이너를 실행하기 위한 **서버리스 실행 환경**

ECS와 EKS는 오케스트레이션만 담당하며 실제 컨테이너는

- EC2
- AWS Fargate

중 하나에서 실행

Fargate 사용 시

- EC2 관리 불필요
- 서버 프로비저닝 불필요
- 사용한 만큼만 비용 발생

---

## 서버리스(Serverless)

### 서버리스란?

서버가 없는 것이 아니라 **서버를 직접 관리하지 않는 컴퓨팅 모델**

운영체제 관리

- 서버 관리
- 패치
- 확장

등을 AWS에서 수행

---

## AWS Lambda

이벤트가 발생했을 때 코드를 실행하는 서버리스 컴퓨팅 서비스

특징

- 서버 관리 불필요
- 실행 시간 기준 과금
- 자동 확장
- 이벤트 기반 실행

대표적인 트리거

- S3 업로드
- API Gateway 호출
- EventBridge
- DynamoDB 이벤트
- SQS 메시지

예시

```text
S3 파일 업로드

↓

Lambda 실행

↓

이미지 리사이징

↓

S3 저장
```

---

## Amazon API Gateway

API를 생성하고 관리하는 서비스

지원 API

- REST API
- HTTP API
- WebSocket API

주요 기능

- 인증(Authentication)
- 인가(Authorization)
- 요청 제한(Rate Limiting)
- 캐싱(Cache)
- Lambda 연동

### REST API

- 가장 다양한 기능 제공
- RESTful API 구축에 적합

### HTTP API

- REST API보다 단순한 구조
- 낮은 비용
- 빠른 성능

### WebSocket API

- 클라이언트와 서버가 지속적으로 연결
- 양방향 통신 지원

대표적인 사용 사례

- 채팅
- 실시간 게임
- 실시간 알림

---

## 서비스 연결 예시

### 컨테이너 기반

```text
GitHub

↓

CI/CD

↓

Amazon ECR

↓

Amazon ECS / Amazon EKS

↓

AWS Fargate 또는 EC2
```

### 서버리스 기반

```text
Client

↓

API Gateway

↓

Lambda

↓

DynamoDB
```

---

## 서비스 역할 비교

| 서비스                         | 역할                             |
| ------------------------------ | -------------------------------- |
| **Docker**                     | 컨테이너 실행 환경               |
| **Amazon ECR**                 | 컨테이너 이미지 저장소           |
| **Amazon ECS**                 | AWS 자체 컨테이너 오케스트레이션 |
| **Amazon EKS**                 | 관리형 Kubernetes                |
| **AWS Fargate**                | 서버리스 컨테이너 실행 환경      |
| **AWS Lambda**                 | 이벤트 기반 서버리스 컴퓨팅      |
| **Amazon API Gateway**         | API 생성 및 관리                 |
| **CloudFormation / Terraform** | Infrastructure as Code(IaC)      |

---

## 정리

- DevOps는 개발과 운영을 자동화하여 빠르고 안정적인 서비스 제공을 목표로 하는 문화 및 방법론
- Docker는 컨테이너 실행 환경, Kubernetes는 컨테이너를 자동 관리하는 오케스트레이션 도구
- ECS는 AWS 자체 컨테이너 오케스트레이션 서비스, EKS는 관리형 Kubernetes 서비스
- Fargate는 서버 관리 없이 컨테이너를 실행하는 서버리스 실행 환경
- Lambda는 이벤트 기반 서버리스 컴퓨팅 서비스이며, API Gateway와 함께 서버리스 아키텍처 구성
