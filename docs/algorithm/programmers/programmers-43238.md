## 🔗 문제 링크

<a class="bookmark-card" href="https://school.programmers.co.kr/learn/courses/30/lessons/43238" target="_blank" rel="noopener noreferrer">
  <div class="bookmark-card-title">입국심사</div>
  <div class="bookmark-card-desc">코딩테스트 연습 - 입국심사 문제입니다.</div>
  <div class="bookmark-card-url">
    <img class="bookmark-card-favicon" src="https://www.google.com/s2/favicons?domain=programmers.co.kr" alt="" />
    <span>school.programmers.co.kr</span>
  </div>
</a>

## 💡 문제 파악

- `n` 명이 입국심사를 기다리고 있고, 심사관은 여러 명이다. `times[i]` 는 `i` 번째 심사관이 한 명을 심사하는 데 걸리는 시간이다.
- 한 심사대에서는 한 번에 한 명만 심사할 수 있고, 비어 있는 심사대가 있으면 그쪽으로 가서 심사받을 수 있다.
- 모든 사람이 심사를 받는 데 걸리는 시간의 **최솟값**을 구한다.

## 🗝️ 접근 방법

- "모든 사람을 심사하는 최소 시간"을 직접 구하기보다, **"시간 `T` 안에 `n` 명을 모두 심사할 수 있는가?"** 를 판별하는 문제로 바꾼다. (결정 문제로 변환)
  - 시간 `T` 동안 `i` 번째 심사관은 `T / times[i]` 명을 심사할 수 있으므로, 이 값을 모든 심사관에 대해 더한 값이 `n` 이상이면 가능하다.
- `T` 가 커질수록 심사할 수 있는 인원은 줄지 않으므로 가능 여부가 `false → true` 로 **단조롭게** 바뀐다. 따라서 **이분 탐색**으로 `true` 가 되는 최소 `T` 를 찾는다.
  - `start = 0`, `end = Long.MAX_VALUE` 로 두고 `mid` 에서 가능하면 `end = mid`, 불가능하면 `start = mid + 1` 로 좁힌다.
  - `start == end` 가 되는 지점이 가능한 최소 시간이다.
- `n` 은 최대 10억, `times[i]` 도 최대 10억이라 답이 `int` 범위를 넘으므로 `long` 을 사용한다.
- `isFinish` 에서 합산 도중 `cnt >= n` 이 되면 바로 `true` 를 반환해 불필요한 계산과 `long` 오버플로를 방지한다.

```java
while (start < end) {
    long mid = start + (end - start) / 2;

    if (isFinish(mid, n, times)) end = mid;
    else start = mid + 1;
}
```

## 🧩 코드 구현

```java
class Solution {
    public long solution(int n, int[] times) {
        long start = 0;
        long end = Long.MAX_VALUE;

        while (start < end) {
            long mid = start + (end - start) / 2;

            if (isFinish(mid, n, times)) end = mid;
            else start = mid + 1;
        }

        return start;
    }

    private boolean isFinish(long time, int n, int[] times) {
        long cnt = 0;
        for (int immigrationOfficer : times) {
            if (cnt >= n) return true;
            cnt += time / immigrationOfficer;
        }
        return (cnt >= n) ? true : false;
    }
}
```
