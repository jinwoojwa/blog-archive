## 🔗 문제 링크

<a class="bookmark-card" href="https://school.programmers.co.kr/learn/courses/30/lessons/72413" target="_blank" rel="noopener noreferrer">
  <div class="bookmark-card-title">합승 택시 요금</div>
  <div class="bookmark-card-desc">코딩테스트 연습 - 합승 택시 요금 문제입니다.</div>
  <div class="bookmark-card-url">
    <img class="bookmark-card-favicon" src="https://www.google.com/s2/favicons?domain=programmers.co.kr" alt="" />
    <span>school.programmers.co.kr</span>
  </div>
</a>

## 💡 문제 파악

- `n` 개의 지점과 각 지점을 잇는 도로(요금) 정보 `fares` 가 주어진다.
- 출발지 `s` 에서 승객 두 명이 각각 목적지 `a`, `b` 로 가야 한다.
- 두 사람은 `s` 에서 어느 지점 `k` 까지는 합승해서 이동하고, 그 지점에서 내려 각자 `a`, `b` 로 따로 이동할 수 있다. (합승하지 않고 처음부터 따로 가는 경우도 가능)
- 총 요금이 최소가 되는 경로를 찾아 그 요금을 구해야 한다.

## 🗝️ 접근 방법

- 두 사람이 갈라지는 지점 `k` 를 1부터 `n` 까지 모두 시도해보고, `s → k`(합승) + `k → a` + `k → b` 요금의 합이 최소가 되는 `k` 를 찾는 문제로 바꿔 생각한다. (`k == s` 인 경우가 처음부터 따로 가는 경우에 해당한다.)
- 이를 위해서는 특정 지점에서 다른 모든 지점까지의 최단 거리가 필요하므로, 다익스트라를 세 번 수행한다.
  - `s` 를 시작점으로 한 `distS[]`
  - `a` 를 시작점으로 한 `distA[]`
  - `b` 를 시작점으로 한 `distB[]`
- 모든 지점 `k` 에 대해 `distS[k] + distA[k] + distB[k]` 를 계산하고, 그중 최솟값이 정답이다.
  - 그래프가 양방향이므로 `distA[k]` 는 `a → k` 와 `k → a` 요금이 같고, `distB[k]` 도 마찬가지이기 때문에 이 합이 성립한다.

```java
int[] distS = dijkstra(s, n);
int[] distA = dijkstra(a, n);
int[] distB = dijkstra(b, n);

int answer = Integer.MAX_VALUE;

for (int k = 1; k <= n; k++) {
    int fare = distS[k] + distA[k] + distB[k];
    answer = Math.min(answer, fare);
}
```

## 🧩 코드 구현

```java
import java.util.*;

class Solution {

    class Edge {
        int next;
        int cost;

        Edge(int next, int cost) {
            this.next = next;
            this.cost = cost;
        }
    }

    List<Edge>[] graph;

    public int solution(int n, int s, int a, int b, int[][] fares) {

        graph = new ArrayList[n + 1];

        for (int i = 1; i <= n; i++) {
            graph[i] = new ArrayList<>();
        }

        for (int[] fare : fares) {
            int c = fare[0];
            int d = fare[1];
            int f = fare[2];

            graph[c].add(new Edge(d, f));
            graph[d].add(new Edge(c, f));
        }

        int[] distS = dijkstra(s, n);
        int[] distA = dijkstra(a, n);
        int[] distB = dijkstra(b, n);

        int answer = Integer.MAX_VALUE;

        for (int k = 1; k <= n; k++) {
            int fare = distS[k] + distA[k] + distB[k];
            answer = Math.min(answer, fare);
        }

        return answer;
    }

    private int[] dijkstra(int start, int n) {

        int[] dist = new int[n + 1];
        Arrays.fill(dist, Integer.MAX_VALUE);

        dist[start] = 0;

        PriorityQueue<int[]> pq = new PriorityQueue<>(Comparator.comparing(a -> a[1]));
        pq.offer(new int[]{start, 0});

        while (!pq.isEmpty()) {
            int[] cur = pq.poll();

            if (cur[1] > dist[cur[0]]) continue;

            for (Edge edge : graph[cur[0]]) {
                int next = edge.next;
                int nextCost = cur[1] + edge.cost;

                if (nextCost < dist[next]) {
                    dist[next] = nextCost;
                    pq.offer(new int[]{next, nextCost});
                }
            }
        }
        return dist;
    }
}
```
