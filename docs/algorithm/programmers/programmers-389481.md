## 🔗 문제 링크

<a class="bookmark-card" href="https://school.programmers.co.kr/learn/courses/30/lessons/389481" target="_blank" rel="noopener noreferrer">
  <div class="bookmark-card-title">봉인된 주문</div>
  <div class="bookmark-card-desc">코딩테스트 연습 - 봉인된 주문 문제입니다.</div>
  <div class="bookmark-card-url">
    <img class="bookmark-card-favicon" src="https://www.google.com/s2/favicons?domain=programmers.co.kr" alt="" />
    <span>school.programmers.co.kr</span>
  </div>
</a>

## 💡 문제 파악

- `a, b, c, ... z, aa, ab, ...` 순서로 무한히 이어지는 문자열들 중 일부(`bans`)가 삭제되어 있다.
- 삭제된 문자열들을 제외하고 남은 문자열들을 순서대로 나열했을 때, `n`번째 문자열이 무엇인지 구하는 문제이다.

## 🗝️ 접근 방법

- 문자열의 순서를 숫자로 매핑하면 `a = 1, b = 2, ..., z = 26, aa = 27, ...`처럼 사실상 26진수 체계가 된다.
- 구하고 싶은 것은 "삭제 후 n번째 문자열"이므로, 어떤 숫자 `x`까지의 원래 문자열 개수에서 그 이하의 금지 문자열 개수(`ban`)를 빼면 살아남은 문자열 개수(`x - ban`)를 구할 수 있다.
- 이분 탐색으로 "살아남은 개수가 `n` 이상이 되는 최소의 `x`"를 찾는다.
  - `mid` 이하에 포함된 금지 문자열 개수는 정렬된 `banned` 배열에서 `upperBound`(이분 탐색)로 구한다.
- 찾은 `x`를 다시 문자열로 변환해서 반환한다.

## 🧩 코드 구현

```java
import java.util.*;

class Solution {

    private final int ALPHABET_CNT = 26;

    public String solution(long n, String[] bans) {
        long[] banned = new long[bans.length];

        for (int i = 0; i < bans.length; i++) {
            banned[i] = toNumber(bans[i]);
        }

        Arrays.sort(banned);

        long left = 1;
        long right = Long.MAX_VALUE / 4;

        while (left < right) {
            long mid = left + (right - left) / 2;

            long removed = upperBound(banned, mid);
            long remain = mid - removed;

            if (remain >= n) {
                right = mid;
            } else {
                left = mid + 1;
            }
        }

        return toString(left);
    }

    private long toNumber(String s) {
        long num = 0;

        for (int i = 0; i < s.length(); ++i) {
            num = num * ALPHABET_CNT + (s.charAt(i) - 'a' + 1);
        }

        return num;
    }

    private String toString(long num) {
        StringBuilder sb = new StringBuilder();

        while (num > 0) {
            num -= 1;
            sb.append((char) ('a' + (num % ALPHABET_CNT)));
            num /= ALPHABET_CNT;
        }

        return sb.reverse().toString();
    }

    private int upperBound(long[] arr, long target) {
        int left = 0;
        int right = arr.length;

        while (left < right) {
            int mid = (left + right) / 2;

            if (arr[mid] <= target) {
                left = mid + 1;
            } else {
                right = mid;
            }
        }

        return left;
    }
}
```
