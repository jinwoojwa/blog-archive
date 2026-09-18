## 🔗 문제 링크

<a class="bookmark-card" href="https://school.programmers.co.kr/learn/courses/30/lessons/42891" target="_blank" rel="noopener noreferrer">
  <div class="bookmark-card-title">무지의 먹방 라이브</div>
  <div class="bookmark-card-desc">코딩테스트 연습 - 무지의 먹방 라이브 문제입니다.</div>
  <div class="bookmark-card-url">
    <img class="bookmark-card-favicon" src="https://www.google.com/s2/favicons?domain=programmers.co.kr" alt="" />
    <span>school.programmers.co.kr</span>
  </div>
</a>

## 💡 문제 파악

- 음식을 1번부터 순서대로, 1초에 1개씩 먹다가 남은 시간이 0인 음식은 건너뛴다.
- 마지막 음식까지 다 먹으면 다시 1번 음식으로 돌아가 반복한다.
- `k`초 후에 먹고 있을 음식의 번호를 구해야 하며, `k`초가 되기 전에 모든 음식을 다 먹어버리면 `-1`을 반환한다.
- `k`가 최대 `2 × 10^13`까지 주어질 수 있어, 매 초 하나씩 시뮬레이션하는 방식은 사용할 수 없다.

## 🗝️ 접근 방법

- 남아있는 모든 음식의 시간을 한 층씩 동시에 깎아나간다고 생각한다.
  - 예를 들어 `[3, 1, 2]`라면 `[2, 0, 1] → [1, 0, 0] → [0, 0, 0]` 순으로 줄어든다.
- 가장 적게 남은 음식부터 처리하기 위해 우선순위 큐를 사용한다.
  - 이전에 깎아낸 층 수를 `prev`, 남은 음식 개수를 `remain`이라 하면, 최소 시간의 음식까지 한 층을 깎는 데 필요한 시간은 `(cur - prev) * remain`이다.
  - `k`가 이 시간보다 크거나 같으면 그 층 전체를 실제로 깎아내고(시간이 0이 된 음식들은 큐에서 제거), 작으면 더 이상 한 층 전체를 깎을 수 없다는 뜻이므로 반복을 종료한다.
- 반복이 끝나면 남은 음식들을 원래 번호 순으로 정렬하고, `k % remain` 번째 음식을 찾아 반환한다.

## 🧩 코드 구현

```java
import java.util.*;

class Solution {
    public int solution(int[] food_times, long k) {
        long total = 0;
        for (int time : food_times) {
            total += time;
        }

        if (total <= k) {
            return -1;
        }

        PriorityQueue<int[]> pq = new PriorityQueue<>( // {time, idx}
            Comparator.comparingInt(arr -> arr[0])
        );

        for (int idx = 0; idx < food_times.length; ++idx) {
            pq.offer(new int[]{food_times[idx], idx + 1});
        }

        long prev = 0;
        long remain = food_times.length;

        while (!pq.isEmpty()) {
            long cur = pq.peek()[0];
            long spend = (cur - prev) * remain;

            if (k >= spend) {
                k -= spend;
                prev = cur;

                while (!pq.isEmpty() && pq.peek()[0] == cur) {
                    pq.poll();
                    remain--;
                }
            } else {
                break;
            }
        }

        List<int[]> foods = new ArrayList<>(pq);
        foods.sort(Comparator.comparingInt(a -> a[1]));

        return foods.get((int) (k % remain))[1];
    }
}
```
