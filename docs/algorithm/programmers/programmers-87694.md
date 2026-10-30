## 🔗 문제 링크

<a class="bookmark-card" href="https://school.programmers.co.kr/learn/courses/30/lessons/87694" target="_blank" rel="noopener noreferrer">
  <div class="bookmark-card-title">아이템 줍기</div>
  <div class="bookmark-card-desc">코딩테스트 연습 - 아이템 줍기 문제입니다.</div>
  <div class="bookmark-card-url">
    <img class="bookmark-card-favicon" src="https://www.google.com/s2/favicons?domain=programmers.co.kr" alt="" />
    <span>school.programmers.co.kr</span>
  </div>
</a>

## 💡 문제 파악

- 여러 직사각형이 겹쳐 있을 때, 이들을 합친 도형의 **바깥 테두리**만을 따라 이동할 수 있습니다.
- `rectangle[i] = [좌하단 x, 좌하단 y, 우상단 x, 우상단 y]` 이고, 좌표는 1 이상 50 이하입니다.
- 캐릭터 위치 `(characterX, characterY)` 에서 아이템 위치 `(itemX, itemY)` 까지 테두리를 따라 이동하는 **최단 거리**를 구합니다.

## 🗝️ 접근 방법

- 단순히 좌표를 그대로 쓰면 **두 직사각형이 한 칸 떨어져 평행하게 붙어 있는 경우**를 구분할 수 없다. 예를 들어 `x = 3` 과 `x = 4` 의 테두리는 실제로는 맞닿아 있지 않지만, 격자에서는 인접한 칸이라 이동 가능한 것으로 잘못 판단합니다.
  - 그래서 **모든 좌표를 2배로 확대**합니다. 이렇게 하면 떨어져 있는 테두리 사이에 빈 칸이 생겨 서로 연결되지 않습니다. (격자 크기는 `101 x 101`)
- 두 개의 배열로 도형을 표시합니다.
  - `map`: 직사각형의 **테두리와 내부를 모두 포함**한 칸
  - `inside`: 직사각형의 **내부**(테두리 제외)인 칸
  - 어떤 칸이 어떤 직사각형의 내부라면, 다른 직사각형의 테두리 위에 있더라도 합친 도형에서는 바깥 테두리가 아닙니다.
- 따라서 이동 가능한 칸은 `map[x][y] == true` 이면서 `inside[x][y] == false` 인 칸입니다. 이 칸들만 이어서 **BFS** 를 수행하면 최단 거리를 구할 수 있습니다.
- 좌표를 2배로 늘렸기 때문에 BFS 로 구한 거리도 2배가 됩니다. 그래서 답은 `dist / 2` 로 반환합니다.

```java
// 테두리 + 내부 모두 표시
for (int x = x1; x <= x2; ++x)
    for (int y = y1; y <= y2; ++y)
        map[x][y] = true;

// 내부만 따로 표시
for (int x = x1 + 1; x < x2; ++x)
    for (int y = y1 + 1; y < y2; ++y)
        inside[x][y] = true;
```

## ⏱️ 복잡도 분석

### 시간 복잡도

- 좌표를 2배로 늘렸으므로 격자의 한 변은 최대 `101` 칸이고, 전체 칸 수는 `101 × 101 = 10,201` 개입니다. 이를 `V` 라고 하겠습니다.
- 직사각형을 격자에 표시하는 데 직사각형 하나당 최대 `100 × 100` 번의 연산이 필요합니다. 직사각형은 최대 4개이므로 약 `40,000` 번입니다.
- BFS 는 각 칸을 큐에 **최대 한 번**만 넣고, 칸마다 4방향만 확인하므로 `O(4V)` 입니다. 약 `40,000` 번입니다.
- 따라서 전체 시간 복잡도는 `O(R × W² + V)` 이고(`R` 은 직사각형 개수, `W` 는 좌표 범위), 이 문제의 제약에서는 **약 10만 번 이내의 연산**으로 끝납니다.

### 공간 복잡도

- `map`, `inside`, `dist` 가 각각 `101 × 101` 크기이므로 `O(V)` 입니다.

### 왜 이 방법으로 풀어야 하는가

- **BFS 대신 DFS 로 모든 경로를 탐색하는 방법**
  - 테두리를 따라 이동하는 경로는 여러 갈래가 있을 수 있고, DFS 는 처음 도달한 경로가 최단이라는 보장이 없습니다. 최단 거리를 얻으려면 모든 경로를 끝까지 비교해야 해서 방문 처리 없이는 경로 수가 폭발적으로 늘어납니다.
  - 이동 비용이 모두 `1` 인 최단 거리 문제이므로, **도달한 순서가 곧 거리 순서**인 BFS 가 처음 도달하는 순간 답이 확정되어 가장 적합합니다.
- **BFS 대신 다익스트라를 쓰는 방법**
  - 간선 가중치가 모두 같기 때문에 우선순위 큐가 하는 일을 일반 큐가 이미 해줍니다. 다익스트라를 쓰면 `O(V log V)` 로 불필요한 비용만 늘어납니다.
- **좌표를 2배로 늘리지 않고 원래 격자(`51 × 51`)에서 푸는 방법**
  - 칸 수는 약 `2,600` 개로 4배 적지만, 한 칸 떨어져 평행한 두 테두리가 붙어 있는 것으로 판단되어 **존재하지 않는 길로 이동**하게 되므로 오답이 나옵니다.
  - 2배로 늘려도 칸 수가 `10,201` 개에 불과해 비용 증가는 무시할 수 있습니다. 정확성을 얻는 대신 치르는 비용이 매우 작으므로 확대하는 편이 맞습니다.
- **도형의 테두리를 기하학적으로 계산하는 방법**
  - 직사각형 변들을 잘라내고 이어 붙여 바깥 테두리의 선분들을 구해야 하는데, 겹침과 접촉 경우의 수가 많아 구현이 복잡하고 실수하기 쉽습니다.
  - 좌표 범위가 `50` 이하로 작기 때문에 격자에 직접 칠하는 방식이 훨씬 간단하고, 위에서 계산했듯 비용도 충분히 작습니다.

## 🧩 코드 구현

```java
import java.util.*;

class Solution {

    boolean[][] map = new boolean[101][101];
    boolean[][] inside = new boolean[101][101];

    int[] dx = {0, 1, 0, -1}; // → ↓ ← ↑
    int[] dy = {1, 0, -1, 0};

    public int solution(int[][] rectangle, int characterX, int characterY, int itemX, int itemY) {
        for (int[] r : rectangle) {
            int x1 = r[0] * 2;
            int y1 = r[1] * 2;
            int x2 = r[2] * 2;
            int y2 = r[3] * 2;

            for (int x = x1; x <= x2; ++x) {
                for (int y = y1; y <= y2; ++y) {
                    map[x][y] = true;
                }
            }

            for (int x = x1 + 1; x < x2; ++x) {
                for (int y = y1 + 1; y < y2; ++y) {
                    inside[x][y] = true;
                }
            }
        }

        int[][] dist = new int[101][101];
        for (int i = 0; i < 101; ++i) Arrays.fill(dist[i], -1);

        Queue<int[]> q = new ArrayDeque<>();

        int startX = characterX * 2;
        int startY = characterY * 2;

        q.offer(new int[]{startX, startY});
        dist[startX][startY] = 0;

        while (!q.isEmpty()) {
            int[] cur = q.poll();
            int x = cur[0];
            int y = cur[1];

            if (x == itemX * 2 && y == itemY * 2) {
                return dist[x][y] / 2;
            }

            for (int dir = 0; dir < 4; ++dir) {
                int nx = x + dx[dir];
                int ny = y + dy[dir];

                if (nx < 0 || ny < 0 || nx > 100 || ny > 100) continue;
                if (dist[nx][ny] != -1 || !map[nx][ny]) continue;
                if (inside[nx][ny]) continue;

                dist[nx][ny] = dist[x][y] + 1;
                q.offer(new int[]{nx, ny});
            }
        }

        return -1;
    }
}
```
