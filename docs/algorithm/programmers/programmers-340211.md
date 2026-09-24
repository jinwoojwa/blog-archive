## 🔗 문제 링크

<a class="bookmark-card" href="https://school.programmers.co.kr/learn/courses/30/lessons/340211" target="_blank" rel="noopener noreferrer">
  <div class="bookmark-card-title">[PCCP 기출문제] 3번 / 충돌위험 찾기</div>
  <div class="bookmark-card-desc">코딩테스트 연습 - [PCCP 기출문제] 3번 / 충돌위험 찾기 문제입니다.</div>
  <div class="bookmark-card-url">
    <img class="bookmark-card-favicon" src="https://www.google.com/s2/favicons?domain=programmers.co.kr" alt="" />
    <span>school.programmers.co.kr</span>
  </div>
</a>

## 💡 문제 파악

- 여러 대의 로봇이 각자 정해진 경로(`routes`)를 따라 1초에 한 칸씩 이동한다.
- 이동하는 동안 같은 시간에 같은 지점에 2개 이상의 로봇이 있으면 충돌 위험이 1회 발생한 것으로 센다.
- 모든 로봇이 이동을 마칠 때까지 발생한 충돌 위험 횟수를 구하는 문제이다.

## 🗝️ 접근 방법

- 모든 로봇을 각자의 경로대로 한 칸씩 이동시키면서, `(시간, x좌표, y좌표)`를 키로 하는 맵에 방문 횟수를 누적한다.
- 경로점 사이를 이동할 때는 x좌표를 먼저 목표 지점까지 옮긴 뒤, y좌표를 옮기는 식으로 한 칸씩 순차적으로 이동시키며 매 시점의 위치를 기록한다.
- 모든 이동이 끝난 뒤, 같은 키에 2개 이상 기록된 경우의 수를 세면 그것이 곧 충돌 위험 횟수가 된다.

## 🧩 코드 구현

```java
import java.util.*;

class Solution {
    public int solution(int[][] points, int[][] routes) {
        Map<String, Integer> map = new HashMap<>();

        for (int[] route : routes) {
            int time = 0;

            int r = points[route[0] - 1][0];
            int c = points[route[0] - 1][1];

            addPositionCount(map, time, r, c);

            for (int i = 1; i < route.length; i++) {
                int[] next = points[route[i] - 1];
                int nr = next[0];
                int nc = next[1];

                while (r != nr) {
                    time++;
                    r += (nr > r) ? 1 : -1;
                    addPositionCount(map, time, r, c);
                }

                while (c != nc) {
                    time++;
                    c += (nc > c) ? 1 : -1;
                    addPositionCount(map, time, r, c);
                }
            }
        }

        int answer = 0;
        for (int cnt : map.values()) {
            if (cnt >= 2) answer++;
        }

        return answer;
    }

    private void addPositionCount(Map<String, Integer> map, int t, int r, int c) {
        String key = t + "," + r + "," + c;
        map.merge(key, 1, Integer::sum);
    }
}
```
