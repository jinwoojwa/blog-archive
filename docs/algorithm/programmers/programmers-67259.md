## 🔗 문제 링크

<a class="bookmark-card" href="https://school.programmers.co.kr/learn/courses/30/lessons/67259" target="_blank" rel="noopener noreferrer">
  <div class="bookmark-card-title">경주로 건설</div>
  <div class="bookmark-card-desc">코딩테스트 연습 - 경주로 건설 문제입니다.</div>
  <div class="bookmark-card-url">
    <img class="bookmark-card-favicon" src="https://www.google.com/s2/favicons?domain=programmers.co.kr" alt="" />
    <span>school.programmers.co.kr</span>
  </div>
</a>

## 💡 문제 파악

- `n x n` 크기의 지도 `board` 위에 `(0, 0)` 에서 `(n-1, n-1)` 까지 경주로를 건설한다.
- `board[i][j] == 1` 인 칸은 벽이라 지나갈 수 없다.
- 직선 도로는 `100원`, 코너(방향이 꺾이는) 도로는 `500원`이 추가로 든다.
- 여러 경로 중 건설 비용이 최소가 되는 값을 구해야 한다.

## 🗝️ 접근 방법

- 같은 칸이라도 **어느 방향으로 도착했는지**에 따라 이후 코너 비용이 달라지므로, 좌표뿐 아니라 방향까지 상태로 가지는 `cost[n][n][4]` 배열로 최소 비용을 관리한다.
- 시작점 `(0, 0)`을 방향 없음(`-1`) 상태로 큐에 넣고, 4방향으로 이동하며 탐색한다.
  - 직전 방향과 다음 방향이 다르면 코너 비용(`500`)을 더하고, 같으면 직선 비용(`100`)만 더한다.
- 새로 계산한 비용이 기존에 기록된 `cost[nx][ny][dir]` 보다 작을 때만 갱신하고 큐에 다시 넣는다.
  - 우선순위 큐 없이 일반 큐(`ArrayDeque`)를 쓰지만, 더 나은 비용이 나올 때마다 재방문을 허용해 값이 계속 줄어드는 방식(SPFA와 유사)으로 동작해 최종적으로 최소 비용에 수렴한다.
- 도착점 `(n-1, n-1)`에 대해 4방향 중 가장 작은 비용이 정답이다.

```java
int nextCost = cur.cost + STRAIGHT;

if (cur.d != -1 && cur.d != dir) {
    nextCost += CORNER;
}

if (nextCost >= cost[nx][ny][dir]) continue;
```

## 🧩 코드 구현

```java
import java.util.*;

class Solution {
    int n;
    int[] dx = {0, 1, 0, -1};
    int[] dy = {1, 0, -1, 0};
    
    static final int STRAIGHT = 100;
    static final int CORNER = 500;
    static final int BIG = Integer.MAX_VALUE;
    
    class Move {
        int d;
        int x;
        int y;
        int cost;
        
        Move(int d, int x, int y, int cost) {
            this.d = d;
            this.x = x;
            this.y = y;
            this.cost = cost;
        }
    }
    
    public int solution(int[][] board) {
        n = board.length;
        
        int[][] costMap = new int[n][n];
        int[][][] cost = new int[n][n][4];

        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                Arrays.fill(cost[i][j], BIG);
            }
        }
        
        Queue<Move> q = new ArrayDeque<>();
        q.offer(new Move(-1, 0, 0, 0));
        
        while (!q.isEmpty()) {
            Move cur = q.poll();
            
            for (int dir = 0; dir < 4; ++dir) {
                int nx = cur.x + dx[dir];
                int ny = cur.y + dy[dir];
                
                if (nx < 0 || ny < 0 || nx >= n || ny >= n) continue;
                if (board[nx][ny] == 1) continue;
                
                int nextCost = cur.cost + STRAIGHT;
                
                // 방향이 바뀌면 코너 비용 추가
                if (cur.d != -1 && cur.d != dir) {
                    nextCost += CORNER;
                }
                
                if (nextCost >= cost[nx][ny][dir]) continue;

                cost[nx][ny][dir] = nextCost;
                q.offer(new Move(dir, nx, ny, nextCost));
            }
        }
        int answer = BIG;

        for (int dir = 0; dir < 4; dir++) {
            answer = Math.min(answer, cost[n - 1][n - 1][dir]);
        }

        return answer;
    }
}
```
