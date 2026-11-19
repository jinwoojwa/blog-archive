## 🔗 문제 링크

<a class="bookmark-card" href="https://school.programmers.co.kr/learn/courses/30/lessons/64062" target="_blank" rel="noopener noreferrer">
  <div class="bookmark-card-title">징검다리 건너기</div>
  <div class="bookmark-card-desc">코딩테스트 연습 - 징검다리 건너기 문제입니다.</div>
  <div class="bookmark-card-url">
    <img class="bookmark-card-favicon" src="https://www.google.com/s2/favicons?domain=programmers.co.kr" alt="" />
    <span>school.programmers.co.kr</span>
  </div>
</a>

## 💡 문제 파악

- 징검다리의 각 디딤돌에는 숫자가 적혀 있고(`stones[i]`), 한 명이 밟고 지나갈 때마다 그 숫자가 1씩 줄어듭니다. 숫자가 `0` 이 되면 더 이상 밟을 수 없습니다.
- 한 명씩 차례로 징검다리를 건너며, 다음 디딤돌이 `0` 이면 **한 번에 최대 `k` 칸**까지 건너뛸 수 있습니다.
- 징검다리를 건널 수 있는 **최대 인원 수**를 구합니다.
- `stones` 의 길이는 최대 200,000 이고 각 값은 최대 200,000,000 입니다.

## 🗝️ 접근 방법

- 먼저 `m` 명이 건널 수 있는지 판단하는 방법을 생각합니다. `m` 번째 사람이 건널 때, 디딤돌의 숫자는 `stones[i] - (m - 1)` 이므로 **`stones[i] < m` 인 디딤돌은 이미 `0` 이어서 밟을 수 없습니다.**
  - 한 번에 최대 `k` 칸까지 건너뛸 수 있으므로, 밟을 수 없는 디딤돌이 **연속으로 `k` 개 이상**이면 건널 수 없습니다.
  - 따라서 `m` 명이 건널 수 있는 조건은 "`stones[i] < m` 인 디딤돌이 연속으로 `k` 개 미만"입니다.
- 이 판단은 **단조성**이 있습니다. `m` 명이 건널 수 있다면 `m - 1` 명도 반드시 건널 수 있고, `m` 명이 건널 수 없다면 `m + 1` 명도 건널 수 없습니다.
  - 그래서 "건널 수 있는 최대 `m`" 을 **이분 탐색**으로 찾을 수 있습니다.
- 탐색 범위는 `0 ~ 200,000,000` 입니다.
  - `isPossible(mid)` 가 `true` 이면 `mid` 명은 가능하므로 `left = mid` 로 두고, `false` 이면 `right = mid - 1` 로 줄입니다.
  - `left` 를 올리는 방식이라 `mid` 를 올림(`(right - left + 1) / 2`)으로 계산해야 `left = mid` 일 때 무한 루프에 빠지지 않습니다.
- `isPossible` 은 디딤돌을 앞에서부터 한 번 훑으며, `stone < num` 이면 연속 개수(`skip`)를 늘리고 아니면 `0` 으로 초기화합니다. `skip` 이 `k` 에 도달하면 `false` 입니다.

```java
private boolean isPossible(int num, int[] stones, int k) {
    int skip = 0;
    for (int stone : stones) {
        if (stone < num) {
            if (++skip >= k) return false;
        }
        else skip = 0;
    }
    return true;
}
```

## ⏱️ 복잡도 분석

### 시간 복잡도

- 이분 탐색의 범위가 `0 ~ 2억` 이므로 `isPossible` 을 약 `log₂(2억) ≈ 28` 번 호출합니다.
- `isPossible` 은 디딤돌을 한 번씩만 확인하므로 `O(N)` 입니다.
- 따라서 전체 시간 복잡도는 **`O(N log M)`** 이고(`N` 은 디딤돌 개수, `M` 은 탐색 범위), 최대 크기에서는 `200,000 × 28 = 약 560만` 번입니다.

### 공간 복잡도

- 추가 배열 없이 변수 몇 개만 사용하므로 `O(1)` 입니다.

### 왜 이 방법으로 풀어야 하는가

- **실제로 한 명씩 건너는 과정을 시뮬레이션하는 방법**
  - 한 명이 건널 때마다 디딤돌을 훑으면 `O(N)` 이고, 건널 수 있는 인원이 최대 2억 명까지 늘어날 수 있으므로 `O(N × M)` (약 `4 × 10^13`) 으로 불가능합니다.
- **인원 수를 1부터 하나씩 늘려 가며 확인하는 방법**
  - 시뮬레이션보다는 낫지만, 답이 최대 2억이면 `isPossible` 을 2억 번 호출해야 해서 역시 `O(N × M)` 입니다.
  - 가능 여부가 단조적이므로 매번 탐색 범위를 절반으로 줄이는 이분 탐색이 적합하고, 호출 횟수를 `M` 에서 `log M` 으로 줄입니다.
- **크기 `k` 의 슬라이딩 윈도우 최댓값을 이용하는 방법**
  - 길이 `k` 인 구간마다 최댓값을 구하고, 그 중 **최솟값**이 답이 됩니다. 한 구간이 모두 `m` 보다 작아지는 순간이 건널 수 없는 순간이기 때문입니다. 덱을 쓰면 `O(N)` 으로 이분 탐색보다 더 빠릅니다.
  - 다만 구현이 복잡하고, 이분 탐색도 약 560만 번이면 충분히 시간 안에 끝납니다. 그래서 "가능한 최대값을 찾는" 형태가 그대로 드러나는 이분 탐색이 이해하기 쉬워 이 방법을 택했습니다.

## 🧩 코드 구현

```java
class Solution {
    public int solution(int[] stones, int k) {
        int left = 0, right = 200_000_000;
        
        while (left < right) {
            int mid = left + (right - left + 1) / 2;
            
            if (isPossible(mid, stones, k)) left = mid;
            else right = mid - 1;
        }
        
        return left;
    }
    
    private boolean isPossible(int num, int[] stones, int k) {
        int skip = 0;
        for (int stone : stones) {
            if (stone < num) {
                if (++skip >= k) return false;
            }
            else skip = 0;
        }
        return true;
    }
}
```
