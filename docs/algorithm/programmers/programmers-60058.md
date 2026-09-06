## 🔗 문제 링크

<a class="bookmark-card" href="https://school.programmers.co.kr/learn/courses/30/lessons/60058" target="_blank" rel="noopener noreferrer">
  <div class="bookmark-card-title">괄호 변환</div>
  <div class="bookmark-card-desc">코딩테스트 연습 - 괄호 변환 문제입니다.</div>
  <div class="bookmark-card-url">
    <img class="bookmark-card-favicon" src="https://www.google.com/s2/favicons?domain=programmers.co.kr" alt="" />
    <span>school.programmers.co.kr</span>
  </div>
</a>

## 💡 문제 파악

- `'('` 와 `')'` 로만 이루어진 문자열 `p` 가 주어진다. (열고 닫는 괄호 개수는 항상 같다)
- 문제에서 정의한 변환 규칙에 따라 `p` 를 **올바른 괄호 문자열**로 바꿔서 반환해야 한다.
- 올바른 괄호 문자열이란, 여는 괄호와 닫는 괄호가 항상 짝을 이루며 문자열을 앞에서부터 읽을 때 닫는 괄호가 여는 괄호보다 먼저 나오는 순간이 없는 문자열이다.

## 🗝️ 접근 방법

문제에서 제시한 재귀 변환 알고리즘을 그대로 구현한다.

1. `p` 가 빈 문자열이면 그대로 반환한다.
2. `p` 를 **균형잡힌 괄호 문자열**인 `u`, `v` 로 분리한다.
   - 앞에서부터 괄호 개수를 세다가 여는 괄호 수와 닫는 괄호 수가 처음으로 같아지는 지점까지가 `u`, 나머지가 `v` 이다.
3. `u` 가 이미 **올바른 괄호 문자열**이라면 `u + solution(v)` 를 반환한다.
4. `u` 가 올바른 괄호 문자열이 아니라면
   - `"(" + solution(v) + ")"` 를 만들고
   - `u` 의 첫 번째와 마지막 문자를 제거한 뒤, 남은 문자열의 괄호 방향을 모두 뒤집어 이어 붙인다.
5. 위 과정을 재귀적으로 반복해 최종 결과 문자열을 얻는다.

균형잡힌 문자열로 분리하는 부분은 괄호 개수 카운트만으로 판단할 수 있다.

```java
if (open == close) {
    return new String[]{
        str.substring(0, i + 1),
        str.substring(i + 1)
    };
}
```

올바른 괄호 문자열인지는 여는 괄호를 `+1`, 닫는 괄호를 `-1` 로 두고 누적합이 음수가 되는 순간이 없고 최종합이 `0` 인지로 판단한다.

```java
if (u.charAt(i) == '(') count++;
else count--;

if (count < 0) return false;
```

## 🧩 코드 구현

```java
import java.util.*;

class Solution {
    public String solution(String p) {
        if (p.isBlank()) return p;
        
        String[] result = splitString(p);
        String u = result[0];
        String v = result[1];
        
        // u가 올바른 괄호 문자열인 경우
        if (isCorrect(u)) {
            return u + solution(v);
        }
        
        // u가 올바른 괄호 문자열이 아닌 경우
        StringBuilder sb = new StringBuilder();

        sb.append("(");
        sb.append(solution(v));
        sb.append(")");
        
        // u의 첫 번째와 마지막 문자를 제거하고 나머지 괄호 방향 뒤집기
        for (int i = 1; i < u.length() - 1; i++) {
            if (u.charAt(i) == '(') sb.append(')');
            else sb.append('(');
        }

        return sb.toString();
        
    }
    
    private boolean isCorrect(String str) {
        int count = 0;

        for (int i = 0; i < str.length(); i++) {
            if (str.charAt(i) == '(') count++;
            else count--;

            if (count < 0) return false;
        }
        return count == 0;
    }
    
    private String[] splitString(String str) {
        int open = 0, close = 0;

        for (int i = 0; i < str.length(); ++i) {
            if (str.charAt(i) == '(') open++;
            else close++;

            if (open == close) {
                return new String[]{
                    str.substring(0, i + 1),
                    str.substring(i + 1)
                };
            }
        }
        return new String[]{str, ""};
    }
}
```
