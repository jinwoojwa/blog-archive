## 🔗 문제 링크

<a class="bookmark-card" href="https://school.programmers.co.kr/learn/courses/30/lessons/49995" target="_blank" rel="noopener noreferrer">
  <div class="bookmark-card-title">쿠키 구입</div>
  <div class="bookmark-card-desc">코딩테스트 연습 - 쿠키 구입 문제입니다.</div>
  <div class="bookmark-card-url">
    <img class="bookmark-card-favicon" src="https://www.google.com/s2/favicons?domain=programmers.co.kr" alt="" />
    <span>school.programmers.co.kr</span>
  </div>
</a>

## 💡 문제 파악

- 바구니 `1 ~ N`에 과자가 담겨 있다.
- 연속된 두 구간 `[l, m]`, `[m+1, r]`의 과자 합이 서로 같도록 두 사람에게 나눠줘야 한다.
- 이 조건을 만족하면서 한 사람에게 줄 수 있는 가장 많은 과자의 수(구간 합)를 구하는 문제이다.

## 🗝️ 접근 방법

- 중심점 `m`을 하나씩 고정하고, 그 왼쪽 구간 `[l, m]`과 오른쪽 구간 `[m+1, r]`을 양쪽 끝으로 넓혀가며 두 구간의 합이 같아지는 경우를 찾는 투 포인터 방식을 사용한다.
- 왼쪽 합이 더 크면 오른쪽 포인터(`r`)를 늘리고, 오른쪽 합이 더 크면 왼쪽 포인터(`l`)를 줄인다.
- 두 합이 같아질 때마다 그 값으로 최댓값을 갱신한다.
- 포인터가 배열의 양 끝에 도달했는데도 더 이상 늘릴 방향이 없다면 해당 중심점에 대한 탐색을 조기 종료한다.

## 🧩 코드 구현

```java
class Solution {
    public int solution(int[] cookie) {
        int maxCookieCnt = 0;

        for (int m = 0; m < cookie.length - 1; ++m) {
            int l = m;
            int r = m + 1;

            int leftSum = cookie[l];
            int rightSum = cookie[r];

            while (true) {
                if (leftSum == rightSum) {
                    maxCookieCnt = Math.max(maxCookieCnt, leftSum);
                }

                if (l == 0 && r == cookie.length - 1) break;

                if (leftSum >= rightSum) {
                    if (r == cookie.length - 1) break;
                    r++;
                    rightSum += cookie[r];
                } else {
                    if (l == 0) break;
                    l--;
                    leftSum += cookie[l];
                }
            }
        }
        return maxCookieCnt;
    }
}
```
