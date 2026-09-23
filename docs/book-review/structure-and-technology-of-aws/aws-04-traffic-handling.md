---
series: 그림으로 이해하는 AWS 구조와 기술
title: ELB·Route 53·CloudFront를 이용한 트래픽 처리
---

# ELB·Route 53·CloudFront를 이용한 트래픽 처리

> 『그림으로 이해하는 AWS 구조와 기술』를 읽고 정리한 내용

## 트래픽 처리란?

사용자의 요청(트래픽)은 하나의 서버에서 모두 처리하기보다 여러 서버에 분산하거나, 사용자와 가까운 위치에서 응답하는 것이 성능과 안정성 측면에서 유리함

AWS는 이를 위해 **Elastic Load Balancing**, **Auto Scaling**, **Route 53**, **CloudFront** 등의 서비스를 제공

---

## Elastic Load Balancing (ELB)

### ELB란?

- AWS에서 제공하는 관리형 로드 밸런서 서비스
- 여러 EC2 인스턴스로 트래픽을 자동 분산
- 특정 서버에 장애가 발생하면 정상 서버로만 요청 전달
- 고가용성(High Availability) 환경 구축 가능

> **로드 밸런서(Load Balancer)**
>
> 서버에 몰리는 접속(트래픽)을 여러 서버나 네트워크로 나누어 보내는 장치
>
> 서버 한 대에 걸리는 부하를 줄여 서비스의 안정성과 성능을 향상시킴

### ELB 종류

#### Application Load Balancer (ALB)

- HTTP, HTTPS 전용 로드 밸런서
- 애플리케이션 계층(Layer 7)에서 동작
- URL 경로(Path), 호스트 이름(Host) 등을 기준으로 라우팅 가능
- SSL/TLS 종료(Termination) 지원
- ECS, EKS, Lambda와 함께 많이 사용

대표 예시

```text
/api/*      → API 서버
/images/*   → 이미지 서버
/admin/*    → 관리자 서버
```

#### Network Load Balancer (NLB)

- TCP, UDP, TLS 기반 통신 처리
- 전송 계층(Layer 4)에서 동작
- 매우 높은 처리량과 낮은 지연 시간 제공
- 고정(Static) IP 주소 사용 가능

대표 사용 사례

- 게임 서버
- 실시간 채팅
- IoT 서비스

#### Gateway Load Balancer (GLB)

- 네트워크 보안 장비와 함께 사용하는 로드 밸런서
- 방화벽(Firewall), IDS, IPS 등의 보안 어플라이언스를 연결할 때 사용
- 네트워크 트래픽을 검사한 뒤 목적지로 전달

#### Classic Load Balancer (CLB)

- 기존 세대의 로드 밸런서
- HTTP와 TCP를 모두 지원
- 현재는 신규 서비스에서는 ALB 또는 NLB 사용이 권장됨

---

## Auto Scaling

### Auto Scaling이란?

서버의 부하에 따라 EC2 인스턴스 수를 자동으로 늘리거나 줄이는 기능

예를 들어

- 사용자가 급증하면 EC2를 자동 생성
- 사용자가 감소하면 EC2를 자동 종료

이를 통해 비용과 성능을 모두 최적화할 수 있음

### Auto Scaling이 사용하는 정보

Auto Scaling은 **Amazon CloudWatch**가 수집한 지표를 기반으로 동작함

대표적인 모니터링 항목

- CPU 사용률
- 메모리 사용량(추가 설정 필요)
- 네트워크 트래픽
- 요청(Request) 수

예시

```text
CPU 사용률 > 70%

↓

EC2 인스턴스 추가

↓

트래픽 분산
```

---

## Amazon Route 53

### Route 53이란?

- AWS의 관리형 DNS(Domain Name System) 서비스
- 도메인 이름을 실제 서버의 IP 주소 또는 AWS 서비스와 연결
- 높은 가용성과 확장성을 제공

예를 들어

```text
www.example.com

↓

Route 53

↓

Application Load Balancer

↓

EC2
```

### 주요 기능

- 도메인 등록
- DNS 레코드 관리
- 헬스 체크(Health Check)
- 장애 발생 시 자동 장애 조치(Failover)
- 다양한 라우팅 정책 제공

### 라우팅 정책

- Simple Routing
- Weighted Routing
- Latency Routing
- Geolocation Routing
- Failover Routing
- Multi Value Routing

예를 들어

- 한국 사용자는 서울 리전으로 연결
- 미국 사용자는 버지니아 리전으로 연결

과 같은 설정이 가능함

---

## Amazon CloudFront

### CloudFront란?

- AWS의 CDN(Content Delivery Network) 서비스
- 사용자와 가까운 Edge Location에서 콘텐츠를 제공
- 웹 서버의 부담을 줄이고 응답 속도를 향상

### CDN이 필요한 이유

예를 들어 서울에만 서버가 있다면

```text
미국 사용자

↓

서울 서버

↓

응답
```

매번 먼 거리를 통신해야 하므로 지연 시간이 증가함

CloudFront를 사용하면

```text
서울 서버

↓

CloudFront Edge Location

↓

미국 사용자
```

미리 캐싱된 콘텐츠를 가까운 Edge Location에서 전달하므로 응답 속도가 빨라짐

### 대표적인 캐싱 대상

- 이미지
- CSS
- JavaScript
- 동영상
- 다운로드 파일

### 장점

- 응답 속도 향상
- 원본 서버의 트래픽 감소
- 전 세계 사용자에게 빠른 콘텐츠 제공
- AWS Shield Standard가 기본 적용되어 DDoS 공격 완화

---

## 서비스 연결 예시

```text
사용자

↓

Route 53

↓

Application Load Balancer

↓

EC2 (Auto Scaling)

↓

RDS

↓

S3

↓

CloudFront (정적 콘텐츠)
```

또는 정적 웹 사이트라면

```text
사용자

↓

Route 53

↓

CloudFront

↓

S3
```

---

## 서비스 역할 비교

| 서비스           | 역할                             |
| ---------------- | -------------------------------- |
| **ELB**          | 여러 서버로 트래픽 분산          |
| **ALB**          | HTTP/HTTPS 요청 분산             |
| **NLB**          | TCP/UDP 기반 고성능 트래픽 분산  |
| **GWLB**         | 보안 장비와 연동하는 로드 밸런서 |
| **Auto Scaling** | 서버 수 자동 증감                |
| **Route 53**     | DNS 및 트래픽 라우팅             |
| **CloudFront**   | CDN을 통한 콘텐츠 캐싱 및 전송   |

---

## 정리

- `ELB`는 여러 서버에 트래픽을 분산하여 서비스의 가용성과 안정성을 높임
- `Auto Scaling`은 CloudWatch 지표를 기반으로 EC2 인스턴스 수를 자동 조절
- `Route 53`은 도메인을 AWS 리소스와 연결하고 다양한 라우팅 정책을 제공하는 DNS 서비스
- `CloudFront`는 전 세계 Edge Location을 이용하는 CDN 서비스로, 사용자에게 더 빠르게 콘텐츠를 전달하고 원본 서버의 부하를 줄여줌
