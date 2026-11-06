## 🔗 문제 링크

<a class="bookmark-card" href="https://school.programmers.co.kr/learn/courses/30/lessons/42895" target="_blank" rel="noopener noreferrer">
  <div class="bookmark-card-title">N으로 표현</div>
  <div class="bookmark-card-desc">코딩테스트 연습 - N으로 표현 문제입니다.</div>
  <div class="bookmark-card-url">
    <img class="bookmark-card-favicon" src="https://www.google.com/s2/favicons?domain=programmers.co.kr" alt="" />
    <span>school.programmers.co.kr</span>
  </div>
</a>

## 💡 문제 파악

- 숫자 `N` 과 사칙연산(`+`, `-`, `*`, `/`)만 사용해서 숫자 `number` 를 만들 때, `N` 을 **사용하는 횟수의 최솟값**을 구합니다.
- `N` 을 이어 붙여 `55`, `555` 같은 수를 만들 수 있고, 괄호도 자유롭게 사용할 수 있습니다.
- 나눗셈에서 나머지는 무시합니다(정수 나눗셈).
- 최솟값이 8보다 크면 `-1` 을 반환합니다.

## 🗝️ 접근 방법

- `dp[i]` 를 **`N` 을 정확히 `i` 번 사용해서 만들 수 있는 수들의 집합**으로 정의합니다. 같은 수가 중복해서 만들어지므로 `Set` 으로 관리합니다.
- `dp[i]` 는 다음 두 가지로 채웁니다.
  - **이어 붙이기**: `N` 을 `i` 번 이어 붙인 수(`5`, `55`, `555`, ...)를 넣습니다.
  - **사칙연산**: `N` 을 `j` 번 사용한 수와 `i - j` 번 사용한 수를 하나씩 골라 사칙연산한 결과를 넣습니다.
    - 예를 들어 `dp[3]` 은 `dp[1]` 과 `dp[2]`, `dp[2]` 와 `dp[1]` 의 조합으로 만들어집니다.
    - `(5+5)+5` 처럼 괄호가 있는 식도 `dp[2]` 의 원소(`5+5`)와 `dp[1]` 의 원소(`5`)를 연산한 것이므로 자연스럽게 포함됩니다.
    - 뺄셈과 나눗셈은 순서에 따라 결과가 달라지므로 `j` 를 `1 ~ i` 까지 모두 돌려 양쪽 순서를 전부 시도합니다.
    - 나눗셈은 `0` 으로 나누는 경우를 제외합니다(`n2 != 0`).
    - `j == i` 일 때는 `dp[0]` 이 비어 있어 아무것도 만들어지지 않으므로 결과에 영향이 없습니다.
- `i` 를 1부터 8까지 늘려가며 채우다가, `number` 가 포함된 가장 작은 `i` 를 반환합니다. 8번까지 만들지 못하면 `-1` 입니다.

```java
private void calc(int i, int j) {
    for (int n1 : dp[j]) {
        for (int n2 : dp[i - j]) {
            dp[i].add(n1 + n2);
            dp[i].add(n1 - n2);
            dp[i].add(n1 * n2);
            if (n2 != 0) dp[i].add(n1 / n2);
        }
    }
}
```

## ⏱️ 복잡도 분석

### 시간 복잡도

- `dp[i]` 는 `dp[j]` 와 `dp[i - j]` 의 모든 원소 쌍에 사칙연산 4가지를 적용해 만듭니다. 따라서 `dp[i]` 를 채우는 비용은 `Σ |dp[j]| × |dp[i-j]| × 4` 입니다.
- 만들 수 있는 식의 개수 상한은 괄호 묶는 방법의 수(카탈란 수)와 연산자 조합의 곱인 `C(i-1) × 4^(i-1)` 입니다. `i = 8` 이면 `429 × 16,384 ≈ 700만` 입니다.
- 실제로는 같은 값이 `Set` 에서 하나로 합쳐지기 때문에 원소 수가 이보다 훨씬 적고, `N` 을 최대 8번까지만 사용하므로 수백만 번 이내의 연산으로 끝납니다.

### 공간 복잡도

- 모든 `dp[i]` 의 원소를 저장하므로 `O(Σ |dp[i]|)` 이고, 마찬가지로 중복 제거 덕분에 상한보다 훨씬 작습니다.

### 왜 이 방법으로 풀어야 하는가

- **`N` 을 사용한 횟수별로 식을 DFS 로 직접 만들어 보는 방법**
  - 같은 값을 만드는 식이 여러 번 중복해서 계산되어, 중복을 제거하는 `Set` 보다 훨씬 많은 연산을 합니다.
  - `dp[i]` 는 **더 작은 횟수의 결과를 재사용**하므로 `(5+5)+5` 처럼 이미 만든 부분 식을 다시 계산하지 않습니다.
- **앞에서부터 한 항씩 이어 붙이며 계산하는 방법 (`현재 값 ∘ N, NN, NNN, ...`)**
  - 구현은 단순하지만 `(N+N)*(N+N)` 처럼 **괄호로 묶인 두 덩어리를 서로 연산하는 식**을 만들 수 없어 정답이 빠질 수 있습니다.
  - `dp[j]` 와 `dp[i-j]` 의 모든 쌍을 조합하는 방식은 가능한 괄호 구조를 모두 포함합니다.
- **`N` 사용 횟수 제한(8번) 의 의미**
  - 횟수가 8을 넘으면 `-1` 이므로 `i` 를 8까지만 늘리면 되고, 이 제한 덕분에 위에서 계산한 정도의 집합 크기로 완전탐색이 가능합니다.

## 🧩 코드 구현

```java
import java.util.*;

class Solution {
    
    Set<Integer>[] dp = new HashSet[9];
    
    public int solution(int N, int number) {    
        for (int i = 0; i <= 8; ++i) {
            dp[i] = new HashSet<>();
        }
        
        // N을 이어붙여 만들 수 있는 수
        for (int i = 1; i <= 8; ++i) {
            int num = Integer.parseInt(String.valueOf(N).repeat(i));
            dp[i].add(num);
        }
        
        // 사칙연산을 통해 만들 수 있는 수
        for (int i = 2; i <= 8; ++i) {
            for (int j = 1; j <= i; ++j) {
                calc(i, j);
            }
        }
        
        for (int i = 1; i <= 8; ++i) {
            if (dp[i].contains(number)) return i;
        }
        
        return -1;
    }
    
    private void calc(int i, int j) {
        for (int n1 : dp[j]) {
            for (int n2 : dp[i - j]) {
                dp[i].add(n1 + n2);
                dp[i].add(n1 - n2);
                dp[i].add(n1 * n2);
                if (n2 != 0) dp[i].add(n1 / n2);
            }
        } 
    }
}
```
