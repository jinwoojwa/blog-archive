## 🔗 문제 링크

<a class="bookmark-card" href="https://school.programmers.co.kr/learn/courses/30/lessons/60061" target="_blank" rel="noopener noreferrer">
  <div class="bookmark-card-title">기둥과 보 설치</div>
  <div class="bookmark-card-desc">코딩테스트 연습 - 기둥과 보 설치 문제입니다.</div>
  <div class="bookmark-card-url">
    <img class="bookmark-card-favicon" src="https://www.google.com/s2/favicons?domain=programmers.co.kr" alt="" />
    <span>school.programmers.co.kr</span>
  </div>
</a>

## 💡 문제 파악

- `n x n` 크기의 벽면에 기둥과 보를 설치/삭제하는 명령이 `build_frame` 순서대로 주어진다.
- 각 명령은 `[x, y, type, command]` 형태이며, `type` 은 기둥(0)/보(1), `command` 는 삭제(0)/설치(1)를 의미한다.
- 설치/삭제 규칙
  - **기둥**은 바닥 위에 있거나, 아래에 기둥이 있거나, 한쪽 끝이 보와 연결되어 있어야 존재할 수 있다.
  - **보**는 양쪽 끝 중 하나라도 기둥 위에 있거나, 양쪽 끝이 모두 다른 보와 연결되어 있어야 존재할 수 있다.
- 설치는 그 순간 규칙을 만족하지 못하면 무효, 삭제는 삭제 후 **구조물 전체**가 여전히 규칙을 만족해야 유효하다.
- 모든 명령을 처리한 뒤 최종적으로 남은 기둥/보 좌표를 정렬해서 반환해야 한다.

## 🗝️ 접근 방법

- 기둥/보의 존재 여부를 각각 `boolean[][] pillar`, `boolean[][] beam` 2차원 배열로 관리한다.
- **설치(`command == 1`)** 시에는 해당 위치만 우선 `true` 로 설치해보고, 그 좌표 하나에 대해서만 지지 조건을 검사한다.
  - 조건을 만족하지 못하면 방금 설치한 값을 다시 `false` 로 되돌린다. (다른 기둥/보에는 영향이 없으므로 국소적으로만 검사하면 충분하다.)
- **삭제(`command == 0`)** 시에는 해당 위치를 먼저 `false` 로 지워보고, **전체 구조물**이 여전히 유효한지를 검사한다.
  - 하나를 삭제하면 그 위/옆에 의존하던 다른 기둥·보가 무너질 수 있으므로, 전체를 순회하며 재검증해야 한다.
  - 조건을 만족하지 못하면 삭제를 취소하고 다시 `true` 로 되돌린다.
- 이처럼 "설치는 국소 검사, 삭제는 전체 검사"로 나누는 것이 이 문제의 핵심이다.
- 마지막으로 남아있는 기둥/보를 모두 모은 뒤, `x` → `y` → `type`(기둥이 보보다 먼저) 순으로 정렬해서 반환한다.

```java
if (command == 1) build(pillar, beam, x, y, type);
else remove(pillar, beam, x, y, type, n);
```

## 🧩 코드 구현

```java
import java.util.*;

class Solution {
    public int[][] solution(int n, int[][] build_frame) {
        boolean[][] pillar = new boolean[n + 1][n + 1];
        boolean[][] beam = new boolean[n + 1][n + 1];
        
        for (int[] frame : build_frame) {
            int x = frame[0];
            int y = frame[1];
            int type = frame[2];
            int command = frame[3];

            if (command == 1) build(pillar, beam, x, y, type);
            else remove(pillar, beam, x, y, type, n);
        }
        return makeAnswer(pillar, beam, n);
    }
    
    private void build(boolean[][] pillar, boolean[][] beam,
                       int x, int y, int type
    ) {
        if (type == 0) { // 기둥
            pillar[x][y] = true;

            if (!isPossiblePillar(pillar, beam, x, y)) pillar[x][y] = false;
        }
        else { // 보
            beam[x][y] = true;

            if (!isPossibleBeam(pillar, beam, x, y)) beam[x][y] = false;
        }
    }
    
    private void remove(boolean[][] pillar, boolean[][] beam,
                       int x, int y, int type, int n
    ) {
        if (type == 0) {
            pillar[x][y] = false;

            if (!isPossible(pillar, beam, n)) pillar[x][y] = true;
        }
        else {
            beam[x][y] = false;

            if (!isPossible(pillar, beam, n)) beam[x][y] = true;
        }
    }
    
    private boolean isPossiblePillar(boolean[][] pillar, boolean[][] beam, int x, int y) {
        // 바닥 위에 있는 경우
        if (y == 0) return true;

        // 아래에 기둥이 있는 경우
        if (pillar[x][y - 1]) return true;

        // 왼쪽 끝에 보가 있는 경우
        if (x > 0 && beam[x - 1][y]) return true;

        // 오른쪽 끝에 보가 있는 경우
        if (beam[x][y]) return true;

        return false;
    }
    
    private boolean isPossibleBeam(boolean[][] pillar, boolean[][] beam, int x, int y) {
        // 왼쪽 끝에 기둥이 있는 경우
        if (pillar[x][y - 1]) return true;

        // 오른쪽 끝에 기둥이 있는 경우
        if (pillar[x + 1][y - 1]) return true;

        // 왼쪽과 오른쪽에 보가 연결되어 있는 경우
        if (x > 0 && beam[x - 1][y] && beam[x + 1][y]) return true;

        return false;
    }
    
    private boolean isPossible(boolean[][] pillar, boolean[][] beam, int n) {
        for (int x = 0; x <= n; ++x) {
            for (int y = 0; y <= n; ++y) {
                if (pillar[x][y] && !isPossiblePillar(pillar, beam, x, y)) return false;
                if (beam[x][y] && !isPossibleBeam(pillar, beam, x, y)) return false;
            }
        }
        return true;
    }
    
    private int[][] makeAnswer(boolean[][] pillar, boolean[][] beam, int n) {
        List<int[]> result = new ArrayList<>();

        for (int x = 0; x <= n; ++x) {
            for (int y = 0; y <= n; ++y) {
                if (pillar[x][y]) result.add(new int[]{x, y, 0});
                if (beam[x][y]) result.add(new int[]{x, y, 1});
            }
        }

        // x 오름차순 → y 오름차순 → 기둥(0) → 보(1)
        result.sort((a, b) -> {
            if (a[0] != b[0]) return Integer.compare(a[0], b[0]);

            if (a[1] != b[1]) return Integer.compare(a[1], b[1]);

            return Integer.compare(a[2], b[2]);
        });

        return result.toArray(new int[result.size()][]);
    }
}
```
