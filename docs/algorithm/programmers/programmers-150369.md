## 🔗 문제 링크

<a class="bookmark-card" href="https://school.programmers.co.kr/learn/courses/30/lessons/150369" target="_blank" rel="noopener noreferrer">
  <div class="bookmark-card-title">택배 배달과 수거하기</div>
  <div class="bookmark-card-desc">코딩테스트 연습 - 택배 배달과 수거하기 문제입니다.</div>
  <div class="bookmark-card-url">
    <img class="bookmark-card-favicon" src="https://www.google.com/s2/favicons?domain=programmers.co.kr" alt="" />
    <span>school.programmers.co.kr</span>
  </div>
</a>

## 💡 문제 파악

- 물류창고에서 일렬로 늘어선 `n`개의 집까지 택배를 배달하고, 동시에 빈 상자를 수거해서 다시 물류창고로 돌아와야 한다.
- 트럭은 한 번에 최대 `cap`개의 상자만 실을 수 있고, `i`번째 집은 물류창고에서 거리 `i`만큼 떨어져 있다.
- 배달과 수거를 모두 마치고 창고로 복귀하는 데 필요한 최소 이동 거리를 구해야 한다.

## 🗝️ 접근 방법

- 트럭은 어차피 가장 멀리 있는 미배달/미수거 집까지 갔다가 돌아와야 하므로, 항상 뒤쪽(먼 집)부터 처리하는 것이 유리하다.
  - 한 번 왕복할 때 가장 먼 집까지 갔다 오면서, 오는 길과 가는 길에 지나치는 다른 집들의 배달/수거도 여력이 되는 만큼 함께 처리할 수 있기 때문이다.
- `deliveryPointer`, `pickupPointer`를 각각 배달/수거가 남은 가장 먼 집의 인덱스로 두고, 둘 다 `-1`이 될 때까지 왕복을 반복한다.
  - 이미 처리(0)된 뒤쪽 집들은 포인터를 앞으로 당겨서 건너뛴다.
  - 이번 왕복에서 가야 하는 가장 먼 지점은 두 포인터 중 더 큰 쪽이므로, `(max(deliveryPointer, pickupPointer) + 1) * 2`(왕복 거리)를 정답에 더한다.
- `work` 메서드는 한 번의 편도 이동 동안 트럭에 실린 용량(`cap`)이 허용하는 만큼 뒤쪽 집부터 순서대로 배달(또는 수거)을 처리한다.
  - 현재 집에 필요한 양이 남은 용량보다 크면, 용량만큼만 처리하고 그 집에는 아직 처리할 물량이 남으므로 포인터는 그대로 둔다(용량 소진, `cap = 0`으로 반복 종료).
  - 남은 용량으로 충분하면 그 집을 완전히 처리하고 포인터를 한 칸 앞으로 옮긴다.
  - 배달은 창고 → 집 방향으로 이동할 때, 수거는 집 → 창고 방향으로 이동할 때 처리되는 것이므로 각각 독립적으로 `work`를 호출해도 두 이동은 같은 왕복 거리 안에서 함께 이뤄진다.
- 두 포인터가 모두 `-1`이 될 때까지, 즉 모든 배달과 수거가 끝날 때까지 이 과정을 반복하면 `answer`에 최소 이동 거리가 누적된다.

## 🧩 코드 구현

```java
class Solution {
    public long solution(int cap, int n, int[] deliveries, int[] pickups) {
        long answer = 0;

        int deliveryPointer = n - 1;
        int pickupPointer = n - 1;

        while (deliveryPointer >= 0 || pickupPointer >= 0) {
            while (deliveryPointer >= 0 && deliveries[deliveryPointer] == 0) deliveryPointer--;
            while (pickupPointer >= 0 && pickups[pickupPointer] == 0) pickupPointer--;

            answer += (Math.max(deliveryPointer, pickupPointer) + 1) * 2;

            // 택배 배달
            deliveryPointer = work(cap, deliveryPointer, deliveries);

            // 택배 수거
            pickupPointer = work(cap, pickupPointer, pickups);
        }
        return answer;
    }

    private int work(int cap, int pointer, int[] arr) {
        while (pointer >= 0 && cap > 0) {
            if (arr[pointer] > cap) {
                arr[pointer] -= cap;
                cap = 0;
            }
            else {
                cap -= arr[pointer];
                arr[pointer] = 0;
                pointer--;
            }
        }
        return pointer;
    }
}
```
