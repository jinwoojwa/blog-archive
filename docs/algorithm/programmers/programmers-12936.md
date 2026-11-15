## 🔗 문제 링크

<a class="bookmark-card" href="https://school.programmers.co.kr/learn/courses/30/lessons/12936" target="_blank" rel="noopener noreferrer">
  <div class="bookmark-card-title">줄 서는 방법</div>
  <div class="bookmark-card-desc">코딩테스트 연습 - 줄 서는 방법 문제입니다.</div>
  <div class="bookmark-card-url">
    <img class="bookmark-card-favicon" src="https://www.google.com/s2/favicons?domain=programmers.co.kr" alt="" />
    <span>school.programmers.co.kr</span>
  </div>
</a>

## 💡 문제 파악

- `1` 번부터 `n` 번까지 `n` 명이 줄을 서는 모든 방법을 **사전 순**으로 나열했을 때, `k` 번째 방법을 구합니다.
- 예를 들어 `n = 3` 이면 `[1,2,3]`, `[1,3,2]`, `[2,1,3]`, `[2,3,1]`, `[3,1,2]`, `[3,2,1]` 순서이고, `k = 5` 이면 `[3,1,2]` 입니다.
- `n` 은 20 이하, `k` 는 `n!` 이하입니다.

## 🗝️ 접근 방법

- `n` 명을 줄 세우는 방법은 `n!` 가지입니다. 이 중 **첫 번째 자리가 같은 방법**은 나머지 `n - 1` 명을 줄 세우는 `(n - 1)!` 가지씩 묶여 있습니다.
  - 예를 들어 `n = 3` 이면 첫 번째 자리가 `1` 인 방법이 2가지, `2` 인 방법이 2가지, `3` 인 방법이 2가지입니다.
- 그러므로 `k` 번째 방법의 첫 번째 자리는 `k` 가 `(n - 1)!` 묶음 중 **몇 번째 묶음**에 속하는지로 정해집니다. 이를 이용해 앞자리부터 하나씩 확정합니다.
  - 먼저 `k` 를 **0부터 세는 인덱스**로 바꾸기 위해 `k--` 를 합니다.
  - `i` 번째 자리에서는 `remain = (n - 1 - i)!` 로 묶음 크기를 구하고, `index = k / remain` 번째로 작은 남은 숫자가 이 자리의 값입니다.
  - 사용한 숫자는 목록에서 제거하고, `k %= remain` 으로 **묶음 안에서의 순서**로 `k` 를 줄입니다.
- 이 과정을 `n` 번 반복하면 모든 자리가 확정됩니다.
- 팩토리얼 값은 미리 `factorials[]` 에 계산해 둡니다. `20! ≈ 2.4 × 10^18` 로 `long` 범위(약 `9.2 × 10^18`) 안에 들어오므로 `n ≤ 20` 에서는 오버플로가 나지 않습니다.

```java
k--;

for (int i = 0; i < n; ++i) {
    long remain = factorials[n - 1 - i];

    int index = (int) (k / remain);
    answer[i] = numbers.get(index);
    numbers.remove(index);

    k %= remain;
}
```

- `n = 3`, `k = 5` 를 따라가면 다음과 같습니다. `k` 는 `4` 가 되고 남은 숫자는 `[1, 2, 3]` 입니다.

| 자리 | `remain` | `index = k / remain` | 선택한 숫자 | 남은 숫자 | `k` |
|---|---|---|---|---|---|
| 0 | `2! = 2` | `4 / 2 = 2` | `3` | `[1, 2]` | `4 % 2 = 0` |
| 1 | `1! = 1` | `0 / 1 = 0` | `1` | `[2]` | `0` |
| 2 | `0! = 1` | `0 / 1 = 0` | `2` | `[]` | `0` |

  - 결과는 `[3, 1, 2]` 입니다.

## ⏱️ 복잡도 분석

### 시간 복잡도

- 팩토리얼을 계산하는 데 `O(n)` 입니다.
- 자리마다 `ArrayList` 에서 숫자를 꺼내고(`get`, `O(1)`) 제거(`remove`, `O(n)`)하므로 자리 하나에 `O(n)` 이고, `n` 자리 전체는 **`O(n²)`** 입니다.
- `n` 이 최대 20 이므로 약 400번 연산으로 끝납니다.

### 공간 복잡도

- 팩토리얼 배열, 숫자 목록, 결과 배열이 모두 크기 `n` 이므로 `O(n)` 입니다.

### 왜 이 방법으로 풀어야 하는가

- **모든 순열을 만들어 `k` 번째를 고르는 방법**
  - 순열이 `n!` 개인데 `20! ≈ 2.4 × 10^18` 이라 만드는 것 자체가 불가능합니다.
- **`k` 번째까지만 DFS 로 순열을 만들다가 멈추는 방법**
  - 사전 순으로 만들면 `k` 번째에서 멈출 수 있지만, `k` 가 `n!` 에 가까우면 거의 모든 순열을 만들어야 합니다. 최악의 경우 `O(k)` (최대 `10^18` 이상) 입니다.
- **다음 순열(`next_permutation`)을 `k - 1` 번 호출하는 방법**
  - 마찬가지로 호출 횟수가 `k` 에 비례하므로 `k` 가 큰 입력에서 불가능합니다.
- **자릿수별로 묶음 크기를 이용해 바로 계산하는 방법 (팩토리얼 진법)**
  - 앞의 방법들은 `k` 번째 이전의 순열을 하나씩 거치지만, 이 방법은 `k` 가 속한 묶음을 나눗셈 한 번으로 건너뛰므로 **`n` 번의 계산**으로 끝납니다. `k` 의 크기와 무관합니다.
- **`ArrayList` 대신 세그먼트 트리나 BIT 로 `k` 번째 남은 숫자를 찾는 방법**
  - `O(n log n)` 으로 더 빠르게 만들 수 있지만, `n ≤ 20` 에서는 `O(n²)` 도 충분히 작아서 구현이 단순한 `ArrayList` 를 선택했습니다.

## 🧩 코드 구현

```java
import java.util.*;

class Solution {
    public int[] solution(int n, long k) {
        // n명의 사람을 줄 세우는 방법의 가짓수 = n!
        // k가 주어졌을 때, 각 자릿수를 구할 수 있음
        
        long[] factorials = makeFactorial(n);
        
        List<Integer> numbers = new ArrayList<>();

        for (int i = 1; i <= n; ++i) {
            numbers.add(i);
        }

        int[] answer = new int[n];
        k--;

        for (int i = 0; i < n; ++i) {
            long remain = factorials[n - 1 - i];

            int index = (int) (k / remain);
            answer[i] = numbers.get(index);
            numbers.remove(index);

            k %= remain;
        }

        return answer;        
    }
    
    private long[] makeFactorial(int n) {
        long[] factorials = new long[n + 1];
        
        factorials[0] = 1;
        
        for (int i = 1; i <= n; ++i) {
            factorials[i] = factorials[i - 1] * i;
        }
        return factorials;
    }
}
```
