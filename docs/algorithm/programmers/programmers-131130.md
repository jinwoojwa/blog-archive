## 🔗 문제 링크

<a class="bookmark-card" href="https://school.programmers.co.kr/learn/courses/30/lessons/131130" target="_blank" rel="noopener noreferrer">
  <div class="bookmark-card-title">혼자 놀기의 달인</div>
  <div class="bookmark-card-desc">코딩테스트 연습 - 혼자 놀기의 달인 문제입니다.</div>
  <div class="bookmark-card-url">
    <img class="bookmark-card-favicon" src="https://www.google.com/s2/favicons?domain=programmers.co.kr" alt="" />
    <span>school.programmers.co.kr</span>
  </div>
</a>

## 💡 문제 파악

- `1 ~ cards.length`의 숫자 카드가 무작위 순서로 상자마다 하나씩 들어 있다(`cards[i]`는 `i+1`번 상자 안의 카드 번호).
- 임의의 상자를 열어 나온 카드 번호에 해당하는 상자를 다시 여는 과정을, 처음 연 번호가 다시 나올 때까지 반복한다.
- 이렇게 열었던 상자들이 `1번 그룹`, 나머지 상자들이 `2번 그룹`이 되며, `1번 그룹 크기 × 2번 그룹 크기`의 최댓값을 구하는 문제이다.

## 🗝️ 접근 방법

- 어떤 상자에서 시작하든, 그 상자가 속한 사이클(그룹)의 크기는 항상 정해져 있다.
- 즉 전체 상자를 순회하면서, 아직 방문하지 않은 상자를 시작점으로 `cards[cur] - 1`을 따라가며 하나의 사이클을 구성하는 상자 개수를 센다.
- 모든 사이클의 크기를 구한 뒤 내림차순 정렬하고, 가장 큰 두 그룹의 크기를 곱하면 정답이 된다.
- 사이클이 하나뿐이라면(그룹을 두 개로 나눌 수 없다면) `0`을 반환한다.

## 🧩 코드 구현

```java
import java.util.*;

class Solution {
    public int solution(int[] cards) {
        List<Integer> group = new ArrayList<>();

        boolean[] visited = new boolean[cards.length];
        for (int i = 0; i < cards.length; ++i) {
            if (visited[i]) continue;

            int cnt = 0;
            int cur = i;

            while (!visited[cur]) {
                visited[cur] = true;
                cnt++;
                cur = cards[cur] - 1;
            }
            group.add(cnt);
        }
        group.sort(Collections.reverseOrder());

        if (group.size() == 1) return 0;
        return group.get(0) * group.get(1);
    }
}
```
