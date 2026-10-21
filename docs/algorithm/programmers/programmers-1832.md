## 🔗 문제 링크

<a class="bookmark-card" href="https://school.programmers.co.kr/learn/courses/30/lessons/1832" target="_blank" rel="noopener noreferrer">
  <div class="bookmark-card-title">보행자 천국</div>
  <div class="bookmark-card-desc">코딩테스트 연습 - 보행자 천국 문제입니다.</div>
  <div class="bookmark-card-url">
    <img class="bookmark-card-favicon" src="https://www.google.com/s2/favicons?domain=programmers.co.kr" alt="" />
    <span>school.programmers.co.kr</span>
  </div>
</a>

## 💡 문제 파악

- `m x n` 크기의 도시 지도에서 왼쪽 위 `(0, 0)` 에서 오른쪽 아래 `(m - 1, n - 1)` 까지 **오른쪽 또는 아래쪽으로만** 이동한다.
- `cityMap` 의 값은 다음과 같다.
  - `0`: 자동차가 자유롭게 지나갈 수 있는 도로
  - `1`: 자동차 통행 금지
  - `2`: 보행자 보호구역. 지나갈 수는 있지만 **방향을 바꿀 수 없다.** (오른쪽으로 들어왔으면 오른쪽으로, 아래로 들어왔으면 아래로 계속 직진)
- 가능한 모든 경로의 수를 `20170805` 로 나눈 나머지를 반환한다.

## 🗝️ 접근 방법

- 일반적인 격자 경로 DP(`dp[x][y] = dp[x-1][y] + dp[x][y-1]`)에서는 칸에 **어느 방향으로 들어왔는지**를 알 수 없다. 그런데 `2` 번 칸은 들어온 방향 그대로 나가야 하므로 방향 정보를 상태에 포함해야 한다.
  - `dpRight[x][y]`: `(x, y)` 에 **오른쪽으로 이동해서 도착**하는 경로의 수
  - `dpDown[x][y]`: `(x, y)` 에 **아래쪽으로 이동해서 도착**하는 경로의 수
- 시작점은 `dpRight[0][0] = 1` 로 둔다. 시작점에서는 어느 방향으로든 나갈 수 있으므로, 이후 전이에서 직진/방향 전환 모두 가능한 `0` 번 칸처럼 취급된다.
- 현재 칸이 `1` 이면 도착할 수 없으므로 건너뛰고, 아니면 이전 칸의 종류에 따라 전이한다.
  - 왼쪽 칸에서 오른쪽으로 들어오는 경우 (`dpRight[x][y]`)
    - 왼쪽 칸이 `2` 이면 직진만 가능하므로 `dpRight[x][y - 1]` 만 가져온다.
    - 왼쪽 칸이 `0` 이면 방향 전환이 가능하므로 `dpRight[x][y - 1] + dpDown[x][y - 1]` 을 가져온다.
  - 위쪽 칸에서 아래로 들어오는 경우 (`dpDown[x][y]`)
    - 위쪽 칸이 `2` 이면 `dpDown[x - 1][y]` 만, `0` 이면 `dpRight[x - 1][y] + dpDown[x - 1][y]` 를 가져온다.
- 합산할 때마다 `MOD` 로 나눈 나머지를 취하고, 도착점에서는 `dpRight + dpDown` 을 다시 `MOD` 로 나눈 값이 답이다.

```java
// 왼쪽 -> 오른쪽
if (y > 0 && cityMap[x][y - 1] != 1) {
    if (cityMap[x][y - 1] == 2) {
        dpRight[x][y] = dpRight[x][y - 1];
    } else {
        dpRight[x][y] = (dpRight[x][y - 1] + dpDown[x][y - 1]) % MOD;
    }
}
```

## 🧩 코드 구현

```java
import java.util.*;

class Solution {
    private final int MOD = 20170805;

    public int solution(int m, int n, int[][] cityMap) {
        int[][] dpRight = new int[m][n];
        int[][] dpDown = new int[m][n];

        dpRight[0][0] = 1;

        for (int x = 0; x < m; ++x) {
            for (int y = 0; y < n; ++y) {
                if (x == 0 && y == 0) continue;

                // 자동차 못감
                if (cityMap[x][y] == 1) continue;

                // 왼쪽 -> 오른쪽
                if (y > 0 && cityMap[x][y - 1] != 1) {
                    if (cityMap[x][y - 1] == 2) {
                        dpRight[x][y] = dpRight[x][y - 1];
                    } else {
                        dpRight[x][y] = (dpRight[x][y - 1] + dpDown[x][y - 1]) % MOD;
                    }
                }

                // 위쪽 -> 아래쪽
                if (x > 0 && cityMap[x - 1][y] != 1) {
                    if (cityMap[x - 1][y] == 2) {
                        dpDown[x][y] = dpDown[x - 1][y];
                    } else {
                        dpDown[x][y] = (dpRight[x - 1][y] + dpDown[x - 1][y]) % MOD;
                    }
                }
            }
        }
        return (dpRight[m - 1][n - 1] + dpDown[m - 1][n - 1]) % MOD;
    }
}
```
