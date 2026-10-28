## 🔗 문제 링크

<a class="bookmark-card" href="https://school.programmers.co.kr/learn/courses/30/lessons/64064" target="_blank" rel="noopener noreferrer">
  <div class="bookmark-card-title">불량 사용자</div>
  <div class="bookmark-card-desc">코딩테스트 연습 - 불량 사용자 문제입니다.</div>
  <div class="bookmark-card-url">
    <img class="bookmark-card-favicon" src="https://www.google.com/s2/favicons?domain=programmers.co.kr" alt="" />
    <span>school.programmers.co.kr</span>
  </div>
</a>

## 💡 문제 파악

- `user_id` 는 응모자 아이디 목록이고, `banned_id` 는 불량 사용자 아이디 목록이다. `banned_id` 의 `*` 는 어떤 문자든 한 글자와 매칭된다.
- `banned_id` 의 각 항목에는 서로 다른 응모자 아이디가 하나씩 대응되어야 한다.
- 제재 대상이 될 수 있는 아이디 조합의 수를 구한다. **아이디 목록의 내용이 같으면 순서와 상관없이 같은 경우**로 본다.
- `user_id` 는 최대 8개라 완전 탐색이 가능하다.

## 🗝️ 접근 방법

- 각 `banned_id[i]` 마다 매칭되는 `user_id` 후보 리스트(`banList[i]`)를 먼저 만든다.
  - 길이가 같고, `*` 가 아닌 위치의 문자가 모두 같으면 매칭된다.
- 후보 리스트를 곱해서 경우의 수를 구하면 `banned_id` 끼리 겹치는 후보가 있을 때 중복으로 세게 된다. 그래서 곱셈 대신 **DFS 로 조합을 직접 만든다.**
  - `idx` 번째 `banned_id` 에 대해 후보 중 아직 쓰지 않은 아이디를 하나 고르고(`isUsed`), 다음 `banned_id` 로 넘어간다.
  - `idx == len` 이 되면 하나의 조합이 완성된 것이다.
- 같은 아이디 집합이 다른 순서로 만들어질 수 있으므로, 완성된 조합을 **정렬한 문자열**로 만들어 `Set` 에 넣어 중복을 제거한다.
  - 최종 답은 `Set` 의 크기다.

```java
for (String bannedId : banList[idx]) {
    if (isUsed.get(bannedId)) continue;

    isUsed.put(bannedId, true);
    dfs(idx + 1, str + "+" + bannedId);
    isUsed.put(bannedId, false);
}
```

## 🧩 코드 구현

```java
import java.util.*;

class Solution {

    int len;
    List<String>[] banList;
    Set<String> answerSet = new HashSet<>();
    Map<String, Boolean> isUsed = new HashMap<>();

    public int solution(String[] user_id, String[] banned_id) {
        len = banned_id.length;

        banList = new List[banned_id.length];
        for (int i = 0; i < len; i++) {
            banList[i] = new ArrayList<>();
        }

        for (String userId : user_id) {
            isUsed.put(userId, false);
        }

        for (int i = 0; i < len; ++i) {
            for (String userId : user_id) {
                if (isMatch(banned_id[i], userId)) banList[i].add(userId);
            }
        }

        dfs(0, "");

        return answerSet.size();
    }

    private void dfs(int idx, String str) {
        if (idx == len) {
            answerSet.add(sortById(str));
            return;
        }

        for (String bannedId : banList[idx]) {
            if (isUsed.get(bannedId)) continue;

            isUsed.put(bannedId, true);

            dfs(idx + 1, str + "+" + bannedId);

            isUsed.put(bannedId, false);
        }
    }

    private boolean isMatch(String bannedId, String userId) {
        if (bannedId.length() != userId.length()) return false;

        for (int i = 0; i < bannedId.length(); ++i) {
            char b = bannedId.charAt(i);
            char u = userId.charAt(i);

            if (b != '*' && b != u) return false;
        }
        return true;
    }

    private String sortById(String str) {
        String[] temp = str.split("\\+");
        Arrays.sort(temp);
        return String.join("+", temp);
    }
}
```
