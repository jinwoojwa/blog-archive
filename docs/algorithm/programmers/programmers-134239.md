## 🔗 문제 링크

<a class="bookmark-card" href="https://school.programmers.co.kr/learn/courses/30/lessons/134239" target="_blank" rel="noopener noreferrer">
  <div class="bookmark-card-title">우박수열 정적분</div>
  <div class="bookmark-card-desc">코딩테스트 연습 - 우박수열 정적분 문제입니다.</div>
  <div class="bookmark-card-url">
    <img class="bookmark-card-favicon" src="https://www.google.com/s2/favicons?domain=programmers.co.kr" alt="" />
    <span>school.programmers.co.kr</span>
  </div>
</a>

## 💡 문제 파악

- 시작 숫자 `k`로 우박수열(콜라츠 수열)을 만들고, 이를 그래프 위에 순서대로 이은 꺾은선으로 나타낸다.
- 주어지는 여러 구간(`ranges`)에 대해, 그 구간에서의 정적분 값(그래프와 x축 사이의 넓이, 부호 포함)을 구하는 문제이다.
- 구간의 끝은 우박수열의 마지막 인덱스를 기준으로 한 음수 오프셋으로 주어질 수 있다.

## 🗝️ 접근 방법

- 먼저 `k`부터 시작해 1이 될 때까지 우박수열(짝수면 2로 나누고, 홀수면 3배 후 1을 더하는 규칙)을 전부 구한다.
- 인접한 두 값 사이의 정적분(사다리꼴의 넓이, `(이전 값 + 다음 값) / 2`)을 누적한 누적합 배열을 미리 구해두면, 임의의 구간에 대한 정적분을 매번 다시 계산하지 않고 두 누적합의 차로 바로 구할 수 있다.
- 각 구간에 대해 시작과 끝 인덱스가 같으면 `0`, 시작이 끝보다 크면(즉, 구간이 뒤집혀 있으면) `-1`을 반환하도록 처리한다.

## 🧩 코드 구현

```java
import java.util.*;

class Solution {
    public double[] solution(int k, int[][] ranges) {
        double[] answer = new double[ranges.length];

        int[] hailstoneSeq = findHailstoneNumber(k);

        double[] prefixSum = new double[hailstoneSeq.length];
        for (int i = 1; i < hailstoneSeq.length; ++i) {
            double integral = ((double) hailstoneSeq[i - 1] + hailstoneSeq[i]) / 2;
            prefixSum[i] = prefixSum[i - 1] + integral;
        }

        int n = hailstoneSeq.length - 1;
        for (int i = 0; i < ranges.length; ++i) {
            int st = ranges[i][0];
            int end = n + ranges[i][1];

            if (st > end) answer[i] = -1;
            else if (st == end) answer[i] = 0;
            else answer[i] = prefixSum[end] - prefixSum[st];
        }

        return answer;
    }

    private int[] findHailstoneNumber(int k) {
        List<Integer> hailstoneSeq = new ArrayList<>();
        hailstoneSeq.add(k);

        while (k != 1) {
            if (k % 2 == 0) k /= 2;
            else k = k * 3 + 1;
            hailstoneSeq.add(k);
        }
        return hailstoneSeq.stream()
            .mapToInt(Integer::intValue)
            .toArray();
    }
}
```
