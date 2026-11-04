## 🔗 문제 링크

<a class="bookmark-card" href="https://school.programmers.co.kr/learn/courses/30/lessons/12938" target="_blank" rel="noopener noreferrer">
  <div class="bookmark-card-title">최고의 집합</div>
  <div class="bookmark-card-desc">코딩테스트 연습 - 최고의 집합 문제입니다.</div>
  <div class="bookmark-card-url">
    <img class="bookmark-card-favicon" src="https://www.google.com/s2/favicons?domain=programmers.co.kr" alt="" />
    <span>school.programmers.co.kr</span>
  </div>
</a>

## 💡 문제 파악

- 자연수 `n` 개로 이루어진 집합 중, 원소의 합이 `s` 이면서 **원소들의 곱이 최대**인 집합을 구합니다.
- 집합은 오름차순으로 정렬해서 반환하고, 조건을 만족하는 집합이 없으면 `[-1]` 을 반환합니다.
- 원소는 자연수이므로 각 원소는 1 이상이어야 합니다.

## 🗝️ 접근 방법

- 합이 일정할 때 곱이 최대가 되려면 원소들이 **서로 최대한 가깝게(균등하게)** 분배되어야 합니다.
  - 예를 들어 합이 10이고 원소가 2개일 때, `1 × 9 = 9`, `2 × 8 = 16`, `3 × 7 = 21`, `4 × 6 = 24`, `5 × 5 = 25` 로 차이가 작을수록 곱이 커집니다.
- 따라서 `s / n` 을 모든 원소의 기본값으로 두고, 나머지 `s % n` 만큼을 원소에 1씩 나누어 더합니다.
  - 나머지를 한 원소에 몰아주면 균등함이 깨지므로, **서로 다른 원소에 1씩** 더합니다.
  - 오름차순으로 반환해야 하므로 뒤쪽 원소부터 1씩 더하면 별도의 정렬이 필요 없습니다.
- `s / n` 이 1보다 작으면(`s < n`) 모든 원소를 자연수로 채울 수 없으므로 `[-1]` 을 반환합니다.

```java
int eachNum = s / n;
int remain = s % n;

for (int i = 0; i < n; ++i) {
    answer[i] = eachNum;
}
for (int i = 0; i < remain; ++i) {
    answer[n - 1 - i]++;
}
```

## ⏱️ 복잡도 분석

### 시간 복잡도

- 모든 원소를 `s / n` 으로 채우는 데 `O(n)`, 나머지 `s % n` 만큼 1씩 더하는 데 최대 `O(n)` (나머지는 항상 `n` 보다 작습니다) 이므로 전체 시간 복잡도는 **`O(n)`** 입니다.
- `n` 은 최대 10,000 이므로 약 2만 번 이내의 연산으로 끝납니다. `s` 의 크기(최대 1억)와는 무관합니다.

### 공간 복잡도

- 결과를 담는 배열 하나만 사용하므로 `O(n)` 입니다.

### 왜 이 방법으로 풀어야 하는가

- **합이 `s` 인 모든 집합을 만들어 곱을 비교하는 방법**
  - 합이 `s` 인 `n` 개의 자연수 조합 수는 `C(s - 1, n - 1)` 로, 제약 범위에서는 셀 수 없을 만큼 많아 불가능합니다.
- **`1` 에서 시작해 `s` 가 될 때까지 가장 작은 원소에 1씩 더하는 방법(그리디 시뮬레이션)**
  - 결과는 같지만 `s - n` 번 반복해야 해서 `O(s)` (최대 1억 번) 입니다. 우선순위 큐를 쓰면 `O(s log n)` 으로 더 느려집니다.
  - 어차피 최종 결과가 "`s / n` 과 `s / n + 1` 만 존재한다"는 것을 알고 있으므로, 나눗셈과 나머지로 **한 번에 계산**하는 것이 가장 효율적입니다.
- **DP (`dp[개수][합]`) 로 곱의 최댓값을 구하는 방법**
  - 상태 수가 `n × s` (최대 `10^12`) 라 메모리와 시간 모두 불가능합니다.
  - 곱이 최대가 되는 형태가 수학적으로 정해져 있으므로(균등 분배) 탐색 자체가 필요 없습니다.

## 🧩 코드 구현

```java
import java.util.*;

class Solution {
    public int[] solution(int n, int s) {
        int eachNum = s / n;
        int remain = s % n;
        
        int[] answer = new int[n];
        for (int i = 0; i < n; ++i) {
            answer[i] = eachNum;
        }
        for (int i = 0; i < remain; ++i) {
            answer[n - 1 - i]++;
        }
        
        if (eachNum < 1) return new int[]{-1};
        return answer;
    }
}
```
