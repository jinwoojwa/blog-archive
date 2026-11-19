## 🔗 문제 링크

<a class="bookmark-card" href="https://school.programmers.co.kr/learn/courses/30/lessons/60057" target="_blank" rel="noopener noreferrer">
  <div class="bookmark-card-title">문자열 압축</div>
  <div class="bookmark-card-desc">코딩테스트 연습 - 문자열 압축 문제입니다.</div>
  <div class="bookmark-card-url">
    <img class="bookmark-card-favicon" src="https://www.google.com/s2/favicons?domain=programmers.co.kr" alt="" />
    <span>school.programmers.co.kr</span>
  </div>
</a>

## 💡 문제 파악

- 문자열을 **같은 길이 단위로 잘라** 앞에서부터 연속해서 같은 조각이 반복되면 `반복 횟수 + 조각` 으로 줄여 표현합니다. 반복 횟수가 1이면 숫자를 붙이지 않습니다.
  - 예를 들어 `aabbaccc` 를 1개 단위로 자르면 `2a2ba3c` (길이 7) 입니다.
- 자르는 단위를 바꿔 가며 압축했을 때, **가장 짧은 압축 문자열의 길이**를 구합니다.
- 문자열의 길이 `s` 는 1 이상 1,000 이하입니다.

## 🗝️ 접근 방법

- 단위 길이마다 압축 결과의 길이가 단조롭게 변하지 않으므로, 가능한 **모든 단위 길이를 하나씩 시도**하고 그중 최솟값을 고릅니다.
- 단위 길이는 `1` 부터 `s.length() / 2` 까지만 확인하면 됩니다. 단위가 문자열 길이의 절반보다 크면 같은 조각이 두 번 이상 나올 수 없어 압축이 일어나지 않기 때문입니다.
- `compressStr(s, sliceSize)` 는 `sliceSize` 단위로 문자열을 잘라 압축한 길이를 구합니다.
  - `idx` 를 `sliceSize` 씩 늘리며 `cur` 조각을 잘라냅니다. 마지막 조각은 길이가 모자랄 수 있으므로 `Math.min` 으로 끝 인덱스를 제한합니다.
  - `cur` 이 직전 조각(`prev`)과 같으면 `prevCnt` 만 늘립니다.
  - 다르면 직전 조각의 압축 결과 길이를 `lenOfStr` 에 더하고, `cur` 을 새 `prev` 로 바꿉니다.
    - 압축 결과 길이 = (`prevCnt` 가 1이 아니면 숫자의 자릿수) + `prev` 의 길이
    - 숫자는 `10` 이상이면 두 자리 이상이므로 `String.valueOf(prevCnt).length()` 로 자릿수를 구합니다.
  - 반복문이 끝난 뒤에도 마지막 `prev` 가 아직 더해지지 않았으므로 한 번 더 더해 줍니다.
- 문자열 길이가 1이면 반복문이 한 번도 돌지 않아 `answer` 가 초기값 `1001` 로 남으므로, 먼저 `1` 을 반환합니다.

```java
if (prev.equals(cur)) {
    prevCnt++;
}
else {
    if (prevCnt != 1) lenOfStr += String.valueOf(prevCnt).length();
    lenOfStr += prev.length();

    prev = cur;
    prevCnt = 1;
}
```

## ⏱️ 복잡도 분석

### 시간 복잡도

- 단위 길이 `sliceSize` 에서 조각 수는 `n / sliceSize` 이고, 조각마다 `substring` 과 `equals` 가 `O(sliceSize)` 이므로 한 번의 `compressStr` 는 **`O(n)`** 입니다.
- 이를 `sliceSize = 1 ~ n / 2` 까지 반복하므로 전체 시간 복잡도는 **`O(n²)`** 입니다. `n = 1,000` 이면 약 `1,000 × 500 = 50만` 번입니다.

### 공간 복잡도

- 조각 두 개(`prev`, `cur`)만 보관하므로 `O(n)` 이하이고, 입력 외에 추가로 큰 공간은 쓰지 않습니다.

### 왜 이 방법으로 풀어야 하는가

- **최적의 단위 길이를 계산이나 규칙으로 바로 찾는 방법**
  - 단위가 길다고 항상 더 잘 압축되는 것도, 짧다고 항상 그런 것도 아닙니다. 예를 들어 `ababcdcdababcdcd` (길이 16) 는 1개 단위로는 전혀 줄지 않지만, 8개 단위로는 `2ababcdcd` (길이 9) 로 크게 줄어듭니다. 단위에 따른 결과가 단조롭지 않으므로 **모두 시도하는 것이 정답에 가장 안전**합니다.
- **모든 단위를 시도하는 완전탐색의 비용**
  - 위에서 계산한 것처럼 약 50만 번 정도라 `n ≤ 1,000` 에서 충분히 빠릅니다. 더 정교한 방법(예: KMP 로 반복 구간을 찾는 방식)은 구현이 훨씬 복잡한데, 이 제약에서는 얻는 이득이 없습니다.
- **문자열 전체를 직접 만들어 길이를 재는 방법**
  - 압축 문자열을 `StringBuilder` 로 만들어도 정답은 같지만, 이 풀이는 문자열을 만들지 않고 **길이만 누적**하므로 불필요한 문자열 생성이 없습니다.

## 🧩 코드 구현

```java
class Solution {
    int answer = 1001;
    
    public int solution(String s) {
        
        if (s.length() == 1) return 1;
        
        for (int sliceSize = 1; sliceSize <= s.length() / 2; ++sliceSize) {
            compressStr(s, sliceSize);
        }
        
        return answer;
    }
    
    private void compressStr(String s, int sliceSize) {
        int lenOfStr = 0;
        
        String prev = "";
        int prevCnt = 1;
        
        for (int idx = 0; idx < s.length(); idx += sliceSize) {
            int endIdx = Math.min(idx + sliceSize, s.length());
            
            String cur = s.substring(idx, endIdx);
            
            if (prev.equals(cur)) {
                prevCnt++;
            }
            else {
                if (prevCnt != 1) lenOfStr += String.valueOf(prevCnt).length();
                lenOfStr += prev.length();
                
                prev = cur;
                prevCnt = 1;
            }
        }
        
        if (prevCnt != 1) {
            lenOfStr += String.valueOf(prevCnt).length();
        }

        lenOfStr += prev.length();
        
        answer = Math.min(answer, lenOfStr);
    }
}
```
