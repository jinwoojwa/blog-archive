## 🔗 문제 링크

<a class="bookmark-card" href="https://school.programmers.co.kr/learn/courses/30/lessons/12979" target="_blank" rel="noopener noreferrer">
  <div class="bookmark-card-title">기지국 설치</div>
  <div class="bookmark-card-desc">코딩테스트 연습 - 기지국 설치 문제입니다.</div>
  <div class="bookmark-card-url">
    <img class="bookmark-card-favicon" src="https://www.google.com/s2/favicons?domain=programmers.co.kr" alt="" />
    <span>school.programmers.co.kr</span>
  </div>
</a>

## 💡 문제 파악

- `N`개의 아파트가 일렬로 늘어서 있고, 일부 아파트 옥상에는 이미 기지국이 설치되어 있다.
- 기지국 하나를 설치하면 좌우로 `w`칸씩, 총 `2w + 1`칸의 전파를 전달할 수 있다.
- 모든 아파트가 전파를 받을 수 있도록 추가로 설치해야 하는 기지국의 최소 개수를 구하는 문제이다.
- `N`의 범위가 최대 2억이므로 `O(N)` 배열을 직접 순회하는 방식은 사용할 수 없다.

## 🗝️ 접근 방법

- 이미 설치된 기지국 정보를 이용해 전파가 닿지 않는 구간(빈 구역)만 먼저 구한다.
- 예를 들어 4번, 11번에 기지국이 설치되어 있고 `w = 1`이라면, 비어 있는 구간은 `1~2`, `6~9`가 된다.
- 각 구역의 길이를 하나의 기지국이 담당할 수 있는 범위인 `2w + 1`로 나누고 올림하면, 그 구역을 커버하는 데 필요한 기지국 개수가 된다.
- 모든 빈 구역에 대해 이 값을 더하면 정답이 된다.

## 🧩 코드 구현

```java
import java.util.*;

class Solution {
    public int solution(int n, int[] stations, int w) {
        int dist = 2 * w + 1;
        int apt = 1;
        int stApt = 0, enApt = 0;
        int answer = 0;

        for (int s : stations) {
            stApt = s - w;
            enApt = s + w;

            if (apt < stApt) {
                answer += calNeedStationNum(apt, stApt - 1, dist);
            }
            apt = enApt + 1;
        }

        if (enApt < n) {
            answer += calNeedStationNum(enApt + 1, n, dist);
        }

        return answer;
    }

    private int calNeedStationNum(int start, int end, int dist) {
        int aptCnt = end - start + 1;
        return (int) Math.ceil((double) aptCnt / dist);
    }
}
```
