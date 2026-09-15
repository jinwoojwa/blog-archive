## 🔗 문제 링크

<a class="bookmark-card" href="https://school.programmers.co.kr/learn/courses/30/lessons/152996" target="_blank" rel="noopener noreferrer">
  <div class="bookmark-card-title">시소 짝꿍</div>
  <div class="bookmark-card-desc">코딩테스트 연습 - 시소 짝꿍 문제입니다.</div>
  <div class="bookmark-card-url">
    <img class="bookmark-card-favicon" src="https://www.google.com/s2/favicons?domain=programmers.co.kr" alt="" />
    <span>school.programmers.co.kr</span>
  </div>
</a>

## 💡 문제 파악

- 사람들의 몸무게 배열 `weights` 가 주어진다. (몸무게는 100 ~ 1000 사이의 정수)
- 두 사람이 시소를 탔을 때 균형이 맞으려면, 두 사람의 몸무게 비율이 다음 중 하나여야 한다.
  - `1 : 1`
  - `2 : 3`
  - `1 : 2`
  - `3 : 4`
- 만들 수 있는 시소 짝꿍의 총 경우의 수를 구해야 한다. (순서는 상관없이, 같은 두 사람 조합은 한 번만 센다.)

## 🗝️ 접근 방법

- 몸무게가 100 ~ 1000 범위로 제한되어 있으므로, 모든 쌍을 직접 비교하지 않고 몸무게별 인원수를 세는 `count[1001]` 배열을 사용한다.
- 각 몸무게 `i` 에 대해, 짝이 될 수 있는 상대 몸무게를 비율별로 계산해서 조합 수를 더한다.
  - **1 : 1** — 같은 몸무게끼리는 `count[i]` 명 중 2명을 뽑는 조합이므로 `count[i] * (count[i] - 1) / 2`.
  - **2 : 3, 1 : 2, 3 : 4** — 상대 몸무게가 정수로 딱 떨어지는 경우에만(나머지가 0일 때만) `count[i] * count[상대 몸무게]` 를 더한다.
- 같은 쌍을 두 번 세지 않기 위해, 비율 계산은 항상 "더 무거운 쪽을 기준으로 더 가벼운 상대를 찾는" 방향으로만 진행한다.
  - 예를 들어 `2 : 3` 비율은 `i` 를 무거운 쪽(3)으로 보고 가벼운 쪽(`i * 2 / 3`)을 찾는 대신, 코드에서는 `i` 를 가벼운 쪽(2)으로 보고 무거운 쪽(`i * 3 / 2`)을 한 번만 찾는 식으로 방향을 고정해 중복 계산을 막는다.

```java
// 1 : 1
answer += count[i] * (count[i] - 1) / 2;

// 2 : 3
if (i * 3 / 2 <= 1000 && i * 3 % 2 == 0) {
    answer += count[i] * count[i * 3 / 2];
}
```

## 🧩 코드 구현

```java
import java.util.*;

class Solution {
    public long solution(int[] weights) {
        long answer = 0;
        
        long[] count = new long[1001];
        for (int i : weights) count[i]++;
        
        for (int i = 100; i <= 1000; ++i) {
            if (count[i] == 0) continue;
            
            // 1 : 1
            answer += count[i] * (count[i] - 1) / 2;
            
            // 2 : 3
            if (i * 3 / 2 <= 1000 && i * 3 % 2 == 0) {
                answer += count[i] * count[i * 3 / 2];
            }
            
            // 1 : 2
            if (i * 2 <= 1000) {
                answer += count[i] * count[i * 2];
            }
            
            // 3 : 4
            if (i * 4 / 3 <= 1000 && i * 4 % 3 == 0) {
                answer += count[i] * count[i * 4 / 3];
            }
        }
        return answer;
    }
}
```
