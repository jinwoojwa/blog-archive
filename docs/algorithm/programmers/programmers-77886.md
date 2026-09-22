## 🔗 문제 링크

<a class="bookmark-card" href="https://school.programmers.co.kr/learn/courses/30/lessons/77886" target="_blank" rel="noopener noreferrer">
  <div class="bookmark-card-title">110 옮기기</div>
  <div class="bookmark-card-desc">코딩테스트 연습 - 110 옮기기 문제입니다.</div>
  <div class="bookmark-card-url">
    <img class="bookmark-card-favicon" src="https://www.google.com/s2/favicons?domain=programmers.co.kr" alt="" />
    <span>school.programmers.co.kr</span>
  </div>
</a>

## 💡 문제 파악

- 0과 1로만 이루어진 문자열이 여러 개 주어진다.
- 문자열에서 `"110"`을 뽑아내어 다른 위치에 그대로 다시 끼워 넣는 작업을 원하는 만큼 반복할 수 있다.
- 이 작업을 반복해서 만들 수 있는 문자열 중 사전 순으로 가장 앞서는 문자열을 각각 구하는 문제이다.

## 🗝️ 접근 방법

- 사전순으로 가장 앞서게 만들려면, `"110"`을 최대한 앞쪽에서 제거하고, 남은 문자열에서 가장 마지막 `0` 바로 뒤에 다시 채워 넣는 것이 유리하다.
- 스택을 이용해 문자열을 순회하면서 `"110"` 패턴이 만들어질 때마다 스택에서 제거하고 제거 횟수를 센다.
- 모든 `"110"`을 제거한 뒤 남은 문자열에서 마지막 `'0'`의 위치를 찾는다.
  - `'0'`이 없다면 문자열의 맨 앞에 `"110"`들을 모두 붙인다.
  - `'0'`이 있다면 그 위치 바로 뒤에 `"110"`들을 모두 끼워 넣는다.

## 🧩 코드 구현

```java
class Solution {

    public String[] solution(String[] s) {
        String[] answer = new String[s.length];

        for (int i = 0; i < s.length; i++) {
            StringBuilder stack = new StringBuilder();
            int count = 0;

            for (char c : s[i].toCharArray()) {
                stack.append(c);

                int len = stack.length();

                if (len >= 3 &&
                        stack.charAt(len - 3) == '1' &&
                        stack.charAt(len - 2) == '1' &&
                        stack.charAt(len - 1) == '0') {

                    stack.delete(len - 3, len);
                    count++;
                }
            }

            int idx = stack.lastIndexOf("0");

            StringBuilder result = new StringBuilder();

            if (idx == -1) {
                for (int j = 0; j < count; j++) {
                    result.append("110");
                }
                result.append(stack);
            } else {
                result.append(stack.substring(0, idx + 1));

                for (int j = 0; j < count; j++) {
                    result.append("110");
                }
                result.append(stack.substring(idx + 1));
            }
            answer[i] = result.toString();
        }
        return answer;
    }
}
```
