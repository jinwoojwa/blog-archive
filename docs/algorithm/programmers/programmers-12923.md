## 🔗 문제 링크

<a class="bookmark-card" href="https://school.programmers.co.kr/learn/courses/30/lessons/12923" target="_blank" rel="noopener noreferrer">
  <div class="bookmark-card-title">숫자 블록</div>
  <div class="bookmark-card-desc">코딩테스트 연습 - 숫자 블록 문제입니다.</div>
  <div class="bookmark-card-url">
    <img class="bookmark-card-favicon" src="https://www.google.com/s2/favicons?domain=programmers.co.kr" alt="" />
    <span>school.programmers.co.kr</span>
  </div>
</a>

## 💡 문제 파악

- 숫자 `n`이 적힌 블록은 도로 위 `n의 배수` 위치마다 설치된다.
- 도로 위 특정 구간 `[begin, end]`에 설치된 블록들의 번호를 구하는 문제이다.
- 각 위치 `p`에 설치되는 블록의 번호는 "`p`의 약수 중 1을 제외한 가장 큰 값(단, 10,000,000 이하)"으로 정리할 수 있다.

## 🗝️ 접근 방법

- 위치 `p`가 소수이면 그 위치를 자신의 배수로 갖는 블록이 없으므로 `0`이 된다(단, `p = 1`인 경우도 `0`).
- `p`가 소수가 아니라면, `p`의 약수 중 자기 자신을 제외한 가장 큰 값이 그 위치에 놓이는 블록의 번호가 된다.
  - 다만 블록 번호는 최대 10,000,000까지만 존재하므로, 약수가 이 값을 넘으면 사용할 수 없다.
- `√p`까지만 순회하며 약수를 찾고, `p / i`가 10,000,000 이하이면 바로 그 값을 답으로 사용한다. 그렇지 않다면 조건을 만족하는 더 작은 약수 중 최댓값을 답으로 사용한다.

## 🧩 코드 구현

```java
import java.util.*;

class Solution {

    static final int MAX_VALUE = 10_000_000;

    public int[] solution(long begin, long end) {
        List<Integer> numList = new ArrayList<>();
        for (int n = (int) begin; n <= (int) end; ++n) {
            int res = check(n);

            if (res == 0) numList.add(0);
            else if (res == -1) numList.add(1);
            else numList.add(res);
        }

        return numList.stream()
            .mapToInt(Integer::intValue)
            .toArray();
    }

    private int check(int num) {
        if (num == 1) return 0;

        int maxVal = -1;
        for (int i = 2; i * i <= num; ++i) {
            if (num % i == 0) {
                int d = num / i;

                if (d <= MAX_VALUE) return d;

                maxVal = Math.max(maxVal, i);
            }
        }
        return maxVal;
    }
}
```
