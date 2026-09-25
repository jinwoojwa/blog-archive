## 🔗 문제 링크

<a class="bookmark-card" href="https://school.programmers.co.kr/learn/courses/30/lessons/12899" target="_blank" rel="noopener noreferrer">
  <div class="bookmark-card-title">124 나라의 숫자</div>
  <div class="bookmark-card-desc">코딩테스트 연습 - 124 나라의 숫자 문제입니다.</div>
  <div class="bookmark-card-url">
    <img class="bookmark-card-favicon" src="https://www.google.com/s2/favicons?domain=programmers.co.kr" alt="" />
    <span>school.programmers.co.kr</span>
  </div>
</a>

## 💡 문제 파악

- 124 나라는 모든 수를 `1, 2, 4` 세 개의 숫자만으로 표현한다.
- 자연수 `n`이 주어지면, 이를 124 나라 표기법으로 변환한 값을 반환해야 한다.

## 🗝️ 접근 방법

- 124 나라 숫자는 각 자리마다 `1, 2, 4` 중 하나가 올 수 있으므로, 자릿수가 `d`인 숫자는 정확히 `3^d`개 존재한다.
  - 1자리: 3개(`1, 2, 4`), 2자리: 9개(`11 ~ 44`), 3자리: 27개, ...
- 따라서 `n`번째 숫자가 몇 자리 수인지 먼저 찾는다.
  - `n`이 현재 자릿수(`digits`)의 전체 개수(`3^digits`)보다 크면, 그 자릿수 구간을 모두 지나쳤다는 뜻이므로 `n`에서 `3^digits`를 빼고 자릿수를 하나 늘린다.
  - 반복이 끝나면 `n`은 찾아낸 자릿수(`digits`) 구간 안에서 몇 번째인지를 나타내는 값(1-indexed)이 된다.
- `n--`으로 0-indexed로 바꾸면, 이제 `n`은 `digits`자리 124 나라 숫자들을 사전순으로 나열했을 때의 인덱스(`0 ~ 3^digits - 1`)가 된다.
  - 이 범위는 정확히 `digits`자리 3진수(`0 ~ 3^digits - 1`)와 개수가 같으므로, `n`을 3진수로 변환하면서 각 자리 값 `0, 1, 2`를 `1, 2, 4`로 매핑하면 124 나라 숫자를 만들 수 있다(일종의 전단사 진법 변환).
- 최하위 자리부터 `n % 3`을 구해 매핑된 문자를 이어 붙이고 `n`을 3으로 나누는 과정을 `digits`번 반복한 뒤, 뒤집어서(`reverse`) 최상위 자리가 앞에 오도록 만든다.

## 🧩 코드 구현

```java
class Solution {
    public String solution(int n) {
        int digits = 1;
        while (n > Math.pow(3, digits)) {
            n -= Math.pow(3, digits);
            digits++;
        }

        n--;

        StringBuilder answer = new StringBuilder();

        for (int i = 0; i < digits; ++i) {
            int remain = n % 3;

            if (remain == 0) answer.append("1");
            else if (remain == 1) answer.append("2");
            else answer.append("4");

            n /= 3;
        }
        return answer.reverse().toString();
    }
}
```
