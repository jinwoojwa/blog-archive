## 🔗 문제 링크

<a class="bookmark-card" href="https://school.programmers.co.kr/learn/courses/30/lessons/152995" target="_blank" rel="noopener noreferrer">
  <div class="bookmark-card-title">인사고과</div>
  <div class="bookmark-card-desc">코딩테스트 연습 - 인사고과 문제입니다.</div>
  <div class="bookmark-card-url">
    <img class="bookmark-card-favicon" src="https://www.google.com/s2/favicons?domain=programmers.co.kr" alt="" />
    <span>school.programmers.co.kr</span>
  </div>
</a>

## 💡 문제 파악

- 각 사원마다 근무 태도 점수와 동료 평가 점수가 주어진다.
- 어떤 사원이 다른 임의의 사원보다 두 점수가 모두 낮다면, 그 사원은 인센티브 대상(석차)에서 제외된다.
- 나머지 사원들은 두 점수의 합이 높은 순으로 석차를 매기며, 완호(0번 사원)의 석차를 구하는 문제이다. 완호가 제외 대상이라면 `-1`을 반환한다.

## 🗝️ 접근 방법

- 사원 수가 최대 10만 명이므로 모든 쌍을 비교하는 `O(N^2)` 방식은 사용할 수 없다.
- 어떤 사원이 인센티브에서 제외되려면, 자신보다 두 점수의 합이 더 큰 사원 중에 자신보다 두 점수 모두 낮은 사원이 있어야 한다.
  - 합이 자신보다 작은 사원에게는 두 점수가 모두 낮을 수 없으므로 비교할 필요가 없다.
- 모든 사원을 점수 합 기준 내림차순으로 정렬한 뒤, 앞에서부터 순회하며 자신보다 앞에 놓인(합이 더 크거나 같은) 사원들만 비교 대상으로 삼는다.
- 제외 대상이 아닌 사원마다 석차를 하나씩 증가시키며, 완호 차례가 되었을 때의 석차를 반환한다.

## 🧩 코드 구현

```java
import java.util.*;

class Solution {

    class Employee {
        int idx;
        int attitude;
        int peer;
        int sum;

        Employee(int idx, int attitude, int peer) {
            this.idx = idx;
            this.attitude = attitude;
            this.peer = peer;
            this.sum = attitude + peer;
        }
    }

    public int solution(int[][] scores) {
        List<Employee> employees = new ArrayList<>();
        for (int i = 0; i < scores.length; ++i) {
            employees.add(new Employee(i, scores[i][0], scores[i][1]));
        }

        employees.sort(Comparator.comparingInt((Employee e) -> e.sum).reversed());

        int rank = 0;
        for (int i = 0; i < employees.size(); ++i) {
            Employee e = employees.get(i);
            boolean excluded = false;

            for (int j = 0; j < i; ++j) {
                if (e.sum == employees.get(j).sum) continue;

                if (e.attitude < employees.get(j).attitude &&
                    e.peer < employees.get(j).peer
                ) {
                    excluded = true;
                    break;
                }
            }
            if (excluded) {
                if (e.idx == 0) return -1;
                continue;
            }
            rank++;
            if (e.idx == 0) break;
        }
        return rank;
    }
}
```
