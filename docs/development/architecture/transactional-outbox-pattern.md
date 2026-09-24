## 트랜잭션 아웃박스 패턴

마이크로서비스 아키텍처에서는 하나의 비즈니스 작업을 처리하면서 데이터베이스를 변경하고 다른 서비스에 이벤트를 발행해야 하는 경우가 많다.

예를 들어 주문 서비스에서 주문이 생성되면 다음과 같은 작업이 필요할 수 있다.

1. 주문 정보를 데이터베이스에 저장한다.
2. `OrderCreated` 이벤트를 발행한다.
3. 결제 서비스는 이벤트를 받아 결제를 처리한다.
4. 재고 서비스는 이벤트를 받아 재고를 차감한다.
5. 알림 서비스는 이벤트를 받아 사용자에게 주문 생성 알림을 보낸다.

구조로 나타내면 다음과 같다.

```text
                    주문 서비스
                       │
              ┌────────┴────────┐
              ↓                 ↓
          주문 DB             Kafka
              │                 │
              ↓                 ↓
          주문 저장         OrderCreated
                                │
                    ┌───────────┼───────────┐
                    ↓           ↓           ↓
                  결제        재고         알림
                 서비스      서비스       서비스
```

문제는 **주문 DB에 데이터를 저장하는 작업과 이벤트를 메시지 브로커에 발행하는 작업이 서로 다른 시스템에서 수행된다는 것**이다.

이때 데이터베이스에는 주문이 저장되었지만 이벤트가 발행되지 않거나, 반대로 이벤트는 발행되었지만 주문 저장에 실패하는 문제가 발생할 수 있다.

이러한 문제를 해결하기 위해 사용하는 패턴이 **트랜잭션 아웃박스(Transaction Outbox) 패턴**이다.

## 왜 트랜잭션 아웃박스가 필요한가

### 주문 생성과 이벤트 발행

주문 서비스를 단순하게 구현하면 다음과 같은 코드가 될 수 있다.

```java
@Transactional
public void createOrder(CreateOrderCommand command) {

    Order order = Order.create(command);

    orderRepository.save(order);

    eventPublisher.publish(
        new OrderCreatedEvent(order.getId())
    );
}
```

코드만 보면 크게 문제가 없어 보인다.

주문을 저장하고 바로 `OrderCreated` 이벤트를 발행하면 되기 때문이다.

하지만 실제로는 다음 두 작업이 서로 다른 시스템에서 수행된다.

```text
주문 서비스
   │
   ├── ① 주문 DB에 주문 저장
   │
   └── ② Kafka에 이벤트 발행
```

`@Transactional`은 일반적으로 데이터베이스 트랜잭션을 의미한다.

즉, 데이터베이스에 수행되는 작업은 하나의 트랜잭션으로 묶을 수 있지만 Kafka와 같은 외부 메시지 브로커까지 동일한 트랜잭션으로 묶는 것은 별개의 문제이다.

따라서 다음과 같은 상황이 발생할 수 있다.

### DB 저장은 성공했지만 이벤트 발행이 실패하는 경우

가장 대표적인 문제이다.

```text
① 주문 DB 저장
        ↓
      성공
        ↓
② Kafka 이벤트 발행
        ↓
      실패
```

결과적으로 데이터베이스에는 주문이 존재한다.

```text
orders

id    member_id    amount
-------------------------
100   member-1     15000
```

하지만 Kafka에는 `OrderCreated` 이벤트가 존재하지 않는다.

```text
Kafka

OrderCreated #100
→ 없음
```

이 경우 결제 서비스나 재고 서비스는 주문이 생성되었다는 사실을 알 수 없다.

```text
Order Service
     │
     ├── 주문 저장 성공
     │
     └── 이벤트 발행 실패
              X
              │
       ┌──────┴──────┐
       ↓             ↓
    Payment       Inventory
    Service        Service

    주문 생성 사실을 전달받지 못함
```

결국 서비스 사이의 데이터가 서로 일치하지 않게 된다.

### 이벤트 발행은 성공했지만 DB 저장이 실패하는 경우

반대의 상황도 문제가 된다.

```text
① Kafka 이벤트 발행
        ↓
      성공
        ↓
② 주문 DB 저장
        ↓
      실패
```

그러면 Kafka에는 `OrderCreated` 이벤트가 존재하지만 실제 주문은 데이터베이스에 존재하지 않는다.

```text
Kafka
└── OrderCreated #100  ✅

orders
└── 주문 #100           ❌
```

결제 서비스가 이벤트를 받아 결제를 처리했는데 주문 서비스에는 해당 주문이 존재하지 않는 상황까지 발생할 수 있다.

결국 핵심 문제는 다음과 같다.

> **하나의 비즈니스 작업에서 데이터베이스 변경과 이벤트 발행이라는 두 가지 작업을 수행하지만, 두 작업을 하나의 트랜잭션으로 묶기 어렵다는 것이다.**

이러한 문제를 흔히 **Dual Write Problem**이라고 한다.

## 트랜잭션 아웃박스 패턴이란

트랜잭션 아웃박스 패턴은 데이터베이스 변경과 이벤트 발행을 직접 묶으려고 하지 않는다.

대신 **발행해야 할 이벤트를 데이터베이스의 Outbox 테이블에 함께 저장한다.**

즉 기존 구조를 다음과 같이 변경한다.

```text
기존

주문 서비스
   │
   ├── 주문 DB
   │
   └── Kafka
```

이를 다음과 같이 변경한다.

```text
주문 서비스
   │
   └── DB
        │
        ├── orders
        │
        └── outbox
                │
                ↓
          Outbox Publisher
                │
                ↓
              Kafka
```

여기서 중요한 것은 **주문 데이터와 Outbox 데이터를 동일한 데이터베이스 트랜잭션으로 저장한다는 것**이다.

```text
@Transactional

┌─────────────────────────────────┐
│          DB Transaction         │
│                                 │
│  orders INSERT                  │
│        +                        │
│  outbox INSERT                  │
│                                 │
└─────────────────────────────────┘
```

이렇게 하면 주문 저장과 이벤트 기록의 성공 여부를 하나의 트랜잭션으로 보장할 수 있다.

## Outbox 테이블

Outbox는 말 그대로 **외부로 전달해야 할 메시지를 임시로 저장하는 공간**이다.

예를 들어 다음과 같은 테이블을 만들 수 있다.

```sql
CREATE TABLE outbox (
    id            BIGINT PRIMARY KEY,
    aggregate_type VARCHAR(100) NOT NULL,
    aggregate_id   VARCHAR(100) NOT NULL,
    event_type     VARCHAR(100) NOT NULL,
    payload        JSON NOT NULL,
    created_at     TIMESTAMP NOT NULL
);
```

각 컬럼은 다음과 같은 의미를 가진다.

| 컬럼             | 설명                            |
| ---------------- | ------------------------------- |
| `id`             | 이벤트를 식별하기 위한 ID       |
| `aggregate_type` | 이벤트가 발생한 애그리거트 타입 |
| `aggregate_id`   | 이벤트가 발생한 애그리거트 ID   |
| `event_type`     | 이벤트 종류                     |
| `payload`        | 실제 이벤트 데이터              |
| `created_at`     | 이벤트 생성 시간                |

예를 들어 주문이 생성되면 다음과 같은 데이터가 저장될 수 있다.

```text
id            = 1
aggregate_type = Order
aggregate_id   = order-100
event_type     = OrderCreated
payload        = {
                   "orderId": "order-100",
                   "memberId": "member-1",
                   "amount": 15000
                 }
created_at     = 2026-09-22 10:00:00
```

## 트랜잭션 아웃박스의 동작 과정

주문 생성 과정을 기준으로 살펴보자.

### 1. 주문 생성

사용자가 주문을 생성한다.

```text
Client
  │
  ↓
Order Service
```

주문 서비스는 주문 객체를 생성한다.

```java
Order order = Order.create(command);
```

### 2. 주문과 Outbox 이벤트를 함께 저장한다

주문 서비스는 주문 데이터를 저장하면서 이벤트도 Outbox 테이블에 저장한다.

```java
@Transactional
public void createOrder(CreateOrderCommand command) {

    Order order = Order.create(command);

    orderRepository.save(order);

    OutboxEvent event = OutboxEvent.create(
        "OrderCreated",
        order.getId(),
        order
    );

    outboxRepository.save(event);
}
```

데이터베이스에는 다음과 같이 저장된다.

```text
orders
--------------------------------
id       member_id       amount
100      member-1        15000


outbox
--------------------------------
id       event_type      aggregate_id
1        OrderCreated    order-100
```

이 두 작업은 하나의 트랜잭션으로 처리된다.

### 3. 트랜잭션 커밋

두 작업이 모두 성공하면 트랜잭션이 커밋된다.

```text
┌──────────────────────────────┐
│        DB Transaction        │
│                              │
│  Order 저장       성공       │
│  Outbox 저장      성공       │
│                              │
└──────────────┬───────────────┘
               ↓
             COMMIT
```

이제 데이터베이스에는 주문과 이벤트가 모두 존재한다.

### 4. Outbox Publisher가 이벤트를 읽는다

이벤트를 Kafka에 전달하는 별도의 Publisher가 Outbox 테이블을 조회한다.

```text
Outbox
   │
   │ 조회
   ↓
Outbox Publisher
```

그리고 Outbox에 저장된 이벤트를 Kafka로 전달한다.

```text
Outbox Publisher
       │
       ↓
     Kafka
       │
       ↓
OrderCreated
```

### 5. 다른 서비스가 이벤트를 처리한다

Kafka에 이벤트가 발행되면 다른 서비스들이 이벤트를 소비한다.

```text
                    Kafka
                      │
              OrderCreated
                      │
          ┌───────────┼───────────┐
          ↓           ↓           ↓
       Payment     Inventory   Notification
       Service      Service      Service
```

이렇게 주문 서비스와 다른 서비스의 결합도를 낮출 수 있다.

## DB 저장에 실패하면 어떻게 되는가

트랜잭션 아웃박스의 가장 중요한 장점 중 하나이다.

다음과 같이 주문 저장에 실패했다고 가정하자.

```text
@Transactional

Order 저장
   ↓
실패
   ↓
ROLLBACK
```

그러면 동일한 트랜잭션에 포함된 Outbox 저장도 함께 롤백된다.

```text
orders
└── Order #100 ❌

outbox
└── OrderCreated #100 ❌
```

따라서 존재하지 않는 주문에 대한 이벤트가 발행되는 문제를 방지할 수 있다.

즉,

> **업무 데이터가 저장되지 않았는데 이벤트만 발행되는 상황을 방지할 수 있다.**

## Kafka가 장애가 발생하면 어떻게 되는가

이번에는 주문 저장과 Outbox 저장은 성공했지만 Kafka가 장애가 발생했다고 가정하자.

```text
Order 저장        성공
Outbox 저장       성공
        ↓
      COMMIT
        ↓
Outbox Publisher
        ↓
Kafka
        ↓
      실패
```

이 경우에도 이벤트는 사라지지 않는다.

```text
orders
└── Order #100

outbox
└── OrderCreated #100
```

Kafka가 복구된 후 Outbox Publisher가 다시 이벤트를 발행하면 된다.

```text
Outbox
   │
   │ 재시도
   ↓
Publisher
   │
   ↓
Kafka
   │
   ↓
OrderCreated
```

이것이 트랜잭션 아웃박스의 핵심적인 장점이다.

> **이벤트를 외부 시스템에 바로 전달하지 않고 데이터베이스에 먼저 기록하기 때문에, 일시적인 메시지 브로커 장애가 발생하더라도 이벤트를 재처리할 수 있다.**

## 트랜잭션 아웃박스의 장점

### 이벤트 유실 방지

가장 큰 장점이다.

기존 방식에서는 다음과 같은 상황에서 이벤트가 유실될 수 있다.

```text
DB 저장
  ↓
COMMIT
  ↓
애플리케이션 장애
  ↓
Kafka 전송 실패
```

트랜잭션 아웃박스를 사용하면 이벤트 자체를 DB에 저장하기 때문에 나중에 다시 처리할 수 있다.

```text
DB 저장
  +
Outbox 저장
  ↓
COMMIT
  ↓
애플리케이션 장애
  ↓
재시작
  ↓
Outbox 재처리
  ↓
Kafka
```

### DB와 이벤트 기록의 원자성 확보

주문 데이터와 Outbox 이벤트를 하나의 데이터베이스 트랜잭션으로 처리한다.

```text
┌─────────────────────────┐
│      Transaction        │
│                         │
│  Order 저장             │
│       +                 │
│  Event 저장             │
│                         │
└─────────────────────────┘
```

따라서 둘 중 하나만 성공하는 상황을 방지할 수 있다.

### 재처리가 가능하다

Outbox에 이벤트가 남아 있기 때문에 이벤트 발행에 실패하더라도 다시 처리할 수 있다.

이는 일시적인 네트워크 장애나 메시지 브로커 장애에 대응하는 데 유용하다.

### 분산 트랜잭션을 피할 수 있다

DB와 Kafka를 하나의 분산 트랜잭션으로 묶으려고 하면 시스템이 복잡해질 수 있다.

트랜잭션 아웃박스는 DB 내부에서 다음 두 작업을 하나의 트랜잭션으로 처리한다.

```text
주문 데이터 변경
        +
이벤트 기록
```

그리고 실제 메시지 발행은 별도의 비동기 프로세스로 처리한다.

이를 통해 DB와 메시지 브로커 사이의 결합을 줄일 수 있다.

## 트랜잭션 아웃박스의 한계

트랜잭션 아웃박스가 모든 문제를 해결해 주는 것은 아니다.

특히 **중복 이벤트**에 주의해야 한다.

### 이벤트가 중복 발행될 수 있다

다음과 같은 상황을 생각해보자.

```text
1. Outbox 조회
       ↓
2. Kafka 이벤트 발행 성공
       ↓
3. Outbox 삭제
       ↓
4. 애플리케이션 장애
```

3번 과정에서 Outbox 삭제가 완료되기 전에 애플리케이션이 장애가 발생할 수 있다.

그러면 다음번 Publisher 실행 시 동일한 이벤트를 다시 읽게 된다.

```text
Outbox
   │
   ↓
Kafka
   │
   ↓
OrderCreated #100

     ↓ 애플리케이션 장애

Outbox에 이벤트가 여전히 존재
   │
   ↓
다시 발행
   │
   ↓
OrderCreated #100
```

결과적으로 동일한 이벤트가 두 번 전달될 수 있다.

따라서 트랜잭션 아웃박스는 **이벤트가 정확히 한 번만 전달되는 것을 보장하는 패턴이 아니다.**

오히려 이벤트가 유실되지 않도록 하고, 발생할 수 있는 중복은 소비자가 처리하도록 설계하는 방식이다.

## Consumer의 멱등성이 중요한 이유

중복 이벤트를 안전하게 처리하려면 Consumer가 **멱등성(Idempotency)**을 가져야 한다.

예를 들어 결제 서비스가 다음 이벤트를 받는다고 가정하자.

```json
{
  "eventId": "event-100",
  "eventType": "OrderCreated",
  "orderId": "order-100"
}
```

Consumer는 처리한 이벤트의 ID를 기록할 수 있다.

```text
processed_events

event_id
---------
event-100
```

이후 동일한 이벤트가 다시 들어오면 이미 처리한 이벤트인지 확인한다.

```text
OrderCreated event-100
          │
          ↓
   이미 처리했는가?
       /       \
     YES        NO
      │          │
      ↓          ↓
    무시       이벤트 처리
                 │
                 ↓
          처리 기록 저장
```

따라서 트랜잭션 아웃박스를 사용할 때는 다음 두 가지를 함께 고려해야 한다.

```text
Transactional Outbox
        +
Idempotent Consumer
```

## 이벤트 순서도 고려해야 한다

이벤트가 중복될 수 있는 것뿐만 아니라 이벤트의 순서도 중요하다.

예를 들어 주문에 대해 다음과 같은 이벤트가 발생한다고 하자.

```text
OrderCreated
      ↓
OrderPaid
      ↓
OrderCancelled
```

Consumer가 이를 다음 순서로 받는다면 문제가 발생할 수 있다.

```text
OrderPaid
      ↓
OrderCreated
      ↓
OrderCancelled
```

따라서 이벤트의 순서가 중요한 경우에는 이벤트 생성 시간이나 시퀀스 번호 등을 이용해 순서를 관리할 필요가 있다.

다만 모든 이벤트 기반 시스템에서 전역적인 이벤트 순서를 보장해야 하는 것은 아니다.

**순서가 비즈니스적으로 중요한 이벤트인지 먼저 판단해야 한다.**

## Outbox 데이터는 어떻게 관리하는가

Outbox를 사용하면 이벤트 데이터가 계속 쌓이게 된다.

```text
outbox

event-1
event-2
event-3
event-4
event-5
...
```

따라서 다음과 같은 관리 전략이 필요하다.

- 발행 완료 이벤트 삭제
- 발행 완료 여부를 상태로 관리
- 일정 기간이 지난 이벤트 삭제
- 실패한 이벤트의 재시도 횟수 관리
- 오래된 이벤트를 별도의 저장소로 이동
- Outbox 테이블에 적절한 인덱스 구성

예를 들어 다음과 같은 컬럼을 추가할 수도 있다.

```text
outbox

id
event_type
aggregate_id
payload
status
retry_count
created_at
published_at
```

이렇게 하면 이벤트 발행 상태와 재시도 횟수 등을 관리할 수 있다.

## Outbox를 어떻게 읽을 것인가

Outbox를 처리하는 대표적인 방법은 **Polling**과 **CDC(Change Data Capture)**이다.

### Polling

가장 단순한 방식은 별도의 Publisher가 주기적으로 Outbox 테이블을 조회하는 것이다.

```text
              Outbox
                 │
              SELECT
                 ↓
          Outbox Publisher
                 │
                 ↓
               Kafka
```

예를 들어 다음과 같이 발행되지 않은 이벤트를 조회할 수 있다.

```sql
SELECT *
FROM outbox
WHERE status = 'PENDING'
ORDER BY created_at
LIMIT 100;
```

Publisher가 이벤트를 Kafka에 전달하고 성공하면 해당 이벤트를 처리 완료 상태로 변경하거나 삭제한다.

### CDC

CDC를 이용하면 데이터베이스의 변경 사항을 감지하여 이벤트를 전달할 수도 있다.

구조는 다음과 같다.

```text
Database
    │
    │ 데이터 변경
    ↓
   CDC
    │
    ↓
Message Broker
    │
    ↓
Consumer
```

Polling처럼 주기적으로 테이블을 조회하는 대신 데이터베이스의 변경 로그 등을 이용해 변경 사항을 전달하는 방식이다.

어떤 방식을 사용할지는 사용하는 데이터베이스와 메시징 시스템, 처리량, 운영 환경 등을 고려해 결정해야 한다.

## 트랜잭션 아웃박스와 Saga

트랜잭션 아웃박스와 Saga는 함께 등장하는 경우가 많지만 해결하려는 문제는 다르다.

트랜잭션 아웃박스는 주로 다음 문제를 해결한다.

> **서비스 내부의 DB 변경과 이벤트 발행 사이의 일관성을 어떻게 보장할 것인가?**

반면 Saga는 여러 마이크로서비스에 걸친 하나의 비즈니스 트랜잭션을 어떻게 처리할 것인가에 대한 패턴이다.

예를 들어 다음과 같은 흐름이 있다고 하자.

```text
Order Service
      ↓
Payment Service
      ↓
Inventory Service
      ↓
Shipping Service
```

각 서비스는 자신의 DB를 가지고 있을 수 있다.

```text
Order Service
    ├── Order DB
    └── Outbox

Payment Service
    ├── Payment DB
    └── Outbox

Inventory Service
    ├── Inventory DB
    └── Outbox
```

이때 각 서비스에서는 트랜잭션 아웃박스를 사용하고, 서비스 간 전체 비즈니스 흐름은 Saga로 관리하는 구조를 사용할 수 있다.

즉 둘은 서로 대체하는 관계라기보다는 **서로 다른 문제를 해결하면서 함께 사용할 수 있는 패턴**이다.

## 정리

트랜잭션 아웃박스 패턴은 **데이터베이스 변경과 이벤트 발행 사이에서 발생할 수 있는 데이터 불일치 문제를 해결하기 위한 패턴**이다.

핵심은 이벤트를 메시지 브로커에 바로 발행하지 않는 것이다.

먼저 업무 데이터와 이벤트를 Outbox 테이블에 함께 저장한다.

```text
주문 생성
   │
   ├── Order 저장
   │
   └── OrderCreated 이벤트 저장
             │
             ↓
          COMMIT
             │
             ↓
       Outbox Publisher
             │
             ↓
           Kafka
             │
       ┌─────┼─────┐
       ↓     ↓     ↓
     결제   재고   알림
```

이를 통해 DB 저장은 성공했지만 이벤트가 유실되는 문제를 방지하고, 메시지 브로커의 일시적인 장애가 발생하더라도 이벤트를 재시도할 수 있다.

다만 트랜잭션 아웃박스가 이벤트의 **정확히 한 번 전달(Exactly Once)**을 보장하는 것은 아니다.

이벤트가 중복 발행될 수 있기 때문에 Consumer는 멱등적으로 설계해야 한다.

따라서 트랜잭션 아웃박스의 핵심을 다음과 같이 정리할 수 있다.

```text
DB 변경 + 이벤트 기록
        ↓
    하나의 DB 트랜잭션
        ↓
      Outbox
        ↓
   비동기 이벤트 발행
        ↓
   Message Broker
        ↓
   Idempotent Consumer
```

결국 트랜잭션 아웃박스는 **DB와 메시지 브로커를 하나의 트랜잭션으로 묶는 기술이 아니라, DB 트랜잭션 안에 "이벤트를 발행해야 한다는 사실"을 함께 기록하고 이후 안전하게 전달하는 패턴**이다.
