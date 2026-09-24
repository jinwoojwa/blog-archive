## 🔗 문제 링크

<a class="bookmark-card" href="https://school.programmers.co.kr/learn/courses/30/lessons/150368" target="_blank" rel="noopener noreferrer">
  <div class="bookmark-card-title">이모티콘 할인행사</div>
  <div class="bookmark-card-desc">코딩테스트 연습 - 이모티콘 할인행사 문제입니다.</div>
  <div class="bookmark-card-url">
    <img class="bookmark-card-favicon" src="https://www.google.com/s2/favicons?domain=programmers.co.kr" alt="" />
    <span>school.programmers.co.kr</span>
  </div>
</a>

## 💡 문제 파악

- 여러 이모티콘의 가격이 주어지고, 각 이모티콘마다 10%, 20%, 30%, 40% 중 하나의 할인율을 정해야 한다.
- 각 사용자는 자신이 관심 있는 할인율 이상인 이모티콘만 구매하며, 구매한 총액이 사용자의 기준 금액 이상이면 구매를 취소하고 대신 이모티콘 플러스 서비스에 가입한다.
- 이모티콘 플러스 가입자 수가 최대가 되도록 하면서, 그 다음으로 판매액이 최대가 되는 조합을 찾아 `[가입자 수, 판매액]`을 반환하는 문제이다.

## 🗝️ 접근 방법

- 이모티콘 개수는 최대 7개, 할인율 선택지는 4가지이므로 전체 경우의 수는 `4^7`(약 1만 6천)로 완전 탐색이 가능하다.
- `DFS`로 각 이모티콘의 할인율을 하나씩 정해나간 뒤, 할인율 조합이 모두 정해지면 모든 사용자를 순회하며 가입자 수와 판매액을 계산한다.
- 계산된 가입자 수가 기존 최댓값보다 크거나, 같으면서 판매액이 더 크면 결과를 갱신한다.

## 🧩 코드 구현

```java
import java.util.*;

class Solution {

    static int[] result = new int[]{0, 0};

    public int[] solution(int[][] users, int[] emoticons) {
        chooseOptions(users, emoticons, 0, new int[emoticons.length]);
        return result;
    }

    private void chooseOptions(int[][] users, int[] emoticons, int idx, int[] choose) {
        if (idx == emoticons.length) {
            calcCost(users, emoticons, choose);
            return;
        }

        for (int sale = 1; sale <= 4; ++sale) {
            choose[idx] = sale * 10;
            chooseOptions(users, emoticons, idx + 1, choose);
        }
    }

    private void calcCost(int[][] users, int[] emoticons, int[] choose) {
        int serviceSubscriber = 0;
        int priceSum = 0;

        for (int[] user : users) {
            int userRate = user[0];
            int userPrice = user[1];

            int price = 0;
            for (int i = 0; i < emoticons.length; ++i) {
                int rate = choose[i];

                if (userRate > rate) continue;

                price += emoticons[i] - (emoticons[i] * rate / 100);
            }
            if (userPrice <= price) serviceSubscriber++;
            else priceSum += price;
        }
        if (result[0] < serviceSubscriber) {
            result = new int[]{serviceSubscriber, priceSum};
        } else if (result[0] == serviceSubscriber && result[1] < priceSum) {
            result = new int[]{serviceSubscriber, priceSum};
        }
    }
}
```
