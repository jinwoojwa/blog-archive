## 🔗 문제 링크

<a class="bookmark-card" href="https://school.programmers.co.kr/learn/courses/30/lessons/68646" target="_blank" rel="noopener noreferrer">
  <div class="bookmark-card-title">풍선 터트리기</div>
  <div class="bookmark-card-desc">코딩테스트 연습 - 풍선 터트리기 문제입니다.</div>
  <div class="bookmark-card-url">
    <img class="bookmark-card-favicon" src="https://www.google.com/s2/favicons?domain=programmers.co.kr" alt="" />
    <span>school.programmers.co.kr</span>
  </div>
</a>

## 💡 문제 파악

- 일렬로 놓인 `n`개의 풍선 중, 인접한 두 풍선 중 하나를 계속 터뜨려서 결국 1개만 남기는 과정을 반복한다.
- 이때 최종적으로 남을 수 있는 풍선의 개수(경우의 수가 아니라, 마지막까지 살아남을 가능성이 있는 풍선 개수)를 구하는 문제이다.
- `n`이 최대 100만이므로 `O(N^2)` 이상의 완전 탐색으로는 시간 초과가 발생한다.

## 🗝️ 접근 방법

- 어떤 풍선 `i`가 끝까지 살아남으려면, 왼쪽에서 온 풍선이든 오른쪽에서 온 풍선이든 하나만 상대하면 된다는 점에 착안한다.
- 즉 `i`번 풍선은 "왼쪽 구간에서 살아남은 하나" 또는 "오른쪽 구간에서 살아남은 하나"와만 비교해서 이기면 된다.
- `i`번 풍선이 왼쪽 구간의 최솟값(`leftMin[i]`)보다 작거나 같거나, 오른쪽 구간의 최솟값(`rightMin[i]`)보다 작거나 같으면 마지막까지 살아남을 수 있다.
- `leftMin`, `rightMin` 배열을 각각 왼쪽에서, 오른쪽에서 한 번씩 순회하며 누적 최솟값으로 미리 구해두면 `O(N)`에 판별할 수 있다.

## 🧩 코드 구현

```java
class Solution {
    public int solution(int[] a) {
        int answer = 0;
        int n = a.length;

        int[] leftMin = new int[n];
        int[] rightMin = new int[n];

        leftMin[0] = a[0];
        for (int i = 1; i < n; ++i) {
            leftMin[i] = Math.min(leftMin[i - 1], a[i]);
        }

        rightMin[n - 1] = a[n - 1];
        for (int i = n - 2; i >= 0; --i) {
            rightMin[i] = Math.min(rightMin[i + 1], a[i]);
        }

        for (int i = 0; i < n; ++i) {
            if (a[i] <= leftMin[i] || a[i] <= rightMin[i]) {
                answer++;
            }
        }

        return answer;
    }
}
```
