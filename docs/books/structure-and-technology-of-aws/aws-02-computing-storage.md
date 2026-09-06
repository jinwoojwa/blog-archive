---
series: 그림으로 이해하는 AWS 구조와 기술
title: EC2·S3·RDS 등 컴퓨팅과 스토리지
---

# EC2·S3·RDS 등 컴퓨팅과 스토리지

> 『그림으로 이해하는 AWS 구조와 기술』를 읽고 정리한 내용

## 컴퓨팅과 스토리지란?

애플리케이션을 운영하기 위해서는 크게 다음과 같은 요소가 필요

- **컴퓨팅(Compute)** : 애플리케이션을 실행하는 서버
- **스토리지(Storage)** : 데이터를 저장하는 공간
- **데이터베이스(Database)** : 데이터를 구조화하여 저장하고 조회하는 시스템

AWS는 각각을 독립적인 서비스로 제공하며 필요에 따라 조합하여 사용할 수 있음

---

## Amazon EC2 (Elastic Compute Cloud)

### EC2란?

- 클라우드에서 가상 서버를 제공하는 컴퓨팅 서비스
- 필요한 CPU, 메모리, 운영체제 등을 선택하여 서버를 생성할 수 있음
- 생성된 각각의 서버를 **인스턴스(Instance)** 라고 함
- 일반적으로 `AMI`, `EBS`, `VPC`, `Security Group`, `Elastic IP` 등을 함께 사용
- SSH를 통해 서버에 접속하여 작업을 수행

### EC2 인스턴스 생성 시 설정 항목

| 항목              | 설명                                        |
| ----------------- | ------------------------------------------- |
| **AMI**           | 운영체제 및 소프트웨어 구성이 포함된 이미지 |
| **인스턴스 유형** | CPU, 메모리 등 서버 사양 결정               |
| **리전(Region)**  | 서버를 생성할 AWS 리전                      |
| **스토리지**      | 일반적으로 EBS를 연결하여 사용              |

### AMI (Amazon Machine Image)

AMI는 EC2 인스턴스를 생성하기 위한 템플릿

예를 들어

- Ubuntu
- Amazon Linux
- Windows Server

등의 운영체제가 설치된 이미지를 선택하여 인스턴스를 생성할 수 있음

### Amazon EBS (Elastic Block Store)

- EC2에서 사용하는 영구(Block) 스토리지
- 운영체제와 애플리케이션 데이터를 저장
- EC2를 중지했다가 다시 시작해도 데이터 유지
- 하나 이상의 EBS 볼륨을 연결하여 사용 가능

> EBS는 EC2 전용 디스크라고 생각하면 됨

---

## Amazon S3 (Simple Storage Service)

### S3란?

- 객체(Object) 스토리지 서비스
- 이미지, 영상, 로그, 백업 파일 등 다양한 데이터를 저장
- 매우 높은 내구성과 확장성을 제공
- 정적 웹 사이트 호스팅도 가능

### 객체(Object) 스토리지

S3는 파일을 **객체(Object)** 단위로 저장

```text
Bucket
 ├── image.png
 ├── profile.jpg
 ├── document.pdf
 └── backup.zip
```

- Bucket : 객체를 저장하는 최상위 공간
- Object : 실제 저장되는 파일

---

## Amazon RDS (Relational Database Service)

### RDS란?

- 관계형 데이터베이스(RDBMS)를 관리형으로 제공하는 서비스
- 데이터베이스 설치 및 운영을 AWS가 대신 수행

지원 데이터베이스

- Amazon Aurora
- PostgreSQL
- MySQL
- MariaDB
- Oracle
- SQL Server
- Db2

### 특징

- VPC 내부에서 인스턴스 형태로 실행
- 자동 백업
- 자동 업데이트
- Multi-AZ(다중 가용 영역) 구성 가능
- 읽기 전용 복제본(Read Replica) 지원

AWS DMS(Database Migration Service)를 이용하면 기존 데이터베이스를 손쉽게 이전하거나 복제할 수 있음

### Amazon Aurora

- AWS가 개발한 관리형 관계형 데이터베이스
- MySQL 및 PostgreSQL과 호환
- 일반 MySQL보다 높은 성능과 가용성을 제공
- 자동 장애 조치(Failover)
- 자동 백업 지원

> Aurora는 RDS에서 선택할 수 있는 데이터베이스 엔진 중 하나

---

## Amazon DynamoDB

- 완전관리형 NoSQL 데이터베이스
- 매우 빠른 읽기/쓰기 성능
- 서버 관리가 필요 없음
- 자동 확장 지원

대표적인 사용 사례

- 게임 랭킹
- 실시간 서비스
- 세션 저장소

---

## Amazon ElastiCache

- Redis, Memcached 기반의 관리형 인메모리 캐시 서비스
- 자주 조회되는 데이터를 메모리에 저장하여 응답 속도를 향상
- 데이터베이스 부하 감소

대표적인 사용 사례

- 로그인 세션
- 인기 게시글 조회
- 캐싱

---

## 서비스 역할 비교

| 서비스          | 역할                            |
| --------------- | ------------------------------- |
| **EC2**         | 애플리케이션 실행 서버          |
| **EBS**         | EC2에서 사용하는 블록 스토리지  |
| **S3**          | 객체(Object) 스토리지           |
| **RDS**         | 관리형 관계형 데이터베이스      |
| **Aurora**      | RDS에서 제공하는 고성능 DB 엔진 |
| **DynamoDB**    | 관리형 NoSQL 데이터베이스       |
| **ElastiCache** | Redis / Memcached 기반 캐시     |

---

## 정리

- EC2는 애플리케이션을 실행하는 가상 서버
- EBS는 EC2에 연결하는 영구 블록 스토리지이며, S3는 파일을 저장하는 객체 스토리지
- RDS는 관리형 관계형 데이터베이스 서비스이고, Aurora는 RDS에서 제공하는 고성능 데이터베이스 엔진
- DynamoDB는 완전관리형 NoSQL 데이터베이스이며, ElastiCache는 Redis와 Memcached 기반의 관리형 캐시 서비스
