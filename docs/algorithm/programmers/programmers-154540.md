## 🔗 문제 링크

<a class="bookmark-card" href="https://school.programmers.co.kr/learn/courses/30/lessons/154540" target="_blank" rel="noopener noreferrer">
  <div class="bookmark-card-title">무인도 여행</div>
  <div class="bookmark-card-desc">코딩테스트 연습 - 무인도 여행 문제입니다.</div>
  <div class="bookmark-card-url">
    <img class="bookmark-card-favicon" src="https://www.google.com/s2/favicons?domain=programmers.co.kr" alt="" />
    <span>school.programmers.co.kr</span>
  </div>
</a>

## 💡 문제 파악

- 지도(`maps`)가 주어지며, 각 칸은 바다(`X`)이거나 식량이 있는 무인도(`1~9`)이다.
- 상하좌우로 연결된 칸들은 하나의 무인도로 취급한다.
- 각 무인도에 있는 식량의 총합(머무를 수 있는 최대 일수)을 오름차순으로 반환해야 하며, 무인도가 하나도 없으면 `[-1]`을 반환한다.

## 🗝️ 접근 방법

- 맵의 모든 칸을 순회하면서, 아직 방문하지 않은 육지 칸을 발견하면 그 지점에서 `BFS`를 시작한다.
- `BFS`로 상하좌우로 이어진 칸들을 모두 방문 처리하면서 식량 값을 더해, 하나의 무인도가 가진 식량 총합을 구한다.
- 구해진 합들을 리스트에 모은 뒤 오름차순으로 정렬해서 반환한다. 무인도가 하나도 없으면 `[-1]`을 반환한다.

## 🧩 코드 구현

```java
import java.util.*;

class Solution {

    static int[] dx = {0, 1, 0, -1};
    static int[] dy = {1, 0, -1, 0};

    public int[] solution(String[] maps) {
        int n = maps.length;
        int m = maps[0].length();

        List<Integer> resultList = new ArrayList<>();
        boolean[][] visited = new boolean[n][m];

        for (int i = 0; i < n; ++i) {
            for (int j = 0; j < m; ++j) {
                if (maps[i].charAt(j) == 'X') continue;
                if (visited[i][j]) continue;

                Queue<int[]> q = new ArrayDeque<>();
                visited[i][j] = true;

                q.offer(new int[]{i, j});
                int foodSum = maps[i].charAt(j) - '0';

                while (!q.isEmpty()) {
                    int[] cur = q.poll();

                    for (int dir = 0; dir < 4; ++dir) {
                        int nx = cur[0] + dx[dir];
                        int ny = cur[1] + dy[dir];

                        if (nx < 0 || ny < 0 || nx >= n || ny >= m) continue;
                        if (visited[nx][ny] || maps[nx].charAt(ny) == 'X') continue;

                        q.offer(new int[]{nx, ny});
                        visited[nx][ny] = true;
                        foodSum += maps[nx].charAt(ny) - '0';
                    }
                }
                resultList.add(foodSum);
            }
        }
        if (resultList.isEmpty()) return new int[]{-1};

        Collections.sort(resultList);
        return resultList.stream()
            .mapToInt(Integer::intValue)
            .toArray();
    }
}
```
