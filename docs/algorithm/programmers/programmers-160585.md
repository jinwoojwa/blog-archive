## 🔗 문제 링크

<a class="bookmark-card" href="https://school.programmers.co.kr/learn/courses/30/lessons/160585" target="_blank" rel="noopener noreferrer">
  <div class="bookmark-card-title">혼자서 하는 틱택토</div>
  <div class="bookmark-card-desc">코딩테스트 연습 - 혼자서 하는 틱택토 문제입니다.</div>
  <div class="bookmark-card-url">
    <img class="bookmark-card-favicon" src="https://www.google.com/s2/favicons?domain=programmers.co.kr" alt="" />
    <span>school.programmers.co.kr</span>
  </div>
</a>

## 💡 문제 파악

- 완성된 3×3 틱택토 보드(`board`)가 주어졌을 때, 정상적인 규칙에 따라 진행되어 나올 수 있는 상황인지 판별하는 문제이다.
- O를 먼저 두고 O, X가 번갈아 두는 규칙, 그리고 승패가 결정된 이후에는 게임이 종료된다는 규칙을 위반하지 않았는지 확인해야 한다.

## 🗝️ 접근 방법

- 먼저 O와 X의 개수를 세어 규칙을 위반했는지 확인한다.
  - X가 O보다 많으면 안 된다.
  - O가 X보다 2개 이상 많으면 안 된다.
- 개수 조건을 통과했다면 승패 조건을 확인한다.
  - O가 X보다 하나 많은데 X가 이미 빙고를 만든 상태라면, X를 놓은 이후에 게임이 끝나지 않고 O를 하나 더 놓은 것이므로 잘못된 상황이다.
  - O와 X의 개수가 같은데 O가 이미 빙고를 만든 상태라면, O가 승리한 뒤에도 게임이 계속된 것이므로 잘못된 상황이다.
- 위 조건에 모두 걸리지 않으면 정상적인 상황(`1`)이다.

## 🧩 코드 구현

```java
class Solution {
    public int solution(String[] board) {
        int oCnt = 0, xCnt = 0;
        for (int i = 0; i < 3; ++i) {
            for (int j = 0; j < 3; ++j) {
                if (board[i].charAt(j) == 'O') oCnt++;
                else if (board[i].charAt(j) == 'X') xCnt++;
            }
        }

        if (xCnt > oCnt) return 0;
        if (oCnt - xCnt > 1) return 0;
        if (oCnt > xCnt && isWin(board, 'X')) return 0;
        if (xCnt == oCnt && isWin(board, 'O')) return 0;

        return 1;
    }

    private boolean isWin(String[] board, char c) {
        for (int row = 0; row < 3; ++row) {
            if (board[row].charAt(0) == c &&
                board[row].charAt(1) == c &&
                board[row].charAt(2) == c
            ) {
                return true;
            }
        }
        for (int col = 0; col < 3; ++col) {
            if (board[0].charAt(col) == c &&
                board[1].charAt(col) == c &&
                board[2].charAt(col) == c
            ) {
                return true;
            }
        }
        if (board[0].charAt(0) == c &&
            board[1].charAt(1) == c &&
            board[2].charAt(2) == c
        ) {
            return true;
        }
        if (board[0].charAt(2) == c &&
            board[1].charAt(1) == c &&
            board[2].charAt(0) == c
        ) {
            return true;
        }
        return false;
    }
}
```
