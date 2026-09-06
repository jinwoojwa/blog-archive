## 🔗 문제 링크

<a class="bookmark-card" href="https://school.programmers.co.kr/learn/courses/30/lessons/49994" target="_blank" rel="noopener noreferrer">
  <div class="bookmark-card-title">방문 길이</div>
  <div class="bookmark-card-desc">코딩테스트 연습 - 방문 길이 문제입니다.</div>
  <div class="bookmark-card-url">
    <img class="bookmark-card-favicon" src="https://www.google.com/s2/favicons?domain=programmers.co.kr" alt="" />
    <span>school.programmers.co.kr</span>
  </div>
</a>

## 💡 문제 파악

- `(5, 5)` 좌표에서 시작해 `dirs` 문자열(`U`, `D`, `L`, `R`)을 따라 한 칸씩 이동한다.
- 좌표 범위는 `(0, 0)` ~ `(10, 10)` 이며, 범위를 벗어나는 이동은 무시하고 제자리에 머문다.
- 이동하면서 지나간 **겹치지 않는 선(길)**의 총 개수를 구해야 한다.
  - 같은 구간을 정방향/역방향으로 다시 지나가도 한 번만 세야 한다.

## 🗝️ 접근 방법

- 좌표 자체가 아니라 **두 점을 잇는 간선(선분)** 을 방문했는지 기록해야 중복 이동을 걸러낼 수 있다.
- 간선을 `visited[x][y][dir]` 형태의 3차원 배열로 표현한다.
- 한 번 이동할 때마다
  1. 이동 방향(`moveDir`)과 그 반대 방향(`reverseDir`)을 구하고, 이동 후 좌표 `(nx, ny)` 를 계산한다.
  2. `(nx, ny)` 가 범위를 벗어나면 무시(`continue`)한다.
  3. `visited[nx][ny][moveDir]` 가 아직 방문되지 않았다면 정답을 1 증가시킨다.
  4. `visited[nx][ny][moveDir]` 와 `visited[x][y][reverseDir]` 를 함께 방문 처리한다.
- 핵심은 같은 간선을 나중에 반대 방향으로 지나갈 때, 이번에 갱신한 값 중 하나가 그 이동의 검사 조건과 정확히 맞아떨어지도록 짝을 지어 표시해두는 것이다.
  - 예를 들어 `(5,5) → (6,5)` 로 이동(R)한 뒤 `visited[5][5][L]` 을 참으로 만들어두면, 나중에 `(6,5) → (5,5)` 로 반대 이동(L)할 때의 검사 조건인 `visited[5][5][L]` 이 이미 참이므로 같은 간선이 중복 계산되지 않는다.
- 이렇게 좌표마다 4방향 간선 방문 여부만 확인하면 되므로, 3차원 방문 배열 `boolean[11][11][4]` 로 문제를 해결할 수 있다.

```java
int reverseDir = (moveDir + 2) % 4; // 반대 방향 계산

if (!visited[nx][ny][moveDir]) answer++;

visited[nx][ny][moveDir] = true;
visited[x][y][reverseDir] = true; // 반대 방향 이동 시 검사에 걸리도록 미리 표시
```

## 🧩 코드 구현

```java
class Solution {
    
    int[] dx = {0, 1, 0, -1}; // R, D, L, U
    int[] dy = {1, 0, -1, 0};
    
    public int solution(String dirs) {
        int answer = 0;
        
        boolean[][][] visited = new boolean[11][11][4]; // 시작 좌표: (5, 5)
        int x = 5, y = 5;
        
        for (int i = 0; i < dirs.length(); ++i) {
            int moveDir = getDir(dirs.charAt(i));
            int reverseDir = (moveDir + 2) % 4;
            
            int nx = x + dx[moveDir];
            int ny = y + dy[moveDir];
            
            if (nx < 0 || ny < 0 || nx > 10 || ny > 10) continue;
            
            // 아직 방문 X
            if (!visited[nx][ny][moveDir]) answer++;
            
            visited[nx][ny][moveDir] = true;
            visited[x][y][reverseDir] = true;
            x = nx; y = ny;
        }
        return answer;
    }
    
    private int getDir(char c) {
        return switch (c) {
                case 'R' -> 0;
                case 'D' -> 1;
                case 'L' -> 2;
                case 'U' -> 3;
                default -> -1;
        };
    }
}
```
