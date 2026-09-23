---
series: 그림으로 이해하는 AWS 구조와 기술
title: VPC와 네트워크
---

# VPC와 네트워크

> 『그림으로 이해하는 AWS 구조와 기술』를 읽고 정리한 내용

## Amazon VPC (Virtual Private Cloud)

### VPC란?

- AWS에서 격리된 가상 네트워크를 생성하는 서비스
- EC2, RDS 등 대부분의 AWS 리소스는 VPC 안에서 실행됨
- 쉽게 말하면 AWS에서 사용하는 하나의 **가상 LAN**이라고 생각하면 됨
- VPC마다 독립적인 IP 주소 범위와 네트워크 구성을 가질 수 있음

### VPC의 주요 기능

- CIDR(Classless Inter-Domain Routing)로 네트워크 주소 범위를 설정
- VPC를 여러 개의 서브넷(Subnet)으로 분할 가능
- 라우팅 테이블(Route Table)을 이용해 패킷의 이동 경로 설정
- 인터넷 게이트웨이, NAT 게이트웨이 등을 이용하여 외부 네트워크와 연결
- 보안 그룹(Security Group), 네트워크 ACL(Network ACL)을 이용한 접근 제어

---

## CIDR(Classless Inter-Domain Routing)

CIDR은 네트워크의 IP 주소 범위를 표현하는 표기법

예를 들어

```text
10.0.0.0/16
```

이라면

- VPC의 네트워크 주소 범위는 `10.0.0.0 ~ 10.0.255.255`
- 이후 이 범위를 여러 개의 서브넷으로 나누어 사용할 수 있음

예시

```text
VPC
10.0.0.0/16

├── Public Subnet
│   10.0.1.0/24
│
└── Private Subnet
    10.0.2.0/24
```

---

## Subnet(서브넷)

서브넷은 VPC를 더 작은 네트워크 단위로 나눈 것

일반적으로 다음 두 가지 형태로 구성

### Public Subnet

- 인터넷 게이트웨이와 연결된 서브넷
- 인터넷에서 직접 접근 가능
- Bastion Host, Load Balancer 등이 위치

### Private Subnet

- 인터넷에서 직접 접근 불가
- EC2, RDS 등 내부 서비스 배치
- 외부 인터넷이 필요하면 NAT Gateway를 이용

---

## Route Table

라우팅 테이블은 패킷이 어디로 이동해야 하는지를 정의하는 규칙

예를 들어

| 목적지      | 대상             |
| ----------- | ---------------- |
| 10.0.0.0/16 | Local            |
| 0.0.0.0/0   | Internet Gateway |

위와 같이 설정하면

- 같은 VPC 내부는 Local로 전달
- 그 외 모든 네트워크는 Internet Gateway로 전달

---

## Internet Gateway (IGW)

인터넷 게이트웨이는 VPC와 인터넷을 연결하는 게이트웨이

특징

- VPC와 인터넷 간 양방향 통신 제공
- Public Subnet이 인터넷과 통신하기 위해 필요
- Public IP 또는 Elastic IP가 할당된 리소스만 인터넷과 직접 통신 가능

> Internet Gateway는 NAT가 아님
> VPC와 인터넷을 연결하는 라우터 역할을 수행함

---

## NAT Gateway

Private Subnet의 리소스가 인터넷으로 나갈 수 있도록 해주는 서비스

특징

- Private Subnet → Internet 가능
- Internet → Private Subnet 직접 접근 불가
- Public Subnet에 생성하여 사용
- Elastic IP를 사용하여 외부와 통신

동작 예시

```text
Private EC2
10.0.2.10

↓

NAT Gateway

↓

Elastic IP

↓

Internet
```

> NAT Gateway는 내부에서 시작한 연결에 대해서만 응답을 받을 수 있음

---

## VPC Endpoint

일부 AWS 서비스는 인터넷 게이트웨이나 NAT Gateway를 거치지 않고도 접근할 수 있음

이를 가능하게 하는 것이 **VPC Endpoint**

즉,

**VPC와 AWS 서비스를 AWS 내부 네트워크로 직접 연결하는 기능**

장점

- 인터넷을 거치지 않음
- 보안 향상
- NAT Gateway 비용 절감
- 네트워크 지연 감소

### Gateway Endpoint

다음 서비스에서 사용

- Amazon S3
- Amazon DynamoDB

라우팅 테이블에 경로를 추가하여 연결

### Interface Endpoint

AWS PrivateLink를 기반으로 동작

ENI(Elastic Network Interface)를 생성하여 연결하며 대부분의 AWS 서비스에서 사용 가능

예시

- AWS Systems Manager
- Amazon CloudWatch
- Amazon ECR
- AWS Secrets Manager

---

## VPC 구성 예시

```text
                 Internet
                     │
          Internet Gateway
                     │
      ┌──────────────┴──────────────┐
      │                             │
 Public Subnet                 Public Subnet
      │                             │
    ALB                      NAT Gateway
      │                             │
      └──────────────┬──────────────┘
                     │
             Private Subnet
          ┌──────────┴──────────┐
          │                     │
        EC2                   RDS
```

---

## 정리

- VPC는 AWS에서 사용하는 독립적인 가상 네트워크
- VPC는 CIDR로 IP 범위를 지정하고 여러 개의 서브넷으로 분리하여 사용
- Public Subnet은 Internet Gateway를 통해 인터넷과 직접 통신하고, Private Subnet은 NAT Gateway를 통해서만 외부와 통신할 수 있음
- Route Table은 패킷의 이동 경로를 결정하며, VPC Endpoint를 이용하면 인터넷을 거치지 않고 AWS 서비스에 안전하게 접근할 수 있음
