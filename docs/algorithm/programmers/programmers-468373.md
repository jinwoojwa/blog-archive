## 🔗 문제 링크

<a class="bookmark-card" href="https://school.programmers.co.kr/learn/courses/30/lessons/468373" target="_blank" rel="noopener noreferrer">
  <div class="bookmark-card-title">바이러스 파이프</div>
  <div class="bookmark-card-desc">코딩테스트 연습 - 바이러스 파이프 문제입니다.</div>
  <div class="bookmark-card-url">
    <img class="bookmark-card-favicon" src="https://www.google.com/s2/favicons?domain=programmers.co.kr" alt="" />
    <span>school.programmers.co.kr</span>
  </div>
</a>

## 💡 문제 파악

- `n`개의 배양체가 파이프로 연결되어 있고, 파이프는 최대 3가지 종류(A, B, C)이다.
- 한 번의 행동은 특정 종류의 파이프를 여는 것이며, 열린 파이프를 통해 감염이 전파된다.
- `k`번의 행동을 할 수 있을 때, 최대로 감염시킬 수 있는 배양체의 개수를 구하는 문제이다.

## 🗝️ 접근 방법

- 매 행동마다 A, B, C 중 어떤 파이프를 열어야 최댓값이 나오는지 미리 알 수 없으므로, 모든 경우의 수를 시도해봐야 한다.
- `k`번의 행동을 순서대로 정해나가는 과정은 `DFS`로 처리한다.
- 한 번의 행동으로 감염이 퍼지는 과정(현재 감염된 배양체들로부터 선택한 종류의 파이프를 타고 전파)은 `BFS`로 처리한다.
- 즉, `DFS`로 행동 순서를 정하고, 그때마다 `BFS`로 감염 상태를 갱신하면서 매 깊이마다 감염체 개수의 최댓값을 갱신한다.

## 🧩 코드 구현

```java
import java.util.*;

class Solution {

    int n;
    int k;
    int answer = 0;
    List<List<Node>> graph = new ArrayList<>();

    public record Node(int num, int type) {}

    public int solution(int n, int infection, int[][] edges, int k) {
        this.n = n;
        this.k = k;

        for (int i = 0; i <= n; ++i) {
            graph.add(new ArrayList<>());
        }

        for (int[] edge : edges) {
            int x = edge[0];
            int y = edge[1];
            int type = edge[2];

            graph.get(x).add(new Node(y, type));
            graph.get(y).add(new Node(x, type));
        }

        boolean[] infected = new boolean[n + 1];
        infected[infection] = true;

        dfs(0, infected);

        return answer;
    }

    private void dfs(int depth, boolean[] infected) {
        answer = Math.max(answer, count(infected));

        if (depth == k) return;

        for (int type = 1; type <= 3; type++) {
            boolean[] next = infected.clone();

            bfs(next, type);

            dfs(depth + 1, next);
        }
    }

    private void bfs(boolean[] infected, int type) {
        Queue<Integer> q = new ArrayDeque<>();

        for (int i = 1; i <= n; i++) {
            if (infected[i]) {
                q.offer(i);
            }
        }

        while (!q.isEmpty()) {
            int cur = q.poll();

            for (Node next : graph.get(cur)) {
                if (next.type() != type) continue;
                if (infected[next.num()]) continue;

                infected[next.num()] = true;
                q.offer(next.num());
            }
        }
    }

    private int count(boolean[] infected) {
        int cnt = 0;

        for (int i = 1; i <= n; i++) {
            if (infected[i]) cnt++;
        }
        return cnt;
    }
}
```
