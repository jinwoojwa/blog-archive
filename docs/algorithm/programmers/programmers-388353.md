## 🔗 문제 링크

<a class="bookmark-card" href="https://school.programmers.co.kr/learn/courses/30/lessons/388353" target="_blank" rel="noopener noreferrer">
  <div class="bookmark-card-title">지게차와 크레인</div>
  <div class="bookmark-card-desc">코딩테스트 연습 - 지게차와 크레인 문제입니다.</div>
  <div class="bookmark-card-url">
    <img class="bookmark-card-favicon" src="https://www.google.com/s2/favicons?domain=programmers.co.kr" alt="" />
    <span>school.programmers.co.kr</span>
  </div>
</a>

## 💡 문제 파악

- `n × m` 크기의 물류창고에 알파벳으로 표시된 컨테이너들이 쌓여 있다.
- 지게차는 창고 외부와 연결된(상하좌우로 한 번에 닿을 수 있는) 컨테이너만 꺼낼 수 있고, 크레인은 어떤 위치에 있든 상관없이 꺼낼 수 있다.
- 요청이 길이 2면 크레인, 길이 1이면 지게차 요청이다.
- 모든 요청을 순서대로 처리한 뒤 창고에 남은 컨테이너의 개수를 구하는 문제이다.

## 🗝️ 접근 방법

- 크레인 요청은 간단하다. 창고 전체를 순회하면서 해당 알파벳 컨테이너를 모두 제거하면 된다.
- 지게차 요청은 외부와 연결되어 있는지를 판단하는 과정이 필요하다.
  - 기존 `storage`를 한 칸씩 감싸는 `board`를 만들어, `(0, 0)`(외부)에서 `BFS`를 수행하면서 빈 칸(`.`)들을 모두 찾아 외부와 연결되어 있음을 표시한다.
  - 이후 창고를 순회하면서, 요청받은 알파벳 컨테이너의 상하좌우 중 하나라도 외부와 연결된 칸이면 지게차로 꺼낼 수 있다고 판단한다.
  - 꺼낼 컨테이너들을 리스트에 모아두었다가 한 번에 제거한다.

## 🧩 코드 구현

```java
import java.util.*;

class Solution {
    int h, w;
    int[] dx = {0, 1, 0, -1};
    int[] dy = {1, 0, -1, 0};

    public int solution(String[] storage, String[] requests) {
        h = storage.length + 2;
        w = storage[0].length() + 2;

        char[][] board = initBoard(storage);

        for (String request : requests) {
            char alphabet = request.charAt(0);

            if (request.length() == 2) {
                crane(board, alphabet);
            } else {
                forklift(board, alphabet);
            }
        }
        return countRemain(board);
    }

    private char[][] initBoard(String[] storage) {
        char[][] board = new char[h][w];

        for (int r = 0; r < h; r++) Arrays.fill(board[r], '.');

        for (int r = 0; r < storage.length; r++) {
            for (int c = 0; c < storage[0].length(); c++) {
                board[r + 1][c + 1] = storage[r].charAt(c);
            }
        }
        return board;
    }

    private void crane(char[][] board, char alphabet) {
        for (int r = 1; r < h - 1; r++) {
            for (int c = 1; c < w - 1; c++) {
                if (board[r][c] == alphabet) {
                    board[r][c] = '.';
                }
            }
        }
    }

    private void forklift(char[][] board, char alphabet) {
        boolean[][] outside = findOutside(board);
        List<int[]> removeList = new ArrayList<>();

        for (int r = 1; r < h - 1; r++) {
            for (int c = 1; c < w - 1; c++) {
                if (board[r][c] != alphabet) continue;

                for (int d = 0; d < 4; d++) {
                    int nr = r + dx[d];
                    int nc = c + dy[d];

                    if (outside[nr][nc]) {
                        removeList.add(new int[]{r, c});
                        break;
                    }
                }
            }
        }
        for (int[] pos : removeList) {
            board[pos[0]][pos[1]] = '.';
        }
    }

    private boolean[][] findOutside(char[][] board) {
        boolean[][] visited = new boolean[h][w];

        Queue<int[]> q = new ArrayDeque<>();
        q.offer(new int[]{0, 0});
        visited[0][0] = true;

        while (!q.isEmpty()) {
            int[] cur = q.poll();

            int x = cur[0];
            int y = cur[1];

            for (int d = 0; d < 4; d++) {
                int nx = x + dx[d];
                int ny = y + dy[d];

                if (nx < 0 || nx >= h || ny < 0 || ny >= w) continue;
                if (visited[nx][ny]) continue;
                if (board[nx][ny] != '.') continue;

                visited[nx][ny] = true;
                q.offer(new int[]{nx, ny});
            }
        }
        return visited;
    }

    private int countRemain(char[][] board) {
        int count = 0;

        for (int r = 1; r < h - 1; r++) {
            for (int c = 1; c < w - 1; c++) {
                if (board[r][c] != '.') count++;
            }
        }
        return count;
    }
}
```
