## 🔗 문제 링크

<a class="bookmark-card" href="https://school.programmers.co.kr/learn/courses/30/lessons/12946" target="_blank" rel="noopener noreferrer">
  <div class="bookmark-card-title">하노이의 탑</div>
  <div class="bookmark-card-desc">코딩테스트 연습 - 하노이의 탑 문제입니다.</div>
  <div class="bookmark-card-url">
    <img class="bookmark-card-favicon" src="https://www.google.com/s2/favicons?domain=programmers.co.kr" alt="" />
    <span>school.programmers.co.kr</span>
  </div>
</a>

## 💡 문제 파악

- 원판이 `n` 개 쌓여 있는 하노이의 탑을 1번 기둥에서 3번 기둥으로 옮기려고 한다.
- 한 번에 한 개의 원판만 옮길 수 있고, 큰 원판이 작은 원판 위에 올라갈 수 없다.
- 옮기는 순서대로 `[from, to]` 쌍을 담은 2차원 배열을 반환해야 한다.

## 🗝️ 접근 방법

하노이의 탑은 대표적인 분할 정복(재귀) 문제로, `n` 개의 원판을 옮기는 문제를 더 작은 `n - 1` 개의 원판을 옮기는 문제로 쪼갤 수 있다.

- `n` 개의 원판을 `from` 기둥에서 `to` 기둥으로 옮기려면
  1. 맨 아래 원판을 제외한 `n - 1` 개의 원판을 `from` 에서 `via`(보조 기둥) 로 옮긴다.
  2. 맨 아래 남은 원판 1개를 `from` 에서 `to` 로 옮긴다. (이 순간을 정답 리스트에 기록)
  3. `via` 에 옮겨둔 `n - 1` 개의 원판을 `to` 로 옮긴다.
- 위 1번과 3번 과정은 원판 개수만 다를 뿐 동일한 형태이므로, `from`, `via`, `to` 를 바꿔가며 재귀 호출한다.
- 종료 조건은 옮길 원판이 없을 때(`n == 0`)이다.

```java
playHanoi(n - 1, from, to, via); // 1) n-1개를 보조 기둥으로
answer.add(new int[]{from, to}); // 2) 마지막 원판 이동 기록
playHanoi(n - 1, via, from, to); // 3) n-1개를 목표 기둥으로
```

## 🧩 코드 구현

```java
import java.util.*;

class Solution {
    List<int[]> answer = new ArrayList<>();

    public int[][] solution(int n) {
        playHanoi(n, 1, 2, 3);

        return answer.toArray(new int[0][]);
    }

		// n개의 원판을 from에서 to로 옮기는 함수
    private void playHanoi(int n, int from, int via, int to) {
        if (n == 0) return;

        playHanoi(n - 1, from, to, via);

        answer.add(new int[]{from, to});

        playHanoi(n - 1, via, from, to);
    }
}
```
