## 🔗 문제 링크

<a class="bookmark-card" href="https://school.programmers.co.kr/learn/courses/30/lessons/92343" target="_blank" rel="noopener noreferrer">
  <div class="bookmark-card-title">양과 늑대</div>
  <div class="bookmark-card-desc">코딩테스트 연습 - 양과 늑대 문제입니다.</div>
  <div class="bookmark-card-url">
    <img class="bookmark-card-favicon" src="https://www.google.com/s2/favicons?domain=programmers.co.kr" alt="" />
    <span>school.programmers.co.kr</span>
  </div>
</a>

## 💡 문제 파악

- 이진 트리의 각 노드에는 양(`0`) 또는 늑대(`1`)가 있고, 루트(0번 노드)에서 시작합니다. 루트는 항상 양입니다.
- 부모 노드를 방문한 상태에서만 자식 노드로 이동할 수 있고, 이동한 노드의 양/늑대가 일행에 추가됩니다.
- 늑대의 수가 양의 수 **이상**이 되면 양이 모두 잡아먹히므로 그 상태는 허용되지 않습니다.
- 늑대에게 잡아먹히지 않고 모을 수 있는 **양의 최대 수**를 구합니다.
- 노드는 최대 17개라 완전 탐색이 가능합니다.

## 🗝️ 접근 방법

- 단순히 자식 방향으로만 내려가는 DFS 는 "지금은 늑대가 있어서 못 가지만, 다른 가지에서 양을 먼저 모으고 오면 갈 수 있는 노드"를 놓칩니다.
- 그래서 **현재까지 방문한 노드 집합에서 다음에 방문할 수 있는 모든 노드**를 후보로 삼는 방식으로 탐색합니다.
  - `visited[]` 로 방문한 노드를 기록합니다.
  - 모든 간선 `[parent, child]` 를 순회하면서 `visited[parent] && !visited[child]` 인 간선이 곧 다음에 갈 수 있는 후보입니다.
    - 노드로 이동하려면 **부모를 이미 방문**했어야 하고(`visited[parent]`), 아직 **방문하지 않은 노드**여야 하기(`!visited[child]`) 때문입니다.
    - 현재 노드의 자식만 보는 게 아니라 방문한 노드 전체에 붙어 있는 노드를 모두 후보로 보므로, 다른 가지에서 양을 먼저 모은 뒤 돌아오는 경우도 탐색됩니다.
- `dfs(sheep, wolf)` 는 다음과 같이 동작합니다.
  - `wolf >= sheep` 이면 양이 잡아먹히므로 더 진행하지 않고 반환합니다.
  - 그렇지 않으면 `answer` 를 `sheep` 으로 갱신합니다.
  - 갈 수 있는 자식 노드를 방문 처리하고, 양이면 `sheep + 1`, 늑대면 `wolf + 1` 로 재귀 호출한 뒤 **백트래킹**으로 방문을 되돌립니다.
- 루트는 항상 양이므로 `visited[0] = true` 로 두고 `dfs(1, 0)` 에서 시작합니다.

```java
for (int[] edge : edges) {
    int parent = edge[0];
    int child = edge[1];

    if (visited[parent] && !visited[child]) {
        visited[child] = true;

        if (info[child] == 0) dfs(sheep + 1, wolf); // 양
        else dfs(sheep, wolf + 1); // 늑대

        visited[child] = false;
    }
}
```

## ⏱️ 복잡도 분석

### 시간 복잡도

- 노드 수를 `n` (최대 17) 이라고 하면, 한 번의 `dfs` 호출은 모든 간선을 한 번씩 확인하므로 `O(n)` 입니다.
- 호출 횟수는 **가능한 방문 순서의 수**에 비례합니다. 이론적인 상한은 `n!` 이지만, 이진 트리라 한 번에 갈 수 있는 후보가 제한되고 `wolf >= sheep` 이 되는 가지는 바로 잘려서 실제 탐색량은 훨씬 적습니다. 그래서 `n ≤ 17` 에서 통과합니다.
- 같은 방문 집합이 다른 순서로 여러 번 탐색될 수 있으므로, 더 안전하게 풀려면 방문 집합을 비트마스크로 저장해 메모이제이션하면 상태가 최대 `2^n` 개로 줄어 `O(2^n × n)` (약 200만 번) 입니다.

### 공간 복잡도

- `visited` 배열과 재귀 깊이가 각각 `O(n)` 입니다.

### 왜 이 방법으로 풀어야 하는가

- **루트에서 자식 방향으로만 내려가는 단순 DFS**
  - 지금은 늑대 때문에 못 가는 노드가, 다른 가지에서 양을 먼저 모으면 갈 수 있게 되는 경우를 놓칩니다.
  - 방문한 노드 전체에 붙은 후보를 모두 시도해야 하므로 방문 집합을 상태로 삼는 탐색이 필요합니다.
- **항상 양이 있는 노드를 먼저 가는 그리디**
  - 양을 먼저 모으다 보면 늑대 뒤에 있는 양들을 영영 얻지 못할 수 있습니다. 때로는 늑대를 데려가야 더 많은 양을 얻으므로 국소적인 선택으로는 최적해를 보장할 수 없습니다.
- **노드 수가 작다는 제약 활용**
  - `n ≤ 17` 이라 완전탐색과 백트래킹이 가능합니다. 노드가 수천 개라면 이 방법은 쓸 수 없고 다른 접근이 필요합니다.

## 🧩 코드 구현

```java
class Solution {

    int answer;
    boolean[] visited;
    int[] info;
    int[][] edges;

    public int solution(int[] info, int[][] edges) {
        this.info = info;
        this.edges = edges;
        visited = new boolean[info.length];

        visited[0] = true;

        dfs(1, 0);

        return answer;
    }

    private void dfs(int sheep, int wolf) {
        if (wolf >= sheep) return;

        answer = Math.max(answer, sheep);

        for (int[] edge : edges) {
            int parent = edge[0];
            int child = edge[1];

            if (visited[parent] && !visited[child]) {
                visited[child] = true;

                if (info[child] == 0) dfs(sheep + 1, wolf); // 양
                else dfs(sheep, wolf + 1); // 늑대

                visited[child] = false;
            }
        }
    }
}
```
