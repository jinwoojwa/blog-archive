## 🔗 문제 링크

<a class="bookmark-card" href="https://school.programmers.co.kr/learn/courses/30/lessons/42897" target="_blank" rel="noopener noreferrer">
  <div class="bookmark-card-title">도둑질</div>
  <div class="bookmark-card-desc">코딩테스트 연습 - 도둑질 문제입니다.</div>
  <div class="bookmark-card-url">
    <img class="bookmark-card-favicon" src="https://www.google.com/s2/favicons?domain=programmers.co.kr" alt="" />
    <span>school.programmers.co.kr</span>
  </div>
</a>

## 💡 문제 파악

- 집들이 원형으로 배치되어 있고, 각 집에는 `money[i]` 만큼의 돈이 있다.
- 바로 이웃한 두 집을 동시에 털면 경보가 울리므로, 인접한 두 집을 연달아 털 수 없다.
- 첫 번째 집과 마지막 집도 원형 구조상 이웃이므로, 두 집을 동시에 털 수 없다.
- 경보가 울리지 않는 선에서 훔칠 수 있는 돈의 최댓값을 구하는 문제이다.
- (원형이라는 조건 자체가, 0번 집과 n-1번 집이 서로 이웃해서 **둘 중 하나는 포기해야 한다**는 제약으로 이어진다.)

## 🗝️ 접근 방법

전형적인 DP 문제에 원형 조건이 추가된 형태이다.

- 일직선 형태라면, `i` 번째 집까지 털었을 때의 최댓값 `dp[i]` 는 다음 점화식을 따른다.
  - `i` 번째 집을 안 턴다면 → `dp[i - 1]`
  - `i` 번째 집을 턴다면 → `dp[i - 2] + money[i]` (바로 이전 집은 털 수 없으므로)
  - `dp[i] = max(dp[i - 1], dp[i - 2] + money[i])`
- 원형 배열에서는 첫 집과 마지막 집을 동시에 털 수 없으므로, 경우를 둘로 나눠 각각 일직선 DP를 돌리고 더 큰 값을 취한다.
  1. **0번 집을 포함하고 마지막 집은 제외**하는 구간 `[0, n - 2]`
  2. **0번 집은 제외하고 마지막 집을 포함**하는 구간 `[1, n - 1]`
- `dp` 배열 전체를 저장할 필요 없이, 직전 두 값(`prev1`, `prev2`)만 유지하는 식으로 구현했다.

```java
int skip = prev1;            // i번째 집을 안 터는 경우
int take = prev2 + money[i]; // i번째 집을 터는 경우

int current = Math.max(skip, take);

prev2 = prev1;
prev1 = current;
```

## 🧩 코드 구현

```java
class Solution {
    public int solution(int[] money) {
        int n = money.length;

        int case1 = rob(money, 0, n - 2);
        int case2 = rob(money, 1, n - 1);

        return Math.max(case1, case2);
    }

    private int rob(int[] money, int st, int end) {
        int prev1 = 0, prev2 = 0;

        for (int i = st; i <= end; i++) {
            int skip = prev1; // i 번째 안 터는 경우
            int take = prev2 + money[i]; // i 번째 터는 경우

            int current = Math.max(skip, take);

            prev2 = prev1;
            prev1 = current;
        }

        return prev1;
    }
}
```
