## 🔗 문제 링크

<a class="bookmark-card" href="https://school.programmers.co.kr/learn/courses/30/lessons/169199" target="_blank" rel="noopener noreferrer">
  <div class="bookmark-card-title">리코쳇 로봇</div>
  <div class="bookmark-card-desc">코딩테스트 연습 - 리코쳇 로봇 문제입니다.</div>
  <div class="bookmark-card-url">
    <img class="bookmark-card-favicon" src="https://www.google.com/s2/favicons?domain=programmers.co.kr" alt="" />
    <span>school.programmers.co.kr</span>
  </div>
</a>

## 💡 문제 파악

- 격자 모양의 보드에서 로봇(`R`)이 목표 지점(`G`)에 **정확히 멈추도록** 하는 최소 이동 횟수를 구합니다.
- 빈 칸(`.`)과 장애물(`D`)이 있고, 로봇은 한 번 이동할 때 상/하/좌/우 중 한 방향을 골라 **장애물이나 벽에 부딪힐 때까지 미끄러집니다.**
- 목표 지점을 지나치기만 하는 것은 도달이 아니며, 그 칸에서 멈춰야 합니다.
- 도달할 수 없으면 `-1` 을 반환합니다.

## 🗝️ 접근 방법

- 이동 한 번의 비용이 모두 1이고 최소 횟수를 구하는 문제이므로 **BFS** 를 사용합니다.
  - 일반적인 BFS 가 한 칸씩 이동하는 것과 달리, 여기서는 **미끄러진 뒤 멈춘 위치**가 하나의 다음 상태(노드)가 됩니다.
- `dist[][]` 를 `-1` 로 초기화해 방문 여부와 이동 횟수를 함께 관리합니다.
  - 시작 위치 `dist[R] = 0` 으로 두고 큐에 넣습니다.
  - 멈춘 위치의 `dist` 가 이미 `0` 이상이면 이미 더 적거나 같은 횟수로 도달한 위치이므로 건너뜁니다.
- 4방향 각각에 대해 `canMove()` 가 `true` 인 동안 한 칸씩 전진시켜 멈추는 위치를 구합니다.
  - 다음 칸이 보드 밖이거나 `D` 이면 멈춥니다.
  - 이동하지 못하고 제자리에 멈추는 경우에는 `dist[cur]` 가 이미 설정되어 있어 자연스럽게 걸러집니다.
- 큐에서 꺼낸 위치가 `G` 이면 그때의 `dist` 를 반환합니다. BFS 이므로 처음 도달한 횟수가 최소입니다.
- 큐가 빌 때까지 도달하지 못하면 `-1` 을 반환합니다.

```java
// 4방향으로 부딪힐 때까지 이동
while (canMove(next, dir)) {
    next.x += dx[dir];
    next.y += dy[dir];
}

if (dist[next.x][next.y] >= 0) continue;

dist[next.x][next.y] = dist[cur.x][cur.y] + 1;
q.offer(next);
```

## ⏱️ 복잡도 분석

### 시간 복잡도

- 보드의 크기는 `n × m` (각각 최대 100) 이고, 로봇이 **멈출 수 있는 위치**는 최대 `n × m` 개입니다. 각 위치는 큐에 최대 한 번만 들어갑니다.
- 한 위치에서 4방향으로 미끄러지는데, 한 방향당 최대 `max(n, m)` 칸을 이동합니다.
- 따라서 전체 시간 복잡도는 **`O(n × m × (n + m))`** 이고, 최대 크기에서는 `100 × 100 × 4 × 100 = 약 400만` 번입니다.

### 공간 복잡도

- `dist` 배열과 큐가 각각 `O(n × m)` 입니다.

### 왜 이 방법으로 풀어야 하는가

- **DFS 로 모든 이동 경로를 탐색하는 방법**
  - 처음 도달한 경로가 최소 이동 횟수라는 보장이 없고, 방문 처리를 하지 않으면 같은 위치를 되돌아 무한히 순환할 수 있습니다.
  - 이동 한 번의 비용이 모두 `1` 이므로 **먼저 도달한 횟수가 곧 최소**인 BFS 가 적합합니다.
- **칸 하나씩 이동하는 일반 BFS 를 그대로 쓰는 방법**
  - 미끄러지는 도중에 지나가는 칸까지 상태로 만들면 "한 칸 이동 = 1" 로 거리가 계산되어 **이동 횟수가 아니라 이동한 칸 수**를 구하게 됩니다. 문제에서 원하는 값이 아닙니다.
  - 그래서 **멈춘 위치만 노드**로 두고, 한 번 미끄러지는 것을 간선 하나(비용 1)로 모델링해야 합니다.
- **다익스트라를 쓰는 방법**
  - 모든 간선의 비용이 `1` 로 같으므로 일반 큐로 충분하고, 우선순위 큐는 `log` 비용만 더합니다.

## 🧩 코드 구현

```java
import java.util.*;

class Solution {
    
    int n, m;
    int[] dx = {0, 1, 0, -1};
    int[] dy = {1, 0, -1, 0};
    String[] board;
    
    class Position {
        int x;
        int y;
        
        Position(int x, int y) {
            this.x = x;
            this.y = y;
        }
    }
    
    public int solution(String[] board) {
        this.n = board.length;
        this.m = board[0].length();
        this.board = board;
        
        Position robot = null;
        Position destination = null;
        
        for (int i = 0; i < n; ++i) {
            for (int j = 0; j < m; ++j) {
                char c = board[i].charAt(j);

                if (c == 'R') robot = new Position(i, j);
                else if (c == 'G') destination = new Position(i, j);
            }
        }
        
        Queue<Position> q = new ArrayDeque<>();
        
        int[][] dist = new int[n][m];
        for (int i = 0; i < n; ++i) {
            Arrays.fill(dist[i], -1);
        }
        
        q.offer(robot);
        dist[robot.x][robot.y] = 0;
        
        while (!q.isEmpty()) {
            Position cur = q.poll();
            
            if (cur.x == destination.x && cur.y == destination.y) return dist[cur.x][cur.y];
            
            for (int dir = 0; dir < 4; ++dir) {
                Position next = new Position(cur.x, cur.y);
                
                // 4방향으로 부딪힐 때까지 이동
                while (canMove(next, dir)) {
                    next.x += dx[dir];
                    next.y += dy[dir];
                }
                
                if (dist[next.x][next.y] >= 0) continue;
                
                dist[next.x][next.y] = dist[cur.x][cur.y] + 1;
                q.offer(next);
            }
        }
        
        return -1;
    }
    
    private boolean canMove(Position pos, int dir) {
        int nx = pos.x + dx[dir];
        int ny = pos.y + dy[dir];

        if (nx < 0 || ny < 0 || nx >= n || ny >= m) return false;
        if (board[nx].charAt(ny) == 'D') return false;
        
        return true;
    }
}
```
