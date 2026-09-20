## 🔗 문제 링크

<a class="bookmark-card" href="https://school.programmers.co.kr/learn/courses/30/lessons/77486" target="_blank" rel="noopener noreferrer">
  <div class="bookmark-card-title">다단계 칫솔 판매</div>
  <div class="bookmark-card-desc">코딩테스트 연습 - 다단계 칫솔 판매 문제입니다.</div>
  <div class="bookmark-card-url">
    <img class="bookmark-card-favicon" src="https://www.google.com/s2/favicons?domain=programmers.co.kr" alt="" />
    <span>school.programmers.co.kr</span>
  </div>
</a>

## 💡 문제 파악

- 트리 형태의 조직 정보(`enroll`, `referral`)와 판매 정보(`seller`, `amount`)가 주어진다.
- 각 판매원은 자신이 번 돈의 10%를 자신을 추천한 사람(부모 노드)에게 지급하고, 나머지를 자신이 가진다.
- 부모에게 전달된 돈 역시 같은 규칙으로 다시 10%가 그 위 부모에게 전달된다.
- 모든 판매가 끝난 뒤, 각 조직원이 정확히 얼마를 벌었는지 배열로 반환하는 문제이다.

## 🗝️ 접근 방법

- 각 판매원의 부모(추천인)를 저장하는 `parentMap`을 먼저 구성한다.
- 판매 정보 하나마다 판매 금액을 들고 부모 노드로 거슬러 올라가면서 10%씩 분배한다.
  - 분배할 금액이 1원 미만(정수 나눗셈 결과 0)이 되면 더 이상 위로 전달하지 않고 종료한다.
  - 루트(`"-"`)에 도달해도 종료한다.
- 금액을 다룰 때 중간 계산값이 `int` 범위를 넘어설 수 있으므로 `long`으로 누적한 뒤 마지막에 변환한다.

## 🧩 코드 구현

```java
import java.util.*;

class Solution {
    public int[] solution(String[] enroll, String[] referral, String[] seller, int[] amount) {
        Map<String, String> parentMap = new HashMap<>();
        for (int i = 0; i < enroll.length; ++i) {
            parentMap.put(enroll[i], referral[i]);
        }

        Map<String, Long> profitMap = new HashMap<>();
        for (int i = 0; i < seller.length; ++i) {
            String cur = seller[i];
            long money = (long) amount[i] * 100;

            while (!cur.equals("-")) {
                long distributedMoney = money / 10;
                long myMoney = money - distributedMoney;

                profitMap.merge(cur, myMoney, Long::sum);

                if (distributedMoney == 0) break;

                cur = parentMap.getOrDefault(cur, "-");
                money = distributedMoney;
            }
        }

        int[] answer = new int[enroll.length];
        for (int i = 0; i < enroll.length; ++i) {
            answer[i] = profitMap.getOrDefault(enroll[i], 0L).intValue();
        }
        return answer;
    }
}
```
