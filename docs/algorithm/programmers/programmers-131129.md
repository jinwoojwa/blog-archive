## 🔗 문제 링크

<a class="bookmark-card" href="https://school.programmers.co.kr/learn/courses/30/lessons/131129" target="_blank" rel="noopener noreferrer">
  <div class="bookmark-card-title">카운트 다운</div>
  <div class="bookmark-card-desc">코딩테스트 연습 - 카운트 다운 문제입니다.</div>
  <div class="bookmark-card-url">
    <img class="bookmark-card-favicon" src="https://www.google.com/s2/favicons?domain=programmers.co.kr" alt="" />
    <span>school.programmers.co.kr</span>
  </div>
</a>

## 💡 문제 파악

- 다트 게임에서 목표 점수(`target`)를 정확히 0으로 만들어야 한다.
- 가능한 최소한의 다트 개수로 점수를 만들어야 하고, 다트 개수가 같다면 싱글/불을 맞춘 횟수가 더 많은 방법을 선택해야 한다.
- 최소 다트 개수와 그때의 싱글/불 횟수를 함께 반환하는 문제이다.

## 🗝️ 접근 방법

- 다트 한 번으로 만들 수 있는 모든 점수(싱글 1~20, 더블 2~40, 트리플 3~60, 불 50)를 미리 리스트로 만든다.
- `dp[i][0]`을 점수 `i`를 만드는 데 필요한 최소 다트 개수, `dp[i][1]`을 그때의 싱글/불 횟수로 정의한다.
- `1 ~ target`까지 각 점수 `i`에 대해, 만들 수 있는 모든 점수 옵션을 하나씩 빼보면서(`dp[i - score.point]`) 더 적은 다트 개수, 혹은 같은 다트 개수라면 더 많은 싱글/불 횟수로 갱신한다.

## 🧩 코드 구현

```java
import java.util.*;

class Solution {
    record Score(int point, int singleOrBull) {}

    public int[] solution(int target) {
        int[][] dp = new int[target + 1][2];

        for (int i = 1; i <= target; ++i) {
            dp[i][0] = 100001;
        }

        List<Score> scores = getScores();

        for (int i = 1; i <= target; ++i) {
            for (Score score : scores) {
                if (i < score.point()) continue;

                int throwCnt = dp[i - score.point()][0] + 1;
                int singleOrBullCnt = dp[i - score.point()][1] + score.singleOrBull();

                if (throwCnt < dp[i][0]) {
                    dp[i][0] = throwCnt;
                    dp[i][1] = singleOrBullCnt;
                } else if (throwCnt == dp[i][0] && singleOrBullCnt > dp[i][1]) {
                    dp[i][1] = singleOrBullCnt;
                }
            }
        }
        return new int[]{dp[target][0], dp[target][1]};
    }

    private List<Score> getScores() {
        List<Score> scores = new ArrayList<>();

        for (int i = 1; i <= 20; ++i) {
            scores.add(new Score(i, 1));       // 싱글
            scores.add(new Score(i * 2, 0));   // 더블
            scores.add(new Score(i * 3, 0));   // 트리플
        }
        scores.add(new Score(50, 1));          // 불

        return scores;
    }
}
```
