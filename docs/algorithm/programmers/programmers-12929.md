## 🔗 문제 링크

<a class="bookmark-card" href="https://school.programmers.co.kr/learn/courses/30/lessons/12929" target="_blank" rel="noopener noreferrer">
  <div class="bookmark-card-title">올바른 괄호의 갯수</div>
  <div class="bookmark-card-desc">코딩테스트 연습 - 올바른 괄호의 갯수 문제입니다.</div>
  <div class="bookmark-card-url">
    <img class="bookmark-card-favicon" src="https://www.google.com/s2/favicons?domain=programmers.co.kr" alt="" />
    <span>school.programmers.co.kr</span>
  </div>
</a>

## 💡 문제 파악

- 괄호쌍의 개수 `n`이 주어졌을 때, `n`개의 괄호쌍으로 만들 수 있는 올바른 괄호 문자열의 개수를 구하는 문제이다.
- 올바른 괄호란 `"(())"`, `"()()"`처럼 여는 괄호와 닫는 괄호가 알맞게 짝지어진 문자열을 의미한다.

## 🗝️ 접근 방법

- 만들 수 있는 모든 경우를 직접 구성해봐야 하므로 `DFS`로 여는 괄호와 닫는 괄호를 하나씩 추가해 나간다.
- 여는 괄호는 아직 `n`개를 다 쓰지 않았을 때만 추가할 수 있다.
- 닫는 괄호는 이미 사용한 여는 괄호 개수보다 적게 사용했을 때만 추가할 수 있다(괄호가 먼저 닫히면 안 되므로).
- 여는 괄호와 닫는 괄호를 각각 `n`개씩 모두 사용했을 때 하나의 올바른 괄호 문자열이 완성된 것으로 보고 개수를 센다.

## 🧩 코드 구현

```java
class Solution {
    private int res = 0;

    public int solution(int n) {
        dfs(n, 0, 0);
        return res;
    }

    private void dfs(int n, int open, int close) {
        if (open == n && close == n) {
            res++;
            return;
        }

        if (open < n) {
            dfs(n, open + 1, close);
        }

        if (close < open) {
            dfs(n, open, close + 1);
        }
    }
}
```
