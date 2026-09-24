## 🔗 문제 링크

<a class="bookmark-card" href="https://school.programmers.co.kr/learn/courses/30/lessons/388352" target="_blank" rel="noopener noreferrer">
  <div class="bookmark-card-title">비밀 코드 해독</div>
  <div class="bookmark-card-desc">코딩테스트 연습 - 비밀 코드 해독 문제입니다.</div>
  <div class="bookmark-card-url">
    <img class="bookmark-card-favicon" src="https://www.google.com/s2/favicons?domain=programmers.co.kr" alt="" />
    <span>school.programmers.co.kr</span>
  </div>
</a>

## 💡 문제 파악

- `1 ~ n`의 정수 중 서로 다른 5개로 이루어진 비밀 코드가 존재한다.
- 비밀 코드를 맞추기 위한 여러 번의 입력(`q`)과, 각 입력이 비밀 코드와 몇 개나 일치하는지를 나타내는 응답(`ans`)이 주어진다.
- 주어진 모든 입력과 응답 조건을 만족하는, 비밀 코드가 될 수 있는 조합의 개수를 구하는 문제이다.

## 🗝️ 접근 방법

- `n`의 범위가 최대 30이므로, `1 ~ 30` 중 5개를 뽑는 모든 조합(`30C5`, 약 14만 가지)을 직접 만들어봐도 시간 내에 처리할 수 있다.
- `DFS`로 5개짜리 숫자 조합을 하나씩 만든 뒤, 주어진 모든 입력·응답 쌍을 만족하는지 검사한다.
- 모든 쌍을 만족하는 조합일 때만 정답 개수를 1 증가시킨다.

## 🧩 코드 구현

```java
import java.util.*;

class Solution {
    int n, answer;
    int[][] q;
    int[] ans;
    Set<Integer> selected = new HashSet<>();

    public int solution(int n, int[][] q, int[] ans) {
        this.n = n;
        this.q = q;
        this.ans = ans;

        dfs(1, 0);

        return answer;
    }

    private void dfs(int start, int cnt) {
        if (cnt == 5) {
            if (isPossible()) answer++;
            return;
        }

        for (int i = start; i <= n; ++i) {
            selected.add(i);
            dfs(i + 1, cnt + 1);
            selected.remove(i);
        }
    }

    private boolean isPossible() {
        for (int i = 0; i < q.length; ++i) {
            int count = 0;
            for (int num : q[i]) {
                if (selected.contains(num)) count++;
            }
            if (count != ans[i]) return false;
        }
        return true;
    }
}
```
