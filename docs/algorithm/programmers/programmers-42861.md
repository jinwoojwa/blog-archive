## 🔗 문제 링크

<a class="bookmark-card" href="https://school.programmers.co.kr/learn/courses/30/lessons/42861" target="_blank" rel="noopener noreferrer">
  <div class="bookmark-card-title">섬 연결하기</div>
  <div class="bookmark-card-desc">코딩테스트 연습 - 섬 연결하기 문제입니다.</div>
  <div class="bookmark-card-url">
    <img class="bookmark-card-favicon" src="https://www.google.com/s2/favicons?domain=programmers.co.kr" alt="" />
    <span>school.programmers.co.kr</span>
  </div>
</a>

## 💡 문제 파악

- `n`개의 섬과, 두 섬을 연결하는 다리 건설 비용 목록 `costs`가 주어진다.
- 모든 섬이 서로 통행 가능하도록(직접 연결이 아니어도 다리를 여러 번 건너 도달 가능하면 됨) 다리를 건설할 때, 필요한 최소 총 비용을 구해야 한다.
- 모든 정점을 최소 비용으로 연결하는 문제이므로 최소 신장 트리(MST)를 구하는 문제로 볼 수 있다.

## 🗝️ 접근 방법

- 프림(Prim) 알고리즘으로 최소 신장 트리를 구한다.
  - 인접 리스트(`graph`)에 각 섬에서 연결 가능한 다른 섬과 비용을 양방향으로 저장한다.
  - 우선순위 큐(`pq`)에 비용이 작은 간선부터 꺼낼 수 있도록 `Bridge`를 `cost` 기준 오름차순으로 정렬해서 사용한다.
- 임의의 시작점(0번 섬)에서 비용 0으로 출발해서, 가장 비용이 작은 간선부터 큐에서 꺼내며 트리를 확장해나간다.
  - 큐에서 꺼낸 섬이 이미 방문했다면 스킵한다(이미 더 싼 비용으로 연결된 상태이므로 무시).
  - 아직 방문하지 않은 섬이면 방문 처리하고, 그 간선의 비용을 정답에 더한다.
  - 방금 방문한 섬과 연결된, 아직 방문하지 않은 이웃 섬들을 모두 큐에 넣는다.
- 모든 섬이 방문될 때까지 반복하면, `answer`에는 트리를 완성하는 데 사용된 간선들의 비용 합, 즉 최소 신장 트리의 총 비용이 남는다.

## 🧩 코드 구현

```java
import java.util.*;

class Solution {

    static class Bridge {
        int island;
        int cost;

        Bridge(int island, int cost) {
            this.island = island;
            this.cost = cost;
        }
    }

    public int solution(int n, int[][] costs) {
        int answer = 0;

        List<Bridge>[] graph = new ArrayList[n];

        for (int i = 0; i < n; i++) {
            graph[i] = new ArrayList<>();
        }

        for (int[] cost : costs) {
            int a = cost[0];
            int b = cost[1];
            int price = cost[2];

            graph[a].add(new Bridge(b, price));
            graph[b].add(new Bridge(a, price));
        }

        boolean[] visited = new boolean[n];

        PriorityQueue<Bridge> pq =
                new PriorityQueue<>(Comparator.comparingInt(bridge -> bridge.cost));

        pq.offer(new Bridge(0, 0));

        while (!pq.isEmpty()) {
            Bridge current = pq.poll();

            if (visited[current.island]) continue;

            visited[current.island] = true;
            answer += current.cost;

            for (Bridge next : graph[current.island]) {
                if (!visited[next.island]) pq.offer(next);
            }
        }
        return answer;
    }
}
```
