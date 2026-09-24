## 🔗 문제 링크

<a class="bookmark-card" href="https://school.programmers.co.kr/learn/courses/30/lessons/340212" target="_blank" rel="noopener noreferrer">
  <div class="bookmark-card-title">[PCCP 기출문제] 2번 / 퍼즐 게임 챌린지</div>
  <div class="bookmark-card-desc">코딩테스트 연습 - [PCCP 기출문제] 2번 / 퍼즐 게임 챌린지 문제입니다.</div>
  <div class="bookmark-card-url">
    <img class="bookmark-card-favicon" src="https://www.google.com/s2/favicons?domain=programmers.co.kr" alt="" />
    <span>school.programmers.co.kr</span>
  </div>
</a>

## 💡 문제 파악

- 퍼즐마다 난이도(`diffs`)와 소요 시간(`times`)이 정해져 있고, 전체 제한 시간(`limit`)이 주어진다.
- 자신의 숙련도(`level`)가 퍼즐 난이도 이상이면 정해진 시간만큼만 걸리지만, 숙련도가 부족하면 `(직전 퍼즐 시간 + 현재 퍼즐 시간) × (난이도 - 숙련도) + 현재 퍼즐 시간`만큼의 시간이 추가로 걸린다.
- 모든 퍼즐을 제한 시간 안에 풀 수 있는 최소 숙련도를 구하는 문제이다.
- 퍼즐 개수는 최대 30만 개, 숙련도는 최대 10^15까지 가능해 모든 값을 직접 대입해볼 수는 없다.

## 🗝️ 접근 방법

- 숙련도가 높아질수록 전체 소요 시간은 단조 감소한다는 성질이 있으므로 이분 탐색을 사용할 수 있는 전형적인 문제이다.
- 숙련도 후보 구간 `[1, 100000]`(문제의 최대 난이도 범위)에서 이분 탐색을 수행하며, 특정 숙련도로 모든 퍼즐을 제한 시간 안에 풀 수 있는지를 판별하는 함수를 반복 호출한다.
- 풀 수 있다면 더 낮은 숙련도도 가능한지 확인하고, 풀 수 없다면 더 높은 숙련도를 시도하는 방향으로 탐색 범위를 좁혀 최소 숙련도를 찾는다.

## 🧩 코드 구현

```java
class Solution {
    public int solution(int[] diffs, int[] times, long limit) {
        int lvLeft = 1;
        int lvRight = 100_000;
        int lvMid = 0;

        while (lvLeft <= lvRight) {
            lvMid = (lvLeft + lvRight) / 2;

            if (!isPossibleToSolvePuzzles(diffs, times, lvMid, limit)) {
                lvLeft = lvMid + 1;
            } else {
                lvRight = lvMid - 1;
            }
        }
        return lvLeft;
    }

    private boolean isPossibleToSolvePuzzles(int[] diffs, int[] times, int level, long limit) {
        long solveTime = 0;
        for (int i = 0; i < diffs.length; ++i) {
            if (diffs[i] <= level) {
                solveTime += times[i];
            } else {
                solveTime += (long) (times[i] + times[i - 1]) * (diffs[i] - level) + times[i];
            }
        }
        return solveTime <= limit;
    }
}
```
