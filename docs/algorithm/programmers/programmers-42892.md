## 🔗 문제 링크

<a class="bookmark-card" href="https://school.programmers.co.kr/learn/courses/30/lessons/42892" target="_blank" rel="noopener noreferrer">
  <div class="bookmark-card-title">길 찾기 게임</div>
  <div class="bookmark-card-desc">코딩테스트 연습 - 길 찾기 게임 문제입니다.</div>
  <div class="bookmark-card-url">
    <img class="bookmark-card-favicon" src="https://www.google.com/s2/favicons?domain=programmers.co.kr" alt="" />
    <span>school.programmers.co.kr</span>
  </div>
</a>

## 💡 문제 파악

- 2차원 좌표로 표현된 노드 정보(`nodeinfo`)가 주어지며, 이 좌표들로 이진 트리를 구성한다.
- 구성한 이진 트리를 전위 순회, 후위 순회한 결과를 각각 배열로 반환하는 문제이다.

## 🗝️ 접근 방법

- 좌표들을 `y좌표 내림차순`으로, y좌표가 같다면 `x좌표 오름차순`으로 정렬하면 루트 노드부터 왼쪽 자식, 오른쪽 자식 순서로 정렬된다.
- 정렬된 첫 번째 노드를 루트로 삼고, 이후 노드들을 하나씩 트리에 삽입한다.
  - 삽입할 노드의 x좌표가 부모보다 작으면 왼쪽 서브트리로, 크면 오른쪽 서브트리로 재귀적으로 내려가며 위치를 찾는다.
- 트리가 모두 구성되면 전위 순회와 후위 순회를 각각 수행해서 결과를 반환한다.

## 🧩 코드 구현

```java
import java.util.*;

class Solution {

    class Node {
        int x;
        int y;
        int num;
        Node left;
        Node right;

        Node(int x, int y, int num) {
            this.x = x;
            this.y = y;
            this.num = num;
        }
    }

    public int[][] solution(int[][] nodeinfo) {
        List<Node> nodes = new ArrayList<>();
        for (int i = 0; i < nodeinfo.length; i++) {
            nodes.add(new Node(nodeinfo[i][0], nodeinfo[i][1], i + 1));
        }

        nodes.sort(Comparator.comparingInt((Node n) -> n.y).reversed()
                      .thenComparingInt(n -> n.x)
        );

        Node root = nodes.get(0);

        for (int i = 1; i < nodes.size(); i++) {
            insert(root, nodes.get(i));
        }

        List<Integer> preorder = new ArrayList<>();
        List<Integer> postorder = new ArrayList<>();

        preorder(root, preorder);
        postorder(root, postorder);

        return new int[][] {
            preorder.stream().mapToInt(Integer::intValue).toArray(),
            postorder.stream().mapToInt(Integer::intValue).toArray()
        };
    }

    private void insert(Node parent, Node child) {
        if (child.x < parent.x) {
            if (parent.left == null) {
                parent.left = child;
            } else {
                insert(parent.left, child);
            }
        } else {
            if (parent.right == null) {
                parent.right = child;
            } else {
                insert(parent.right, child);
            }
        }
    }

    private void preorder(Node node, List<Integer> result) {
        if (node == null) return;

        result.add(node.num);
        preorder(node.left, result);
        preorder(node.right, result);
    }

    private void postorder(Node node, List<Integer> result) {
        if (node == null) return;

        postorder(node.left, result);
        postorder(node.right, result);
        result.add(node.num);
    }
}
```
