## 🔗 문제 링크

<a class="bookmark-card" href="https://school.programmers.co.kr/learn/courses/30/lessons/140107" target="_blank" rel="noopener noreferrer">
  <div class="bookmark-card-title">점 찍기</div>
  <div class="bookmark-card-desc">코딩테스트 연습 - 점 찍기 문제입니다.</div>
  <div class="bookmark-card-url">
    <img class="bookmark-card-favicon" src="https://www.google.com/s2/favicons?domain=programmers.co.kr" alt="" />
    <span>school.programmers.co.kr</span>
  </div>
</a>

## 💡 문제 파악

- x축, y축 모두 0부터 `k`씩 증가하는 좌표 `(a×k, b×k)`에 점을 찍을 수 있다(`a, b`는 0 이상의 정수).
- 원점으로부터의 거리가 `d`를 넘지 않는 위치에만 점을 찍을 수 있다.
- `k`와 `d`가 주어졌을 때, 찍을 수 있는 점의 총 개수를 구하는 문제이다.
- `a, b`가 모두 양의 정수 범위(0 이상)이므로 1사분면과 두 축 위의 점만 고려하면 된다.

## 🗝️ 접근 방법

- y좌표를 `0`부터 `k`씩 늘려가면서, 원점과의 거리가 `d`를 넘지 않는 x좌표의 최댓값을 계산한다.
- `x^2 = d^2 - y^2`이므로, `y`가 정해지면 `x`의 최댓값은 `sqrt(d^2 - y^2)`으로 구할 수 있다.
- 이 최대 x좌표를 `k`로 나누고 1을 더하면(`x = 0`도 포함해야 하므로) 해당 y좌표 위에서 찍을 수 있는 점의 개수가 된다.
- 모든 y좌표에 대해 이 값을 더하면 전체 점의 개수가 된다.

## 🧩 코드 구현

```java
class Solution {
    public long solution(int k, int d) {
        long dd = 1L * d * d;

        long dotCnt = 0;
        for (int y = 0; y <= d; y += k) {
            long xx = dd - 1L * y * y;
            long maxX = (long) Math.sqrt(xx);

            dotCnt += maxX / k + 1;
        }
        return dotCnt;
    }
}
```
