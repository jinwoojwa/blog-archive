## 🔗 문제 링크

<a class="bookmark-card" href="https://school.programmers.co.kr/learn/courses/30/lessons/12952" target="_blank" rel="noopener noreferrer">
  <div class="bookmark-card-title">N-Queen</div>
  <div class="bookmark-card-desc">코딩테스트 연습 - N-Queen 문제입니다.</div>
  <div class="bookmark-card-url">
    <img class="bookmark-card-favicon" src="https://www.google.com/s2/favicons?domain=programmers.co.kr" alt="" />
    <span>school.programmers.co.kr</span>
  </div>
</a>

## 💡 문제 파악

- `n × n` 크기의 체스판에 퀸 `n` 개를 서로 공격할 수 없도록 배치하는 **경우의 수**를 구합니다.
- 퀸은 같은 행, 같은 열, 같은 대각선에 있는 다른 퀸을 공격할 수 있습니다.
- `n` 은 12 이하의 자연수라 백트래킹으로 충분히 탐색할 수 있습니다.

## 🗝️ 접근 방법

- 퀸 `n` 개가 모두 서로 다른 행에 있어야 하므로, **행 단위로 한 줄에 퀸을 하나씩** 놓으면 행 충돌은 처음부터 생기지 않습니다.
  - `queen[row]` 에 `row` 행에 놓인 퀸의 열 번호를 저장합니다. 아직 놓지 않았으면 `-1` 입니다.
- `dfs(row, queen)` 은 다음과 같이 동작합니다.
  - `row == n` 이면 모든 행에 퀸을 놓았다는 뜻이므로 `answer` 를 1 증가시키고 반환합니다.
  - 그렇지 않으면 `row` 행의 모든 열 `i` 에 대해, 이미 놓인 위쪽 행(`0 ~ row - 1`)의 퀸들과 충돌하는지 검사합니다.
    - **같은 열**: `queen[j] == i`
    - **대각선**: 행 차이와 열 차이의 절댓값이 같습니다. `row - j == Math.abs(i - queen[j])`
  - 충돌하지 않으면 `queen[row] = i` 로 놓고 `dfs(row + 1, queen)` 을 호출한 뒤, **백트래킹**으로 `-1` 로 되돌립니다.
- 충돌하는 위치는 그 아래 행을 더 탐색하지 않고 바로 건너뛰므로(가지치기), 모든 배치를 다 만들어 보는 것보다 훨씬 적게 탐색합니다.

```java
for (int j = 0; j < row; ++j) {
    if (queen[j] == i) { // 같은 열
        possible = false;
        break;
    }
    if (row - j == Math.abs(i - queen[j])) { // 같은 대각선
        possible = false;
        break;
    }
}
```

## ⏱️ 복잡도 분석

### 시간 복잡도

- 행마다 퀸을 하나씩 놓고, 이미 놓인 열은 다시 쓸 수 없으므로 탐색하는 배치의 상한은 `n!` 입니다. 각 후보 위치는 위쪽 행들과 충돌 검사를 하므로 `O(n)` 이 듭니다.
- 이 상한은 매우 느슨합니다. `n = 12` 에서 `12! ≈ 4.8억` 이지만, 대각선 충돌이 생기는 순간 그 아래 행을 탐색하지 않는 **가지치기** 덕분에 실제 탐색 노드 수는 이보다 훨씬 적습니다.
- 그래서 제약인 `n ≤ 12` 에서 충분히 시간 안에 끝납니다.

### 공간 복잡도

- `queen` 배열이 `O(n)`, 재귀 깊이가 최대 `n` 이므로 `O(n)` 입니다.

### 왜 이 방법으로 풀어야 하는가

- **모든 배치를 만든 뒤 검사하는 방법**
  - 각 행에 열을 자유롭게 고르면 `n^n` 가지 (`12^12 ≈ 8.9조`) 이고, 체스판 칸 중 `n` 개를 고르는 경우는 더 많아 현실적으로 불가능합니다.
- **열이 겹치지 않는 순열을 모두 만든 뒤 대각선을 검사하는 방법**
  - 후보가 `n!` (`12! ≈ 4.8억`) 개로 줄지만, 끝까지 만든 뒤에야 검사하므로 이미 틀린 배치의 나머지 행도 전부 탐색하게 됩니다.
  - 퀸을 하나 놓을 때마다 바로 충돌을 검사하는 **백트래킹**이 틀린 가지를 일찍 잘라내므로 훨씬 적게 탐색합니다.
- **비트마스크나 열·대각선 방문 배열로 충돌을 `O(1)` 에 검사하는 방법**
  - 상수 시간 검사로 더 빠르게 만들 수 있지만, `n ≤ 12` 에서는 현재 방식으로도 충분합니다. 구현이 단순한 쪽을 택했습니다.

## 🧩 코드 구현

```java
import java.util.*;

class Solution {
    int n;
    int answer = 0;

    public int solution(int n) {
        this.n = n;

        int[] queen = new int[n];
        Arrays.fill(queen, -1);

        dfs(0, queen);

        return answer;
    }

    private void dfs(int row, int[] queen) {
        if (row == n) {
            answer++;
            return;
        }

        for (int i = 0; i < n; ++i) {
            boolean possible = true;
            for (int j = 0; j < row; ++j) {
                if (queen[j] == i) {
                    possible = false;
                    break;
                }
                if (row - j == Math.abs(i - queen[j])) {
                    possible = false;
                    break;
                }
            }

            if (!possible) continue;

            queen[row] = i;
            dfs(row + 1, queen);
            queen[row] = -1;
        }
    }
}
```
