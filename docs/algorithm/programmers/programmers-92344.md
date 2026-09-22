## 🔗 문제 링크

<a class="bookmark-card" href="https://school.programmers.co.kr/learn/courses/30/lessons/92344" target="_blank" rel="noopener noreferrer">
  <div class="bookmark-card-title">파괴되지 않은 건물</div>
  <div class="bookmark-card-desc">코딩테스트 연습 - 파괴되지 않은 건물 문제입니다.</div>
  <div class="bookmark-card-url">
    <img class="bookmark-card-favicon" src="https://www.google.com/s2/favicons?domain=programmers.co.kr" alt="" />
    <span>school.programmers.co.kr</span>
  </div>
</a>

## 💡 문제 파악

- `N × M` 크기의 맵에 각 칸마다 내구도(`board`)를 가진 건물이 있다.
- 공격 스킬은 지정된 직사각형 범위의 내구도를 감소시키고, 회복 스킬은 지정된 범위의 내구도를 증가시킨다.
- 모든 스킬이 적용된 이후, 내구도가 1 이상인(파괴되지 않은) 건물의 개수를 구하는 문제이다.
- 스킬마다 범위 안의 모든 칸을 직접 순회하며 갱신하면 `O(K × N × M)`이 되어 시간 초과가 발생한다.

## 🗝️ 접근 방법

- 구간에 값을 더하고 최종 결과만 한 번에 구할 때 쓰는 2차원 누적합(diff array) 기법을 사용한다.
- 각 스킬의 영향 범위 `(r1, c1) ~ (r2, c2)`에 대해, 실제 칸을 채우는 대신 `diff` 배열의 네 꼭짓점에만 값을 표시해둔다.
  - `diff[r1][c1] += value`, `diff[r1][c2+1] -= value`, `diff[r2+1][c1] -= value`, `diff[r2+1][c2+1] += value`
- 모든 스킬을 처리한 뒤, 행 방향과 열 방향으로 각각 한 번씩 누적합을 구하면 각 칸에 실제로 적용된 변화량이 계산된다.
- 원래 `board` 값에 이 변화량을 더해서 1 이상인 칸의 개수를 센다.

## 🧩 코드 구현

```java
class Solution {

    int n, m;

    public int solution(int[][] board, int[][] skill) {
        n = board.length;
        m = board[0].length;

        int[][] diff = new int[n + 1][m + 1];

        applySkills(diff, skill);
        accumulate(diff);

        return countUndestroyed(board, diff);
    }

    private void applySkills(int[][] diff, int[][] skills) {
        for (int[] skill : skills) {
            int type = skill[0];
            int r1 = skill[1];
            int c1 = skill[2];
            int r2 = skill[3];
            int c2 = skill[4];
            int degree = skill[5];

            int value = (type == 1) ? -degree : degree;

            diff[r1][c1] += value;
            diff[r1][c2 + 1] -= value;
            diff[r2 + 1][c1] -= value;
            diff[r2 + 1][c2 + 1] += value;
        }
    }

    private void accumulate(int[][] diff) {
        for (int r = 0; r <= n; r++) {
            for (int c = 1; c <= m; c++) {
                diff[r][c] += diff[r][c - 1];
            }
        }

        for (int c = 0; c <= m; c++) {
            for (int r = 1; r <= n; r++) {
                diff[r][c] += diff[r - 1][c];
            }
        }
    }

    private int countUndestroyed(int[][] board, int[][] diff) {
        int count = 0;

        for (int r = 0; r < n; r++) {
            for (int c = 0; c < m; c++) {
                if (board[r][c] + diff[r][c] > 0) {
                    count++;
                }
            }
        }

        return count;
    }
}
```
