## 🔗 문제 링크

<a class="bookmark-card" href="https://school.programmers.co.kr/learn/courses/30/lessons/181188" target="_blank" rel="noopener noreferrer">
  <div class="bookmark-card-title">요격 시스템</div>
  <div class="bookmark-card-desc">코딩테스트 연습 - 요격 시스템 문제입니다.</div>
  <div class="bookmark-card-url">
    <img class="bookmark-card-favicon" src="https://www.google.com/s2/favicons?domain=programmers.co.kr" alt="" />
    <span>school.programmers.co.kr</span>
  </div>
</a>

## 💡 문제 파악

- 폭격 미사일이 `(s, e)` 형태의 열린 구간으로 여러 개 주어진다.
- 요격 미사일 하나를 특정 x좌표에 쏘면, 그 x좌표를 지나는 모든 폭격 미사일을 동시에 요격할 수 있다.
- 단, 구간의 양 끝점 `s`, `e`에서 쏜 요격 미사일로는 해당 폭격 미사일을 요격할 수 없다(열린 구간이므로).
- 모든 폭격 미사일을 요격하기 위한 최소한의 요격 미사일 개수를 구하는 문제이다.

## 🗝️ 접근 방법

- 전형적인 구간 스케줄링(그리디) 문제이다. `targets` 배열을 끝점(`e`) 기준으로 오름차순 정렬한다.
- 정렬된 순서대로 순회하면서, 직전에 쏜 요격 미사일이 현재 구간을 커버하지 못한다면 새로운 요격 미사일을 쏜다.
- 이때 새로 쏘는 위치는 현재 구간의 끝점보다 살짝 작은 값(`end - 1`)으로 잡아, 열린 구간 조건을 만족시키면서 최대한 많은 뒤따르는 구간까지 함께 커버되도록 한다.

## 🧩 코드 구현

```java
import java.util.*;

class Solution {
    public int solution(int[][] targets) {
        Arrays.sort(targets, (a, b) -> Integer.compare(a[1], b[1]));

        int missile = 0;
        int lastShot = -1;

        for (int[] target : targets) {
            int start = target[0];
            int end = target[1];

            if (lastShot < start) {
                missile++;
                lastShot = end - 1;
            }
        }

        return missile;
    }
}
```
