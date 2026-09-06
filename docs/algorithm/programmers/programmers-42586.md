---
title: 기능개발 - 프로그래머스
---

## 🔗 문제 링크

<a href="https://school.programmers.co.kr/learn/courses/30/lessons/42586" target="_blank" rel="noopener noreferrer" class="bookmark-link">
  <div class="bookmark-content">
    <div class="bookmark-title">프로그래머스 - 기능개발</div>
    <div class="bookmark-url">school.programmers.co.kr</div>
  </div>
</a>

## 💡 문제 파악

- 작업 진도, 작업 속도 리스트가 주어진다.
- 각 작업은 진도가 100%가 되어야 종료되며, 배포는 하루에 한 번만 가능하다.
- 각 배포마다 몇 개의 기능이 배포되는지를 반환하는 문제이다.

## 🗝️ 접근 방법

- 작업 진도, 속도로부터 각 작업별로 필요한 날짜를 구할 수 있다.
- 진도 리스트가 주어진 순서대로 배포가 되어야 하므로 큐(Queue) 자료구조를 사용한다.
- 각 작업마다 아래 알고리즘을 적용한다
  - 큐에 작업이 존재하고, 가장 앞에 있는 기능보다 남은 기한이 크다면
    - `date` 만큼 걸리는 현재 작업은 큐에 있는 작업들과 같이 배포될 수 없다.
    - 따라서, 결과 배열에 현재 큐에 들어 있는 작업의 개수를 넣고, 큐를 비워준다.
  - 만약 `date` 가 가장 앞에 있는 기능의 기한보다 작거나 같다면
    - 현재 작업은 큐에 있는 작업들과 같이 배포될 수 있다.

```java
if (!q.isEmpty() && q.peek() < date) {
    ans.add(q.size());
    q.clear();
}
q.offer(date);
```

## 🧩 코드 구현

```java
import java.util.*;

class Solution {
    public int[] solution(int[] progresses, int[] speeds) {
        Queue<Integer> q = new ArrayDeque<>();
        List<Integer> ans = new ArrayList<>();

        for (int i = 0; i < progresses.length; ++i) {
            double remain = (double)(100 - progresses[i]) / speeds[i];
            int date = (int)Math.ceil(remain);

            if (!q.isEmpty() && q.peek() < date) {
                ans.add(q.size());
                q.clear();
            }

            q.offer(date);
        }
        ans.add(q.size());

        return ans.stream().mapToInt(Integer::intValue).toArray();
    }
}
```
