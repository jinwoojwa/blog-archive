---
series: 그림으로 이해하는 AWS 구조와 기술
title: AWS 기초와 IAM
---

# AWS 기초와 IAM

> 『그림으로 이해하는 AWS 구조와 기술』를 읽고 정리한 내용

## AWS란?

### AWS (Amazon Web Services)

- 아마존이 제공하는 클라우드 컴퓨팅 서비스(= 클라우드)
- 시스템 구축에 필요한 다양한 자원을 인터넷을 통해 빌려 사용할 수 있음
- 다양한 서비스를 조합하여 애플리케이션 인프라를 구축 가능
- 300개 이상의 서비스를 제공하며 필요에 따라 자유롭게 조합 가능
- `IaaS`, `PaaS`, `Serverless(FaaS)` 등 다양한 클라우드 모델 지원

### 대표적인 클라우드 서비스

| 클라우드  | 서비스                     |
| --------- | -------------------------- |
| Amazon    | AWS                        |
| Microsoft | Azure                      |
| Google    | Google Cloud Platform(GCP) |

---

## AWS의 대표 서비스

AWS는 300개 이상의 서비스를 제공하지만 자주 사용되는 서비스는 크게 다음과 같이 구분할 수 있음

### 컴퓨팅 & 컨테이너

| 서비스                                      | 설명                                                                 |
| ------------------------------------------- | -------------------------------------------------------------------- |
| **Amazon EC2 (Elastic Compute Cloud)**      | 클라우드에서 가상 서버를 제공하는 컴퓨팅 서비스                      |
| **Amazon ECS (Elastic Container Service)**  | Docker 컨테이너를 실행·관리하는 AWS의 컨테이너 오케스트레이션 서비스 |
| **Amazon EKS (Elastic Kubernetes Service)** | Kubernetes를 손쉽게 운영할 수 있는 관리형 Kubernetes 서비스          |
| **AWS Lambda**                              | 서버 관리 없이 코드를 실행하는 서버리스(Serverless) 컴퓨팅 서비스    |
| **AWS Batch**                               | 대규모 배치(Batch) 작업을 자동으로 실행하고 관리하는 서비스          |

### 스토리지 & 데이터베이스

| 서비스                                       | 설명                                                                  |
| -------------------------------------------- | --------------------------------------------------------------------- |
| **Amazon S3 (Simple Storage Service)**       | 객체(Object) 스토리지 서비스로 파일 저장 및 정적 웹사이트 호스팅 지원 |
| **Amazon EBS (Elastic Block Store)**         | EC2에 연결하여 사용하는 영구 블록 스토리지 서비스                     |
| **Amazon RDS (Relational Database Service)** | MySQL, PostgreSQL, MariaDB 등 관계형 데이터베이스를 관리형으로 제공   |
| **Amazon Aurora**                            | MySQL·PostgreSQL과 호환되는 고성능 관리형 관계형 데이터베이스         |
| **Amazon DynamoDB**                          | 완전관리형 NoSQL 데이터베이스 서비스                                  |
| **Amazon ElastiCache**                       | Redis, Memcached 기반의 관리형 인메모리 캐시 서비스                   |

### 네트워크 & 트래픽 관리

| 서비스                                 | 설명                                                  |
| -------------------------------------- | ----------------------------------------------------- |
| **Amazon VPC (Virtual Private Cloud)** | AWS에서 격리된 가상 네트워크를 구성하는 서비스        |
| **Elastic Load Balancing (ELB)**       | 여러 서버로 트래픽을 자동 분산하는 로드 밸런서 서비스 |
| **Amazon Route 53**                    | 도메인과 DNS(Domain Name System)를 관리하는 서비스    |

---

## IAM (Identity and Access Management)

### IAM이란?

- AWS의 인증(Authentication)과 인가(Authorization)를 관리하는 서비스
- 사용자 및 서비스의 접근 권한을 세밀하게 제어할 수 있음
- 최소 권한 원칙(Principle of Least Privilege)을 적용하는 것이 권장됨

---

## AWS 계정 종류

AWS에서 사용하는 계정은 크게 두 가지이다.

| 계정           | 설명                                |
| -------------- | ----------------------------------- |
| **Root 계정**  | AWS 가입 시 생성되는 최고 권한 계정 |
| **IAM 사용자** | 실제 사용자가 사용하는 계정         |

> 일반적인 운영에서는 Root 계정보다는 IAM 사용자를 생성하여 사용하는 것이 권장됨

---

## IAM 사용자 (IAM User)

- 실제 사용자 또는 애플리케이션을 위한 계정
- 여러 명의 사용자를 생성 가능
- 사용자별로 권한을 다르게 부여할 수 있음
- 정책(Policy) 또는 그룹(Group), 역할(Role)을 통해 권한을 부여

---

## IAM 그룹 (IAM Group)

- 여러 IAM 사용자를 하나의 그룹으로 관리
- 그룹에 정책을 연결하면 모든 사용자에게 동일한 권한이 적용
- 한 사용자는 여러 그룹에 속할 수 있음

---

## IAM 역할 (IAM Role)

- 사용자 또는 AWS 서비스가 일시적으로 사용하는 권한
- 로그인 계정이 아닌 '역할' 개념
- 주로 EC2, Lambda 등이 다른 AWS 서비스를 사용할 때 활용

---

## IAM 정책 (IAM Policy)

- 사용자, 그룹, 역할에 권한을 부여하는 JSON 문서
- Allow 또는 Deny를 정의

### 정책 종류

| 종류                  | 설명                                |
| --------------------- | ----------------------------------- |
| Identity-based Policy | 사용자, 그룹, 역할의 권한 정의      |
| Resource-based Policy | 특정 리소스에 접근 가능한 주체 정의 |

### 정책 구성 요소

| 요소      | 설명                 |
| --------- | -------------------- |
| Principal | 권한을 사용하는 주체 |
| Action    | 수행 가능한 작업     |
| Resource  | 대상 리소스          |
| Effect    | Allow / Deny         |
| Condition | 권한 적용 조건       |

---

## AWS Managed Service

- AWS가 인프라 운영을 대신 수행하는 서비스
- 자동 백업
- 자동 업데이트
- 모니터링
- 보안 관리

대표적인 서비스

- Amazon S3
- Amazon RDS

---

## Amazon CloudWatch

- AWS 리소스의 모니터링 서비스
- 메트릭과 로그를 수집
- 임계값 초과 시 다양한 액션 수행 가능

예시

- CPU 사용률
- 메모리 사용량
- 네트워크 트래픽
- 디스크 읽기/쓰기

CloudWatch Alarm을 이용하면

- 이메일 알림
- EC2 시작/중지
- Auto Scaling
- Lambda 실행

등을 자동으로 수행할 수 있다.

> 누가 언제 어떤 리소스에 접근했는지 확인하려면 **AWS CloudTrail**을 함께 사용

---

## 정리

- AWS는 다양한 클라우드 서비스를 제공하는 플랫폼
- IAM은 AWS 리소스에 대한 인증과 인가를 담당하며 최소 권한 원칙을 적용하는 것이 중요
- 권한은 사용자(User), 그룹(Group), 역할(Role)에 정책(Policy)을 연결하여 관리
- CloudWatch는 리소스 상태를 모니터링하고, CloudTrail은 AWS 리소스의 작업 이력을 기록함
