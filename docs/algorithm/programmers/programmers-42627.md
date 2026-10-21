## 🔗 문제 링크

<a class="bookmark-card" href="https://school.programmers.co.kr/learn/courses/30/lessons/42627" target="_blank" rel="noopener noreferrer">
  <div class="bookmark-card-title">디스크 컨트롤러</div>
  <div class="bookmark-card-desc">코딩테스트 연습 - 디스크 컨트롤러 문제입니다.</div>
  <div class="bookmark-card-url">
    <img class="bookmark-card-favicon" src="https://www.google.com/s2/favicons?domain=programmers.co.kr" alt="" />
    <span>school.programmers.co.kr</span>
  </div>
</a>

## 💡 문제 파악

- `jobs[i] = [요청 시각, 소요 시간]` 이고, 디스크는 한 번에 하나의 작업만 처리하며 처리 중에는 중단할 수 없다.
- 디스크가 놀고 있을 때는 대기 큐에 있는 작업 중 **소요 시간 → 요청 시각 → 작업 번호** 순으로 우선순위가 높은 작업을 먼저 처리한다.
- 각 작업의 반환 시간은 `종료 시각 - 요청 시각` 이고, 이 값들의 **평균(소수점 이하 버림)** 을 최소로 만드는 스케줄링 결과를 구한다. 즉, 위 우선순위 규칙대로 처리했을 때의 평균을 구하면 된다.

## 🗝️ 접근 방법

- 요청 시각이 빠른 작업이 있어도 다른 작업보다 소요 시간이 길면 최우선이 아니다. 그래서 **요청 시각 기준 정렬 리스트**와 **우선순위 큐**를 분리해서 사용한다.
  - `jobList`: 요청 시각 → 작업 번호 순으로 정렬한 전체 작업 목록. `idx` 로 아직 큐에 넣지 않은 작업의 위치를 가리킨다.
  - `pq`: 현재 시각까지 요청이 들어온 작업들을 소요 시간 → 요청 시각 → 작업 번호 순으로 꺼내는 우선순위 큐.
- 현재 시각 `time` 을 기준으로 다음을 반복한다.
  - 요청 시각이 `time` 이하인 작업을 모두 `pq` 에 넣는다.
  - `pq` 가 비어 있지 않으면 하나 꺼내서 처리한다. `time += duration` 으로 갱신하고, 반환 시간 `time - startTime` 을 `total` 에 더한다.
  - `pq` 가 비어 있으면 디스크가 놀고 있는 상태이므로, 다음 작업의 요청 시각으로 `time` 을 건너뛴다.
- 모든 작업을 처리하면 `total / size` 를 반환한다. 정수 나눗셈이라 소수점 이하가 자동으로 버려진다.

```java
while (idx < size || !pq.isEmpty()) {
    while (idx < size && jobList.get(idx).startTime <= time) {
        pq.offer(jobList.get(idx++));
    }

    if (!pq.isEmpty()) {
        Job job = pq.poll();
        time += job.duration;
        total += time - job.startTime;
    } else {
        time = jobList.get(idx).startTime;
    }
}
```

## 🧩 코드 구현

```java
import java.util.*;

class Solution {

    class Job {
        int num;
        int startTime;
        int duration;

        public Job(int num, int startTime, int duration) {
            this.num = num;
            this.startTime = startTime;
            this.duration = duration;
        }
    }

    public int solution(int[][] jobs) {
        List<Job> jobList = new ArrayList<>();
        for (int i = 0; i < jobs.length; ++i) {
            jobList.add(new Job(i, jobs[i][0], jobs[i][1]));
        }

        Collections.sort(jobList, Comparator.comparingInt((Job j) -> j.startTime)
                        .thenComparing(j -> j.num));

        // 소요시간 -> 요청 시각 -> 작업 번호
        PriorityQueue<Job> pq = new PriorityQueue<>(
            Comparator.comparingInt((Job j) -> j.duration)
                      .thenComparing(j -> j.startTime)
                      .thenComparing(j -> j.num)
        );

        int time = 0, total = 0, idx = 0, size = jobs.length;

        while (idx < size || !pq.isEmpty()) {
            while (idx < size && jobList.get(idx).startTime <= time) {
                pq.offer(jobList.get(idx++));
            }

            if (!pq.isEmpty()) {
                Job job = pq.poll();
                time += job.duration;
                total += time - job.startTime;
            } else {
                time = jobList.get(idx).startTime;
            }
        }
        return total / size;
    }
}
```
