## 🔗 문제 링크

<a class="bookmark-card" href="https://school.programmers.co.kr/learn/courses/30/lessons/12905" target="_blank" rel="noopener noreferrer">
  <div class="bookmark-card-title">가장 큰 정사각형 찾기</div>
  <div class="bookmark-card-desc">코딩테스트 연습 - 가장 큰 정사각형 찾기 문제입니다.</div>
  <div class="bookmark-card-url">
    <img class="bookmark-card-favicon" src="https://www.google.com/s2/favicons?domain=programmers.co.kr" alt="" />
    <span>school.programmers.co.kr</span>
  </div>
</a>

## 💡 문제 파악

- 0과 1로 채워진 표(board)가 주어진다.
- 표 안에서 1로만 이루어진 가장 큰 정사각형을 찾아, 그 넓이를 반환하는 문제이다.

## 🗝️ 접근 방법

- 전형적인 DP 문제로, `dp[row][col]`을 `(row, col)`을 오른쪽 아래 꼭짓점으로 하는 정사각형의 최대 한 변의 길이로 정의한다.
- `board[row][col]`이 1이라면, 왼쪽(`←`), 위(`↑`), 왼쪽 위 대각선(`↖`) 세 칸의 `dp` 값 중 최솟값에 1을 더한 값이 `dp[row][col]`이 된다.
- 세 방향 중 하나라도 정사각형을 이어갈 수 없으면 그 칸에서 한 변의 길이가 제한되기 때문이다.
- 계산 도중 나온 `dp` 값의 최댓값을 저장해두고, 마지막에 제곱해서 넓이를 구한다.

## 🧩 코드 구현

```java
class Solution {
    public int solution(int[][] board) {
        int h = board.length;
        int w = board[0].length;

        int[][] dp = new int[h + 1][w + 1];

        int answer = 0;
        for (int row = 1; row <= h; ++row) {
            for (int col = 1; col <= w; ++col) {
                if (board[row - 1][col - 1] == 0) continue;

                dp[row][col] = Math.min(
                    dp[row - 1][col - 1],
                    Math.min(dp[row][col - 1], dp[row - 1][col])
                ) + 1;

                answer = Math.max(answer, dp[row][col]);
            }
        }
        return answer * answer;
    }
}
```
