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

- `0` 과 `1` 로만 이루어진 문자열 `x` 가 있을 때, 다음 연산을 원하는 만큼 할 수 있습니다.
  - 문자열 안에서 `110` 이라는 부분 문자열을 하나 뽑아 제거하고, 그 문자열을 **임의의 위치**에 다시 삽입합니다.
- 이 연산을 여러 번 적용해서 만들 수 있는 문자열 중 **사전 순으로 가장 앞서는 문자열**을 구합니다.
- 문자열 배열 `s` 가 주어지고, 각 문자열마다 답을 구해 배열로 반환합니다.
- `s` 의 길이와 각 문자열의 길이는 최대 1,000,000 이고, 모든 문자열의 길이의 합도 1,000,000 이하입니다.

## 🗝️ 접근 방법

- 사전 순으로 가장 앞서려면 **앞쪽에 `0` 이 최대한 많이 오고, `1` 이 최대한 뒤로** 가야 합니다.
- 먼저 문자열에서 `110` 을 **모두 제거**합니다.
  - 제거하면 앞뒤 문자가 붙으면서 새로운 `110` 이 생길 수 있으므로, 새로 생기는 것까지 계속 제거합니다.
  - 이 과정은 **스택**처럼 문자를 하나씩 쌓다가, 맨 위 세 글자가 `110` 이 되는 순간 빼내는 방식으로 한 번에 처리할 수 있습니다.
  - 제거한 `110` 의 개수는 `cnt` 에 저장합니다.
- `110` 을 모두 제거하고 남은 문자열에는 `110` 이 더 이상 없으므로, `1` 뒤에 `1` 과 `0` 이 이어지는 모양이 존재하지 않습니다. 따라서 남은 문자열은 `000...0111...1` 처럼 **`0` 들이 앞에, `1` 들이 뒤에** 있는 형태입니다.
- 제거한 `110` 들을 다시 넣을 위치는 **남은 문자열의 마지막 `0` 바로 뒤**입니다.
  - 앞쪽의 `0` 들을 건드리지 않으면서, `110` 이 `1` 들보다 앞에 오게 되어 가장 작은 문자열이 됩니다.
  - 더 앞에 넣으면 `0` 대신 `1` 이 먼저 나오는 자리가 생기고, 더 뒤에 넣으면 `1` 들 뒤에 `110` 이 와서 `0` 이 더 늦게 나옵니다. 둘 다 사전 순으로 불리합니다.
  - 남은 문자열에 `0` 이 없다면 `lastIndexOf` 가 `-1` 을 반환하므로 `idx + 1 = 0`, 즉 맨 앞에 삽입됩니다.
- `110` 을 `cnt` 개 이어 붙인 문자열을 한 번에 삽입하면 끝입니다.

```java
for (char c : str.toCharArray()) {
    sb.append(c);

    int len = sb.length();

    // 마지막 세 문자가 110인지 확인
    if (len >= 3 &&
        sb.charAt(len - 3) == '1' &&
        sb.charAt(len - 2) == '1' &&
        sb.charAt(len - 1) == '0') {

        // 110 제거
        sb.delete(len - 3, len);
        cnt++;
    }
}

// 남은 문자열에서 마지막 0의 위치
int idx = sb.lastIndexOf("0");

// 110을 마지막 0 뒤에 삽입
sb.insert(idx + 1, "110".repeat(cnt));
```

## ⏱️ 복잡도 분석

### 시간 복잡도

- 문자열의 각 문자는 한 번 쌓이고 많아야 한 번 제거되므로, 제거 과정은 `O(L)` 입니다. (`L` 은 문자열 길이입니다.)
- 마지막 `0` 을 찾는 데 `O(L)`, `110` 을 삽입하는 데 `O(L)` 이 걸립니다.
- 문자열 하나에 `O(L)` 이고, 모든 문자열 길이의 합이 최대 1,000,000 이므로 전체 시간 복잡도는 **`O(전체 문자열 길이의 합)`** 입니다.

### 공간 복잡도

- 스택 역할을 하는 `StringBuilder` 와 결과 문자열에 `O(L)` 의 공간을 사용합니다.

### 왜 이 방법으로 풀어야 하는가

- **실제로 `110` 을 옮겨 보며 모든 경우를 탐색하는 방법**
  - `110` 을 고르는 위치와 삽입하는 위치가 모두 `O(L)` 가지이고, 이를 여러 번 반복해야 하므로 경우의 수가 폭발적으로 늘어납니다. 길이가 최대 1,000,000 이므로 불가능합니다.
- **`110` 을 하나씩 찾아 문자열에서 제거하는 방법**
  - 매번 `110` 을 검색하고 중간을 지우면 한 번에 `O(L)` 이고, 제거 횟수가 최대 `L / 3` 이므로 `O(L²)` 이 되어 시간 초과가 납니다. 스택을 쓰면 새로 생기는 `110` 까지 한 번의 순회로 처리됩니다.
- **스택으로 제거 + 마지막 `0` 뒤에 삽입**
  - `110` 을 어디에 옮기든 결국 모두 같은 위치에 모이는 것이 최적이므로, 모든 `110` 을 먼저 빼 둔 뒤 가장 유리한 한 곳에 넣으면 됩니다. 한 번의 순회로 끝나서 `O(L)` 에 해결되므로 이 방법을 택했습니다.

## 🧩 코드 구현

```java
class Solution {
    
    private final String OOZ = "110";
    
    public String[] solution(String[] s) {
        String[] answer = new String[s.length];
        
        for (int i = 0; i < s.length; ++i) {
            String ans = solve(s[i]);
            answer[i] = ans;
        }
        
        return answer;
    }
    
    private String solve(String str) {
        StringBuilder sb = new StringBuilder();
        int cnt = 0;
        
        for (char c : str.toCharArray()) {
            sb.append(c);

            int len = sb.length();

            // 마지막 세 문자가 110인지 확인
            if (len >= 3 &&
                sb.charAt(len - 3) == '1' &&
                sb.charAt(len - 2) == '1' &&
                sb.charAt(len - 1) == '0') {

                // 110 제거
                sb.delete(len - 3, len);
                cnt++;
            }
        }

        // 남은 문자열에서 마지막 0의 위치
        int idx = sb.lastIndexOf("0");

        // 110을 마지막 0 뒤에 삽입
        sb.insert(idx + 1, "110".repeat(cnt));

        return sb.toString();
    }
}
```
