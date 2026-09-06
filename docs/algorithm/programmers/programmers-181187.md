---
title: 두 원 사이의 정수 쌍 - 프로그래머스
---

## 🔗 문제 링크

<a href="https://school.programmers.co.kr/learn/courses/30/lessons/181187" target="_blank" rel="noopener noreferrer" class="bookmark-link">
  <div class="bookmark-content">
    <div class="bookmark-title">프로그래머스 - 두 원 사이의 정수 쌍</div>
    <div class="bookmark-url">school.programmers.co.kr</div>
  </div>
</a>

## 💡 문제 파악

- 반지름이 `r1`, `r2` 인 원 2개가 주어짐
- 두 원 사이 공간에 포함된 `x`, `y` 좌표가 정수인 점의 개수를 구하는 문제

## 🗝️ 접근 방법

- `x` 나 `y` 중 하나를 고정시켜 놓고 1사분면의 점의 개수를 구하고 4를 곱한다
- `y` 를 고정시킨다면, $x^2 + y^2 = r2^2$ → $x = \sqrt{r2^2 - y^2}$ 를 통해 `x` 의 최대값을 구하고
- `minX ~ maxX + 1` 만큼의 점이 가능하게 된다.
- `y = 1 ~ r2` 까지 위의 과정을 수행한다.

## 🧩 코드 구현

```java
class Solution {
    public long solution(int r1, int r2) {
        long answer = 0;

        long r1Sq = 1L * r1 * r1;
        long r2Sq = 1L * r2 * r2;

        for (long y = 1; y <= r2; y++) {
            long maxX = (long) Math.sqrt(r2Sq - y * y);

            long minX;
            if (y >= r1) {
                minX = 0;
            } else {
                minX = (long) Math.ceil(
                        Math.sqrt(r1Sq - y * y)
                );
            }

            answer += maxX - minX + 1;
        }

        return answer * 4;
    }
}
```

```
// 태그
Algorithm, java, Programmers
```
