## 🔗 문제 링크

<a class="bookmark-card" href="https://school.programmers.co.kr/learn/courses/30/lessons/389479" target="_blank" rel="noopener noreferrer">
  <div class="bookmark-card-title">서버 증설 횟수</div>
  <div class="bookmark-card-desc">코딩테스트 연습 - 서버 증설 횟수 문제입니다.</div>
  <div class="bookmark-card-url">
    <img class="bookmark-card-favicon" src="https://www.google.com/s2/favicons?domain=programmers.co.kr" alt="" />
    <span>school.programmers.co.kr</span>
  </div>
</a>

## 💡 문제 파악

- 0시부터 23시까지 시간대별 이용자 수 `players`가 주어진다.
- 서버 한 대는 이용자 `m`명을 감당할 수 있고, 한 번 증설하면 `k`시간 동안 운영된 뒤 자동으로 반납된다.
- 각 시간대마다 `(운영 중인 서버 수) x m`이 해당 시간대 이용자 수보다 많거나 같아질 때까지 서버를 증설해야 하며, 하루 동안 필요한 최소 증설 횟수를 구해야 한다.

## 🗝️ 접근 방법

- 0시부터 23시까지 순서대로 시뮬레이션하면서, 각 시간대에 반납되는 서버와 새로 증설해야 하는 서버를 처리한다.
- 증설된 서버의 반납 시각(`time + k`)을 큐에 저장해두고, 매 시간대 시작마다 큐의 맨 앞부터 확인해서 `반납 시각 <= 현재 시각`인 서버를 큐에서 제거하고 운영 중인 서버 수를 줄인다.
  - 큐에 반납 시각을 오름차순으로 넣기 때문에 맨 앞만 확인해도 반납 대상을 모두 처리할 수 있다.
- 반납 처리 후, `운영 중인 서버 수 x m`이 현재 이용자 수 이하인 동안 서버를 한 대씩 증설한다.
  - 증설할 때마다 반납 시각을 큐에 넣고, 운영 중인 서버 수와 전체 증설 횟수를 1씩 늘린다.
- 24개 시간대를 모두 순회한 뒤 누적된 증설 횟수를 반환한다.

## 🧩 코드 구현

```java
import java.util.*;

class Solution {
    public int solution(int[] players, int m, int k) {

        Queue<Integer> serverQ = new ArrayDeque<>();
        int runningServers = 1;
        int expansionCnt = 0;

        for (int time = 0; time <= 23; ++time) {
            int users = players[time];

            // 현재 시간대에 끝나는 서버 제거하기
            while (!serverQ.isEmpty() && serverQ.peek() <= time) {
                serverQ.poll();
                runningServers--;
            }

            // 현재 서버로 유저 감당 가능한 지 체크
            while (runningServers * m <= users) {
                runningServers++;
                serverQ.offer(time + k);
                expansionCnt++;
            }
        }
        return expansionCnt;
    }
}
```
