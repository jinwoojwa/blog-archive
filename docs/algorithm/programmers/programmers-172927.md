## 🔗 문제 링크

<a class="bookmark-card" href="https://school.programmers.co.kr/learn/courses/30/lessons/172927" target="_blank" rel="noopener noreferrer">
  <div class="bookmark-card-title">광물 캐기</div>
  <div class="bookmark-card-desc">코딩테스트 연습 - 광물 캐기 문제입니다.</div>
  <div class="bookmark-card-url">
    <img class="bookmark-card-favicon" src="https://www.google.com/s2/favicons?domain=programmers.co.kr" alt="" />
    <span>school.programmers.co.kr</span>
  </div>
</a>

## 💡 문제 파악

- 다이아몬드, 철, 돌 곡괭이의 개수(`picks`)와 캐야 하는 광물의 순서(`minerals`)가 주어진다.
- 곡괭이 하나를 선택하면 정해진 순서대로 5개의 광물을 연속으로 캐야 하고, 광물 종류와 곡괭이 종류의 조합에 따라 소모되는 피로도가 다르다.
- 모든 광물을 다 캐거나 모든 곡괭이를 다 쓰면 작업이 끝나며, 이때 필요한 최소 피로도를 구하는 문제이다.

## 🗝️ 접근 방법

- 광물이 주어지는 순서가 무작위이기 때문에 그리디하게 곡괭이를 고를 수 없다.
- `minerals`의 길이는 최대 50이므로, 5개씩 묶어도 최대 10묶음이다. 각 묶음마다 3종류의 곡괭이 중 하나를 고르는 모든 경우의 수는 `3^10`(약 6만) 수준이라 완전 탐색이 가능하다.
- `DFS`로 각 묶음에 어떤 곡괭이를 사용할지 순서대로 결정하면서, 그때마다 소모되는 피로도를 누적하고 최솟값을 갱신한다.
- 다이아몬드 곡괭이는 모든 광물을 피로도 1로, 철 곡괭이는 다이아몬드만 5, 나머지는 1로, 돌 곡괭이는 다이아몬드 25 · 철 5 · 돌 1로 계산한다.

## 🧩 코드 구현

```java
import java.util.*;

class Solution {
    static int result = 100_000_000;

    public int solution(int[] picks, String[] minerals) {
        dfs(picks, minerals, 0, 0);

        return result;
    }

    private static void dfs(int[] picks, String[] minerals, int idx, int fatigue) {
        if (idx >= minerals.length || useAllPicks(picks)) {
            result = Math.min(result, fatigue);
            return;
        }

        for (int i = 0; i < 3; ++i) {
            if (picks[i] <= 0) continue;

            picks[i]--;

            int possibleIdx = Math.min(idx + 5, minerals.length) - 1;
            int consumedFatigue = mineMinerals(idx, possibleIdx, minerals, i);
            dfs(picks, minerals, possibleIdx + 1, fatigue + consumedFatigue);

            picks[i]++;
        }
    }

    private static boolean useAllPicks(int[] picks) {
        for (int i = 0; i < 3; ++i) {
            if (picks[i] > 0) return false;
        }
        return true;
    }

    private static int mineMinerals(int st, int end, String[] minerals, int pickNum) {
        int fatigue = 0;
        for (int i = st; i <= end; ++i) {
            fatigue += mining(pickNum, minerals[i]);
        }
        return fatigue;
    }

    private static int mining(int pickNum, String mineral) {
        if (pickNum == 0) return 1;
        if (pickNum == 1) {
            if (mineral.equals("diamond")) return 5;
            return 1;
        }
        if (mineral.equals("diamond")) return 25;
        else if (mineral.equals("iron")) return 5;
        return 1;
    }
}
```
