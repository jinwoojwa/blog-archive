## Kafka 운영: Consumer Lag, Retry, DLQ

Kafka를 운영할 때는 메시지 적체와 처리 실패를 함께 관찰하고 대응해야 합니다. 이 글에서는 Consumer Lag, Retry, DLQ(Dead Letter Queue)를 중심으로 운영 전략을 정리합니다.

## 먼저 알아둘 용어

| 용어           | 설명                                                                                          |
| -------------- | --------------------------------------------------------------------------------------------- |
| **Topic**          | 이벤트를 종류별로 모아 두는 논리적 채널입니다. 예: `inventory-events`                         |
| **Partition**      | Topic을 나눈 단위입니다. 메시지는 Partition 안에 순서대로 쌓이며, **순서는 Partition 안에서만 보장**됩니다. |
| **Offset**         | Partition 안에서 각 메시지에 붙는 순번입니다.                                                 |
| **Key**            | 메시지를 어느 Partition에 넣을지 정하는 값입니다. **같은 Key는 항상 같은 Partition**으로 갑니다.  |
| **Consumer Group** | 같은 Topic을 나눠 읽는 Consumer 묶음입니다. **한 Partition은 같은 Group 안의 Consumer 하나만** 읽습니다. |

Offset과 Commit에 대한 자세한 내용은 [Kafka Consumer의 전달 보장과 멱등성](./kafka-consumer-delivery-guarantee.md)에서 다룹니다.

## Consumer Lag과 메시지 처리 성능

**Consumer Lag**은 Partition의 최신 Offset과 Consumer Group이 Commit한 Offset의 차이입니다. 즉 Kafka에 쌓인 메시지 중 Consumer가 아직 처리하지 못한 **메시지 개수**입니다. **시간이 아니라 건수**라는 점에 주의해야 합니다.

```text
Lag = 최신 Offset(Log End Offset) - Consumer Commit Offset
```

```text
Partition 최신 Offset: 10,000
Consumer Commit Offset: 9,200

Consumer Lag: 800
```

Lag은 `kafka-consumer-groups.sh --describe --group <group>` 명령이나 Prometheus 같은 모니터링 도구의 Kafka 메트릭으로 Partition별로 확인할 수 있습니다.

```text
GROUP      TOPIC             PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG
inventory  inventory-events  0          9200            10000           800
```

Lag 자체가 항상 문제는 아닙니다. 순간적으로 주문이 몰리면 일시적으로 증가할 수 있습니다. 다만 **계속 증가한다면 생산 속도가 소비 속도보다 빠르다**는 뜻입니다.

재고 이벤트가 계속 쌓이는 상황을 가정해 보겠습니다.

```text
Producer 발행 속도: 초당 1,000건
Consumer 처리 속도: 초당 600건

매초 Lag 증가량: 약 400건
```

Lag 건수는 처리 속도와 함께 봐야 의미가 보입니다. 이미 쌓인 Lag이 800건이고 Consumer가 초당 600건을 처리한다면 1~2초면 따라잡지만, 초당 2건을 처리한다면 400초가 걸립니다. 즉 `Lag ÷ 소비 속도`로 **해소 시간**을 추정할 수 있습니다.

이때 확인할 사항은 다음과 같습니다.

- Consumer의 비즈니스 처리나 외부 DB 조회가 느린지
- 특정 Partition에만 메시지가 몰렸는지
- Consumer 수가 Partition 수보다 적은지
- Broker 또는 네트워크에 병목이 있는지
- 재시도가 반복되어 처리 시간이 길어졌는지

대응 방법도 원인에 따라 달라집니다.

| 원인                      | 우선 고려할 대응                        |
| ------------------------- | --------------------------------------- |
| Consumer 처리 자체가 느림 | DB 쿼리, 외부 호출, 배치 처리 방식 개선 |
| Consumer 수 부족          | Partition 수 범위 안에서 Consumer 확장  |
| 특정 Key에 이벤트 집중    | Key 설계와 Partition 분산 검토          |
| 일시적인 트래픽 급증      | 처리량 확장, Lag 알림 기준 조정         |
| 반복 실패 메시지          | Retry 정책과 DLQ 확인                   |

Partition을 **무작정 늘리는 것이 항상 해결책은 아닙니다**. 같은 키의 순서 보장 범위가 달라지고, 키 쏠림이 있으면 Partition 수가 많아도 특정 Partition만 계속 밀릴 수 있습니다.

운영에서는 Topic별, Consumer Group별 Lag을 지속적으로 관찰해야 합니다. 특히 “Lag이 얼마나 큰가”뿐 아니라 **“얼마나 오래 증가하는가”**가 중요합니다.

## 메시지 처리 실패 대응: Retry와 DLQ

메시지 처리는 실패를 전제로 설계해야 합니다. 재고 DB가 잠시 느려질 수도 있고, 잘못된 데이터가 들어올 수도 있으며, 외부 서비스가 응답하지 않을 수도 있습니다.

### 일시적 오류와 영구적 오류를 구분합니다

**일시적 오류**는 잠시 뒤 다시 시도하면 성공할 가능성이 있습니다.

- DB 연결 일시 장애
- 네트워크 타임아웃
- 외부 서비스의 일시적인 `5xx` 오류

**영구적 오류**는 같은 입력으로 재시도해도 성공할 가능성이 낮습니다.

- 필수 필드 누락
- 지원하지 않는 이벤트 버전
- 이미 삭제된 참조 데이터
- 비즈니스 규칙 위반

모든 실패를 즉시 무한 재시도하면 장애가 더 커질 수 있습니다. 오류 종류에 따라 Retry와 DLQ를 분리해야 합니다.

### Retry와 Backoff

일시적 오류라면 바로 연속해서 재시도하기보다 일정 시간 기다리는 **Backoff**를 둡니다. Backoff는 재시도 사이의 대기 시간을 점점 늘려 가는 방식입니다.

```text
1차 실패 → 1초 후 재시도
2차 실패 → 5초 후 재시도
3차 실패 → 30초 후 재시도
최종 실패 → DLQ 이동
```

이 방식은 장애가 난 DB나 외부 서비스에 과도한 부하를 주지 않습니다. 대기 시간은 보통 1초, 2초, 4초처럼 지수적으로 늘리고, 여러 Consumer가 동시에 재시도하지 않도록 약간의 무작위 값(**Jitter**)을 더하기도 합니다.

Retry를 구현하는 방식은 크게 두 가지입니다.

| 방식                         | 동작                                                         | 장점                 | 단점                                              |
| ---------------------------- | ------------------------------------------------------------ | -------------------- | ------------------------------------------------- |
| **블로킹 재시도**            | 같은 Consumer가 대기한 뒤 같은 메시지를 다시 처리합니다.     | 순서가 유지됩니다.   | 재시도 동안 해당 Partition의 뒤 메시지가 모두 밀려 Lag이 증가합니다. |
| **별도 Retry Topic**(논블로킹) | 실패 메시지를 `-retry` Topic으로 보내고 다음 메시지를 처리합니다. | 처리량이 유지됩니다. | 같은 Key의 순서가 바뀔 수 있습니다.               |

재시도가 길어지면 Lag이 커지는 이유가 바로 블로킹 재시도에 있습니다. **순서가 중요하면 블로킹**을, **순서보다 처리량이 중요하면 Retry Topic**을 선택합니다.

다만 Retry는 순서와도 관련이 있습니다. 같은 키의 이벤트가 다음 순서로 들어왔다고 가정해 보겠습니다.

```text
재고 +10 → 예약 -3 → 출고 -2
```

중간의 `예약 -3` 처리에 실패했다고 해서 뒤의 `출고 -2`를 먼저 처리하면 **상태가 꼬일 수 있습니다**. 순서가 중요한 이벤트라면 실패 메시지를 건너뛰고 다음 메시지를 처리하는 정책을 신중히 선택해야 합니다.

[Transactional Outbox](./transactional-outbox-pattern.md)(DB 변경과 이벤트 기록을 같은 트랜잭션으로 묶고 별도로 발행하는 패턴)의 발행 규칙도 이 문제를 고려해야 합니다. 같은 파티션 키의 앞선 이벤트 발행이 실패하면 뒤의 이벤트를 섣불리 진행하지 않습니다. 순서를 지키는 대신 해당 이벤트 묶음의 처리가 잠시 멈출 수 있습니다.

### Dead Letter Queue, DLQ

정해진 횟수만큼 재시도해도 처리할 수 없는 메시지는 별도 Topic으로 옮겨 보관합니다. 이것이 **DLQ**입니다.

```text
inventory 이벤트 Topic
  └─ Consumer 처리 실패
       └─ Retry 실패
            └─ inventory 이벤트 DLQ
```

DLQ는 **실패 메시지를 버리는 장소가 아닙니다**. 원본 메시지, 실패 원인, 재시도 횟수, 최초 수신 시각 등을 남겨 두고 운영자가 원인을 분석하고 복구할 수 있게 하는 장소입니다.

DLQ에는 원본 메시지와 함께 다음과 같은 정보를 헤더나 본문에 남겨 둘 수 있습니다.

```json
{
  "originalTopic": "inventory-events",
  "originalPartition": 3,
  "originalOffset": 18452,
  "eventId": "evt-123",
  "errorClass": "IllegalArgumentException",
  "errorMessage": "unsupported event version: 3",
  "retryCount": 3,
  "firstReceivedAt": "2026-10-10T09:30:05Z",
  "payload": { "reservationId": "reservation-20261010-001" }
}
```

DLQ에 쌓인 메시지는 다음 절차로 다룰 수 있습니다.

1. 실패 원인을 확인합니다.
2. 데이터 문제라면 데이터를 수정하거나 이벤트 계약을 보완합니다.
3. 코드 문제라면 배포 후 재처리합니다.
4. 재처리해도 해결되지 않는 경우 비즈니스적으로 보정합니다.

DLQ 메시지를 본 Topic으로 **무작정 되돌리면 같은 실패가 반복**될 수 있습니다. 재처리 전에는 실패 원인이 해결되었는지 먼저 확인해야 합니다.

또한 Retry와 DLQ 재처리는 같은 메시지를 다시 처리하는 일입니다. 이미 일부 작업이 반영된 상태일 수 있으므로, Consumer는 `event-id` 기반의 **멱등 처리**를 갖추고 있어야 합니다. 자세한 내용은 [Kafka Consumer의 전달 보장과 멱등성](./kafka-consumer-delivery-guarantee.md)을 참고합니다.

## 정리

- **Consumer Lag**은 생산 속도와 소비 속도의 차이로 쌓이는 미처리 메시지 양입니다.
- Lag은 크기뿐 아니라 **지속적으로 증가하는지**와 원인을 함께 확인해야 합니다.
- **일시적 오류**는 Backoff를 둔 Retry로 대응하고, **영구적 오류**는 DLQ로 분리합니다.
- 순서가 중요한 이벤트에서는 실패 메시지를 건너뛰어 다음 메시지를 처리할지 신중히 결정해야 합니다.
- DLQ 메시지는 원인을 해결한 뒤 재처리하며, 무작정 원본 Topic으로 되돌리지 않습니다.
