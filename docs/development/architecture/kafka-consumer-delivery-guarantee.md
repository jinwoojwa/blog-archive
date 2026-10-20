## Kafka Consumer의 전달 보장과 멱등성

> 공부하고 정리한 내용이라 틀린 부분이 있을 수 있습니다. 잘못된 내용을 발견하시면 알려주세요.

Kafka Consumer는 메시지를 읽는 것에서 끝나지 않습니다. Offset Commit 시점과 중복 처리 방지 전략까지 함께 설계해야 합니다.

## 메시지 처리 과정

이제 재고 예약을 예시로 메시지 하나가 이동하는 흐름을 따라가 보겠습니다.

```text
1. OMS가 재고 예약 요청
2. 재고 서비스가 가용 재고를 확인하고 예약 생성
3. ReservationChanged 이벤트 기록
4. Kafka Topic에 이벤트 발행
5. OMS 또는 WMS가 이벤트 소비
6. 각 서비스가 자신의 DB와 업무 상태 갱신
7. 성공한 위치까지 Offset Commit
```

예시 이벤트는 다음과 같은 형태가 될 수 있습니다.

```json
{
  "reservationId": "reservation-20261010-001",
  "warehouseId": "warehouse-seoul-01",
  "eventType": "RESERVATION_CREATED",
  "occurredAt": "2026-10-10T09:30:00Z",
  "items": [
    {
      "productId": "coffee-bean-001",
      "quantity": 3
    }
  ]
}
```

메시지 본문에는 비즈니스 데이터가 들어가고, Kafka 헤더에는 `event-id`, `event-type`, `aggregate-type`, `occurred-at`처럼 처리에 필요한 메타데이터를 둘 수 있습니다.

여기서 `event-id`는 매우 중요합니다. 네트워크 장애나 프로세스 종료가 발생하면 같은 이벤트가 다시 발행되거나 다시 소비될 수 있습니다. 소비자는 이 ID로 이미 처리한 이벤트인지 판단할 수 있어야 합니다.

## Offset Commit과 메시지 전달 보장

Consumer가 메시지를 읽었다는 것과, 비즈니스 처리를 성공했다는 것은 다릅니다.

Kafka에서 Consumer는 메시지를 처리한 뒤 Offset을 Commit합니다. Commit은 “이 Partition의 여기까지 처리가 완료되었다”고 기록하는 행위입니다.

### Commit을 먼저 하면: At-most-once

```text
1. 메시지 수신
2. Offset Commit
3. 재고 처리
4. 애플리케이션 장애
```

이 경우 Offset은 이미 앞으로 이동합니다. 재고 처리가 끝나기 전에 장애가 나면 Kafka는 메시지를 다시 전달하지 않습니다. 메시지가 유실될 수 있지만 중복 처리 가능성은 낮습니다.

이를 **At-most-once**, 즉 “최대 한 번 전달”이라고 합니다.

### 처리 후 Commit하면: At-least-once

```text
1. 메시지 수신
2. 재고 처리 성공
3. Offset Commit 전에 애플리케이션 장애
4. 재시작 후 같은 메시지 재수신
```

이 경우 메시지는 다시 전달될 수 있습니다. 메시지 유실 가능성은 낮아지지만 중복 처리가 발생할 수 있습니다.

이를 **At-least-once**, 즉 “최소 한 번 전달”이라고 합니다.

대부분의 비즈니스 이벤트 처리에서는 유실보다 중복이 더 안전합니다. 그래서 보통은 처리 성공 후 Offset을 Commit하고, 소비자가 중복 메시지를 안전하게 처리하도록 설계합니다.

### Auto Commit과 Manual Commit

Auto Commit은 일정 간격으로 Offset을 자동 저장합니다. 구현은 간단하지만 실제 처리 성공 시점과 Commit 시점이 어긋날 수 있습니다.

Manual Commit은 애플리케이션이 비즈니스 작업 성공을 확인한 뒤 직접 Offset을 기록합니다. 실패 처리와 재시도 정책을 명확히 통제해야 하는 Consumer라면 이 방식이 더 적합합니다.

### Exactly-once를 오해하지 않기

Kafka에서 Exactly-once는 특정 조건에서 Kafka 내부의 읽기·쓰기 처리를 한 번처럼 보이게 만드는 기능입니다. 하지만 이것이 외부 DB 작업까지 자동으로 정확히 한 번 실행해 준다는 뜻은 아닙니다.

예를 들어 Consumer가 다음을 수행한다고 가정해 보겠습니다.

```text
Kafka 메시지 소비
  └─ 외부 DB에 재고 이력 저장
      └─ Offset Commit
```

DB 저장에는 성공했지만 Offset Commit 전에 장애가 발생하면 메시지는 다시 전달됩니다. DB에 이미 기록한 작업을 구분하지 못하면 이력이 중복 저장될 수 있습니다.

따라서 외부 DB, HTTP API, 이메일 발송처럼 Kafka 밖에 영향을 주는 작업에서는 멱등성 설계가 필요합니다.

### Idempotency: 같은 메시지가 여러 번 와도 안전하게 처리합니다

At-least-once 전달에서는 중복 메시지가 정상적인 상황입니다. 따라서 Consumer는 같은 메시지를 두 번 받아도 결과가 한 번 처리한 것과 같아야 합니다. 이를 멱등성(Idempotency)이라고 합니다.

이벤트에는 고유한 `event-id`를 둘 수 있습니다. Consumer는 이 값을 처리 이력 테이블에 저장하고, 이미 처리한 이벤트라면 작업을 건너뜁니다.

```text
1. event-id = evt-123 수신
2. 처리 이력에 evt-123이 있는지 확인
3. 없으면 비즈니스 처리 + evt-123 기록
4. 있으면 이미 처리한 이벤트로 판단하고 종료
```

구현할 때는 “조회 후 저장”을 단순히 분리하기보다, 이벤트 ID에 유니크 제약을 두는 것이 안전합니다. 여러 Consumer 인스턴스가 동시에 같은 메시지를 처리하려는 경쟁 상황도 막아야 하기 때문입니다.

```sql
create table processed_event (
  event_id varchar(64) primary key,
  processed_at datetime not null
);
```

또한 멱등성은 이벤트 전체가 아니라 실제 부작용까지 고려해야 합니다.

- DB 상태 변경은 이벤트 ID 또는 비즈니스 키를 기준으로 중복을 방지해야 합니다.
- 외부 API 호출은 상대 API의 idempotency key 지원 여부를 확인해야 합니다.
- 이메일·푸시 발송은 발송 요청 ID를 기록하거나 중복 발송 허용 여부를 결정해야 합니다.
- 재고 차감은 단순 누적 연산보다 요청 ID 기반의 이력과 상태 전이로 관리하는 편이 안전합니다.

## 정리

- Offset Commit은 Consumer가 어디까지 처리했는지 기록하는 작업입니다.
- Commit을 먼저 하면 메시지 유실 위험이 있고, 처리 후 Commit하면 중복 처리 가능성이 있습니다.
- 일반적인 비즈니스 이벤트는 At-least-once를 전제로 처리 후 Commit하는 방식이 안전합니다.
- Kafka의 Exactly-once는 외부 DB나 API 호출까지 정확히 한 번 실행해 주지 않습니다.
- Consumer는 event-id와 유니크 제약 등을 이용해 실제 부작용까지 멱등하게 처리해야 합니다.
