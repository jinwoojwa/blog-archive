## 🔗 문제 링크

<a class="bookmark-card" href="https://school.programmers.co.kr/learn/courses/30/lessons/389480" target="_blank" rel="noopener noreferrer">
  <div class="bookmark-card-title">완전범죄</div>
  <div class="bookmark-card-desc">코딩테스트 연습 - 완전범죄 문제입니다.</div>
  <div class="bookmark-card-url">
    <img class="bookmark-card-favicon" src="https://www.google.com/s2/favicons?domain=programmers.co.kr" alt="" />
    <span>school.programmers.co.kr</span>
  </div>
</a>

## 💡 문제 파악

- A와 B 두 명이 `info.length` 개의 물건을 하나씩, 순서대로 **둘 중 한 명이** 훔친다. (물건마다 훔치는 사람을 정해야 함)
- `info[i] = [a, b]` 는 `i` 번째 물건을 A가 훔치면 A의 흔적이 `a` 만큼, B가 훔치면 B의 흔적이 `b` 만큼 늘어난다는 뜻이다.
- A의 흔적은 `n` 미만, B의 흔적은 `m` 미만으로 유지해야 잡히지 않는다.
- 모든 물건을 훔치면서 둘 다 잡히지 않을 수 있는 경우 중, **A의 최종 흔적이 최소가 되는 값**을 구한다. 그런 분배가 불가능하면 `-1`을 반환한다.

## 🗝️ 접근 방법

- `i` 번째 물건까지 처리했을 때 "A의 흔적이 정확히 `a`, B의 흔적이 정확히 `b`가 되는 분배가 가능한가?"를 상태로 갖는 `dp[len + 1][n][m]` 불리언 배열을 사용한다.
- `dp[0][0][0] = true` 에서 시작해서, 물건을 하나씩 처리할 때마다 두 가지 선택을 모두 전이시킨다.
  - A가 훔치는 경우: `traceA + a < n` 이면 `dp[i + 1][traceA + a][traceB] = true`
  - B가 훔치는 경우: `traceB + b < m` 이면 `dp[i + 1][traceA][traceB + b] = true`
  - 흔적이 임계값(`n`, `m`) 이상이 되는 전이는 잡히는 경우이므로 아예 만들지 않는다.
- 모든 물건(`len`)을 처리한 뒤, `dp[len][traceA][traceB]` 가 `true` 인 상태들 중 `traceA` 가 가장 작은 값을 찾는다.
  - `traceA` 를 `0`부터 순서대로 탐색하므로, 처음으로 `true` 를 만나는 값이 곧 최솟값이 되어 바로 반환할 수 있다.
- 끝까지 `true` 인 상태가 하나도 없다면 둘 다 안 잡히게 모든 물건을 분배할 방법이 없다는 뜻이므로 `-1`을 반환한다.

```java
// A가 훔침
if (traceA + a < n) {
    dp[i + 1][traceA + a][traceB] = true;
}

// B가 훔침
if (traceB + b < m) {
    dp[i + 1][traceA][traceB + b] = true;
}
```

## 🧩 코드 구현

```java
class Solution {
    public int solution(int[][] info, int n, int m) {
        int len = info.length;

        // i개의 물건을 처리했을 때, A의 흔적이 a, B의 흔적이 b가 되는 경우가 존재함?
        boolean[][][] dp = new boolean[len + 1][n][m]; // i, a, b

        dp[0][0][0] = true;

        for (int i = 0; i < len; i++) {
            int a = info[i][0];
            int b = info[i][1];

            for (int traceA = 0; traceA < n; ++traceA) {
                for (int traceB = 0; traceB < m; ++traceB) {
                    if (!dp[i][traceA][traceB]) continue;

                    // A가 훔침
                    if (traceA + a < n) {
                        dp[i + 1][traceA + a][traceB] = true;
                    }

                    // B가 훔침
                    if (traceB + b < m) {
                        dp[i + 1][traceA][traceB + b] = true;
                    }
                }
            }
        }

        for (int traceA = 0; traceA < n; ++traceA) {
            for (int traceB = 0; traceB < m; ++traceB) {
                if (dp[len][traceA][traceB]) {
                    return traceA;
                }
            }
        }
        return -1;
    }
}
```
