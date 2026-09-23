---
series: 그림으로 이해하는 AWS 구조와 기술
title: AWS 보안(IAM, Security Group, NACL 등)
---

# AWS 보안(IAM, Security Group, NACL 등)

> 『그림으로 이해하는 AWS 구조와 기술』를 읽고 정리한 내용

## AWS 보안이란?

AWS에서는 하나의 기능만으로 보안을 구성하지 않음

대표적으로

- **IAM** : 누가(Who) 접근할 수 있는가
- **Security Group** : 어떤 인스턴스에 접근할 수 있는가
- **Network ACL** : 어떤 서브넷에 접근할 수 있는가
- **WAF** : 웹 공격을 어떻게 차단할 것인가
- **GuardDuty** : 침입이나 이상 행위를 어떻게 탐지할 것인가

와 같은 여러 서비스를 함께 사용하여 보안을 구성함

---

## IAM (Identity and Access Management)

IAM은 AWS 리소스에 대한 인증(Authentication)과 인가(Authorization)를 관리하는 서비스

주요 기능

- IAM 사용자(User) 생성
- IAM 그룹(Group) 관리
- IAM 역할(Role) 생성
- IAM 정책(Policy)을 통한 권한 관리
- 최소 권한 원칙(Least Privilege) 적용

예를 들어

- 개발자는 EC2만 관리 가능
- 운영자는 RDS까지 접근 가능
- Lambda는 특정 S3 버킷만 접근 가능

과 같이 필요한 권한만 부여할 수 있음

---

## Security Group

### Security Group이란?

- EC2, RDS 등에 적용하는 **가상 방화벽**
- 인스턴스 단위로 적용
- 허용(Allow) 규칙만 설정 가능
- Stateful 방식으로 동작

Stateful이란

요청을 허용하면 응답 패킷은 별도의 규칙 없이 자동 허용되는 방식

예를 들어

```
Inbound
TCP 443 허용
```

이라면

```
클라이언트 → EC2
```

뿐 아니라

```
EC2 → 클라이언트
```

응답도 자동으로 허용

### 대표적인 규칙

| 포트 | 용도       |
| ---- | ---------- |
| 22   | SSH        |
| 80   | HTTP       |
| 443  | HTTPS      |
| 3306 | MySQL      |
| 5432 | PostgreSQL |

---

## Network ACL (Network Access Control List)

### Network ACL이란?

- 서브넷(Subnet)에 적용되는 가상 방화벽
- 허용(Allow)과 거부(Deny) 모두 설정 가능
- Stateless 방식으로 동작
- 번호(Number)가 낮은 규칙부터 순서대로 적용

Stateless란

들어오는 트래픽과 나가는 트래픽을 각각 별도로 검사하는 방식

예를 들어

```
Inbound 허용
```

만 설정하면

```
Outbound
```

도 별도로 허용해야 응답이 가능함

---

## Security Group과 Network ACL 비교

| 항목      | Security Group            | Network ACL                     |
| --------- | ------------------------- | ------------------------------- |
| 적용 범위 | 인스턴스                  | 서브넷                          |
| 규칙      | 허용만 가능               | 허용/거부 가능                  |
| 동작 방식 | Stateful                  | Stateless                       |
| 규칙 적용 | 모든 규칙을 종합하여 판단 | 번호 순서대로 적용              |
| 기본 설정 | 모든 Outbound 허용        | Inbound/Outbound 모두 규칙 필요 |

> 일반적으로 Security Group이 기본적인 접근 제어를 담당하고, Network ACL은 서브넷 전체에 대한 추가적인 보안 정책을 적용할 때 사용

---

## AWS WAF (Web Application Firewall)

### WAF란?

웹 애플리케이션을 대상으로 하는 공격을 차단하는 방화벽

일반적인 방화벽은 IP, 포트 등의 네트워크 정보만 검사하지만,

WAF는

- HTTP
- HTTPS
- URL
- Header
- Cookie
- Query String

등 웹 요청의 내용까지 분석하여 공격을 차단함

대표적으로 방어 가능한 공격

- SQL Injection(SQLi)
- Cross Site Scripting(XSS)
- 악성 Bot
- 비정상적인 HTTP 요청

CloudFront, ALB, API Gateway 등과 함께 사용할 수 있음

---

## Amazon Shield

AWS에서 제공하는 DDoS(Distributed Denial of Service) 공격 방어 서비스

### Shield Standard

- 모든 AWS 고객에게 기본 제공
- CloudFront, Route 53, ELB 등을 DDoS 공격으로부터 보호

### Shield Advanced

- 대규모 DDoS 공격 대응
- 24시간 AWS DDoS 대응팀 지원
- 추가 비용 발생

---

## Amazon GuardDuty

GuardDuty는 AWS 환경에서 발생하는 이상 행위를 탐지하는 관리형 위협 탐지 서비스

AI와 머신러닝을 이용하여 다양한 로그를 분석

분석 대상

- AWS CloudTrail
- VPC Flow Logs
- DNS Logs
- EKS Audit Logs
- S3 이벤트

탐지 가능한 사례

- 비정상적인 로그인
- 암호화폐 채굴 시도
- EC2 침해
- 악성 IP와의 통신
- 자격 증명 탈취

---

## AWS CloudTrail

CloudTrail은 AWS 계정에서 발생하는 API 호출과 사용자 활동을 기록하는 서비스

예를 들어

- 누가 EC2를 생성했는지
- 누가 IAM 권한을 변경했는지
- 누가 S3 객체를 삭제했는지

등을 확인할 수 있음

보안 감사(Audit)와 사고 분석에 자주 사용됨

---

## Amazon Inspector

EC2, ECR, Lambda 등의 보안 취약점을 자동으로 검사하는 서비스

대표 기능

- 운영체제 취약점 검사
- 라이브러리 취약점 검사
- CVE(Common Vulnerabilities and Exposures) 탐지
- 보안 권장 사항 제공

---

## 서비스 역할 비교

| 서비스             | 역할                         |
| ------------------ | ---------------------------- |
| **IAM**            | 사용자 및 서비스 권한 관리   |
| **Security Group** | 인스턴스 단위 접근 제어      |
| **Network ACL**    | 서브넷 단위 접근 제어        |
| **AWS WAF**        | 웹 공격 차단                 |
| **AWS Shield**     | DDoS 공격 방어               |
| **GuardDuty**      | 이상 행위 및 위협 탐지       |
| **CloudTrail**     | API 호출 및 사용자 활동 기록 |
| **Inspector**      | 보안 취약점 검사             |

---

## 정리

- IAM은 사용자와 서비스의 인증 및 권한을 관리
- Security Group은 인스턴스 단위, Network ACL은 서브넷 단위의 가상 방화벽
- WAF는 웹 공격을 차단하고, Shield는 DDoS 공격을 방어함
- GuardDuty는 이상 행위를 탐지하며, CloudTrail은 AWS 활동을 기록하고 Inspector는 보안 취약점을 검사함
