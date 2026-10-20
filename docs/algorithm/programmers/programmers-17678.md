## 🔗 문제 링크

<a class="bookmark-card" href="https://school.programmers.co.kr/learn/courses/30/lessons/17678" target="_blank" rel="noopener noreferrer">
  <div class="bookmark-card-title">[1차] 셔틀버스</div>
  <div class="bookmark-card-desc">코딩테스트 연습 - [1차] 셔틀버스 문제입니다.</div>
  <div class="bookmark-card-url">
    <img class="bookmark-card-favicon" src="https://www.google.com/s2/favicons?domain=programmers.co.kr" alt="" />
    <span>school.programmers.co.kr</span>
  </div>
</a>

## 💡 문제 파악

- 셔틀버스는 `09:00` 부터 `t` 분 간격으로 총 `n` 회 운행하고, 한 번에 최대 `m` 명까지 태울 수 있다.
- `timetable` 은 크루들이 대기열에 도착하는 시각 목록이다. 버스가 도착했을 때 이미 도착해 있는 크루만 탈 수 있고, 먼저 온 사람이 먼저 탄다.
- 같은 시각에 도착한 크루들 중에서는 콘(나)이 가장 뒤에 선다.
- 셔틀을 타고 사무실로 갈 수 있는 도착 시각 중 **가장 늦은 시각**을 `HH:MM` 형식으로 반환한다.

## 🗝️ 접근 방법

- `timetable` 을 오름차순으로 정렬한 뒤, 버스를 순서대로 시뮬레이션한다. (`"HH:MM"` 형식은 같은 자릿수라 문자열 정렬이 곧 시간순 정렬이다.)
  - `i` 번째 버스의 출발 시간은 `540 + i * t` 분이다. (`09:00` = 540분)
  - 버스 출발 시간 이하로 도착한 크루를 앞에서부터 최대 `m` 명까지 태운다. `idx` 는 다음에 탈 크루를 가리키는 포인터다.
- 콘이 탈 수 있는 가장 늦은 시각은 **마지막 버스**에서만 결정하면 된다.
  - 마지막 버스에 **자리가 남아 있으면**: 버스 출발 시간에 맞춰 도착해도 탈 수 있다. (같은 시각 도착이면 콘이 가장 뒤라서 자리가 남을 때만 가능)
  - 마지막 버스가 **만석이면**: 마지막으로 탄 크루보다 먼저 서야 하므로 그 크루의 도착 시각보다 **1분 일찍** 도착해야 한다.
- 시각은 모두 분 단위 정수로 바꿔서 비교하고, 마지막에 `%02d:%02d` 로 다시 문자열로 변환한다.

```java
if (i == n - 1) { // 마지막 버스
    if (cnt < m) {
        answer = busTime;                          // 자리가 남으면 버스 시간에 도착
    } else {
        answer = toTime(timetable[idx - 1]) - 1;   // 만석이면 마지막 탑승자보다 1분 일찍
    }
}
```

## 🧩 코드 구현

```java
import java.util.*;

class Solution {
    public String solution(int n, int t, int m, String[] timetable) {
        Arrays.sort(timetable);

        int idx = 0;
        int answer = 0;

        for (int i = 0; i < n; ++i) {
            int busTime = 540 + i * t; // 이번 버스 출발 시간 (분)

            int cnt = 0; // 이번 버스에 탄 크루 수

            // 버스 출발 시간까지 도착한 크루를 최대 m명 태움
            while (idx < timetable.length
                    && toTime(timetable[idx]) <= busTime
                    && cnt < m) {
                idx++;
                cnt++;
            }

            if (i == n - 1) { // 마지막 버스
                if (cnt < m) {
                    // 자리가 남으면 -> 버스 출발 시간에 도착해도 됨
                    answer = busTime;
                } else {
                    // 만석이면 -> 마지막 탑승자보다 1분 일찍 도착해야 함
                    answer = toTime(timetable[idx - 1]) - 1;
                }
            }
        }

        return String.format("%02d:%02d", answer / 60, answer % 60);
    }

    private int toTime(String time) {
        String[] t = time.split(":");

        return Integer.parseInt(t[0]) * 60 + Integer.parseInt(t[1]);
    }
}
```
