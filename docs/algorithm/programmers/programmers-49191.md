## 🔗 문제 링크

<a class="bookmark-card" href="https://school.programmers.co.kr/learn/courses/30/lessons/49191" target="_blank" rel="noopener noreferrer">
  <div class="bookmark-card-title">순위</div>
  <div class="bookmark-card-desc">코딩테스트 연습 - 순위 문제입니다.</div>
  <div class="bookmark-card-url">
    <img class="bookmark-card-favicon" src="https://www.google.com/s2/favicons?domain=programmers.co.kr" alt="" />
    <span>school.programmers.co.kr</span>
  </div>
</a>

## 💡 문제 파악

- 1번부터 `n`번까지 번호가 매겨진 `n`명의 권투 선수가 있고, `results[i] = [A, B]` 는 A가 B를 이겼다는 뜻이다.
- 실력이 좋은 선수는 항상 실력이 낮은 선수를 이기므로, 승패 관계는 **추이적**이다. (A가 B를 이기고 B가 C를 이기면 A는 C도 이긴다.)
- 경기 결과로부터 **정확한 순위를 알 수 있는 선수의 수**를 구한다.

## 🗝️ 접근 방법

- 어떤 선수의 순위를 정확히 알 수 있으려면, **나머지 `n - 1`명 모두와의 승패 관계가 (직접 또는 간접적으로) 결정**되어 있어야 한다.
  - 내가 (간접적으로 포함해서) 이긴 선수 수 + 나를 (간접적으로 포함해서) 이긴 선수 수 = `n - 1` 이면 순위가 확정된다.
- 이를 위해 그래프를 두 방향으로 만든다.
  - `winGraph`: 이긴 선수 → 진 선수
  - `loseGraph`: 진 선수 → 이긴 선수
- 각 선수 `i` 에 대해 BFS 를 두 번 수행한다.
  - `winGraph` 에서 도달 가능한 선수 수: `i` 가 이긴(이길 수 있는) 선수 수
  - `loseGraph` 에서 도달 가능한 선수 수: `i` 를 이긴 선수 수
  - 두 값의 합이 `n - 1` 이면 `answer++` 한다.
- 시작 노드는 `visited` 처리만 하고 `count` 에는 포함하지 않으므로, 반환값은 자기 자신을 제외한 도달 가능 노드 수다.
- 선수는 최대 100명, 경기 결과는 최대 4,500개라 모든 선수에 대해 BFS 를 돌려도 충분하다.

```java
int win = bfs(i, winGraph);   // i가 이긴 선수 수
int lose = bfs(i, loseGraph); // i를 이긴 선수 수

if (win + lose == n - 1) answer++;
```

## 🧩 코드 구현

```java
import java.util.*;

class Solution {

    List<Integer>[] winGraph;
    List<Integer>[] loseGraph;

    public int solution(int n, int[][] results) {
        winGraph = new List[n + 1];
        loseGraph = new List[n + 1];

        for (int i = 1; i <= n; i++) {
            winGraph[i] = new ArrayList<>();
            loseGraph[i] = new ArrayList<>();
        }

        for (int[] result : results) {
            int win = result[0];
            int lose = result[1];

            winGraph[win].add(lose);  // 이긴 선수 -> 진 선수
            loseGraph[lose].add(win); // 진 선수 -> 이긴 선수
        }

        int answer = 0;

        for (int i = 1; i <= n; i++) {
            int win = bfs(i, winGraph);   // i가 이긴 선수 수
            int lose = bfs(i, loseGraph); // i를 이긴 선수 수

            if (win + lose == n - 1) answer++;
        }

        return answer;
    }

    private int bfs(int start, List<Integer>[] graph) {
        Queue<Integer> q = new ArrayDeque<>();
        boolean[] visited = new boolean[graph.length];

        q.offer(start);
        visited[start] = true;

        int count = 0;

        while (!q.isEmpty()) {
            int cur = q.poll();

            for (int next : graph[cur]) {
                if (visited[next]) continue;

                visited[next] = true;
                count++;
                q.offer(next);
            }
        }

        return count;
    }
}
```
