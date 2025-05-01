---
title: 다익스트라 심화 - 벨만 포드(Bellman-Ford)
date: 2024-05-12 14:47:00 +09:00
categories: [algorithm]
tags:
  [
    algorithm,
    BOJ,
    java,
    dijkstra,
    bellman ford
  ]
---

## 다익스트라의 한계

[다익스트라(dijkstra) 알고리즘](https://ajroot5685.github.io/posts/dijkstra/)

<img src="/assets/img/24/05/12/dijkstra.png" alt="dijkstra" width=500>

- 위 그림에서 다익스트라는 **각 노드에 한번만 방문**하고, 바로 앞에 있는 간선 중에서 **가장 짧은 것**을 선택한다.
- 그 결과 `1->3` 경로의 거리는 음수 가중치를 인식하지 못해 5가 아닌 **10**이라는 결과가 도출된다.
- 이렇게 **다익스트라는 음수 가중치를 가진 간선이 있다면 사용할 수 없다.**

<br>

## 벨만 포드(Bellman-Ford) 알고리즘
- 다익스트라와 마찬가지로 한 노드에서 다른 노드들까지의 거리를 구할 수 있다.
- 간선의 가중치가 음수라도 최단거리를 구할 수 있다.
- 간선들 중에 음수 사이클이 있는 경우를 판별할 수 있다.
- **단, 음수 사이클을 포함한 그래프에서 최단 거리를 구할 수는 없다.**

<br>

## 벨만 포드 로직
1. 출발 노드를 설정한다.
2. 최단 거리 배열을 초기화한다.
3. 다음 과정을 `n-1(n : 정점 개수)` 번 반복한다.
    - 모든 간선을 하나씩 확인한다.
    - 해당 간선으로 다른 노드로 가는 비용을 계산하여 최단 거리 배열을 업데이트한다.
4. 한번 더 수행하여 음수 사이클이 있는지 확인한다.
    - 이때 최단 거리 배열이 업데이트되었다면 음수 사이클이 존재하는 것이다.
    - 특정 노드에 도달하기 위해 거치는 노드 개수의 최대치는 `n-1개` 이기 때문이다.
        > 같은 노드를 2번 지나쳤는데도 최단 거리 갱신이 된다면 음수 사이클을 지나간 것이다.

<br>

## 벨만 포드 예시 - 음수 사이클이 없는 경우
- 노드 개수 : 4개, 출발 노드 : 1번
- 배열 초기화 상태로 시작

<img src="/assets/img/24/05/12/bm1-1.jpg" alt="bm1-1" width=500>
<img src="/assets/img/24/05/12/bm1-2.jpg" alt="bm1-2" width=300>

> 각 반복마다 간선을 확인하는 순서는 동일하다고 가정

<img src="/assets/img/24/05/12/bm1-3.jpg" alt="bm1-3" width=500>
<img src="/assets/img/24/05/12/bm1-4.jpg" alt="bm1-4" width=500>
<img src="/assets/img/24/05/12/bm1-5.jpg" alt="bm1-5" width=500>
<img src="/assets/img/24/05/12/bm1-6.jpg" alt="bm1-6" width=500>

> 예시에서는 노드 개수가 적어 한번만에 최종 최단 거리가 도출되었지만, 노드 개수가 많고 음수 간선이 많으면 계속 바뀔 수 있다.

- `n(=4)` 번째 반복 결과 최단 거리 배열이 바뀌지 않았으므로 음수 사이클은 존재하지 않는다.

<br>

## 벨만 포드 예시 - 음수 사이클이 있는 경우
- 위의 예시에서 `3 -> 4` 간선의 가중치가 1로 줄어드는 경우 음수 사이클이 생긴다.
- 이 경우에는 어떤 결과가 나오는지 알아보자.

<img src="/assets/img/24/05/12/bm2-1.jpg" alt="bm2-1" width=500>
<img src="/assets/img/24/05/12/bm2-2.jpg" alt="bm2-2" width=300>
<img src="/assets/img/24/05/12/bm2-3.jpg" alt="bm2-3" width=500>
<img src="/assets/img/24/05/12/bm2-4.jpg" alt="bm2-4" width=500>
<img src="/assets/img/24/05/12/bm2-5.jpg" alt="bm2-5" width=500>
<img src="/assets/img/24/05/12/bm2-6.jpg" alt="bm2-6" width=500>

- 음수 사이클의 존재로 최단 거리 배열이 계속 수정되었고, 음수 사이클을 확인하는 n번째 반복에서도 값이 변경되어 음수 사이클이 있음을 확인했다.

<br>

## BOJ 11657 - 타임머신 (골드 4)
[11657-타임머신](https://www.acmicpc.net/problem/11657)

- 벨만 포드를 구현하는 문제이다.
- 음수 사이클이 존재한다면 -1 을 출력한다.
- 음수 사이클이 없다면 최단 경로들을 출력하고, 경로가 없다면 -1을 출력한다.

<br>

```java
import java.io.*;
import java.util.*;

public class Main {

    static FastReader scan = new FastReader();
    static StringBuilder sb = new StringBuilder();


    static int n;
    static int m;
    static ArrayList<Node> edge;

    public static void main(String[] args) {
        input();
        solve();
    }

    static void solve() {
        long[] distance = new long[n + 1];
        Arrays.fill(distance, Long.MAX_VALUE);

        distance[1] = 0;

        for (int i = 1; i <= n; i++) {
            for (Node node : edge) {
                if (distance[node.from] == Long.MAX_VALUE) {
                    continue;
                }

                if (distance[node.to] > distance[node.from] + node.cost) {
                    distance[node.to] = distance[node.from] + node.cost;

                    if (i == n) {
                        System.out.println(-1);
                        return;
                    }
                }
            }
        }

        for (int i = 2; i <= n; i++) {
            if (distance[i] == Long.MAX_VALUE) {
                sb.append(-1).append("\n");
            } else {
                sb.append(distance[i]).append("\n");
            }
        }

        System.out.println(sb);
    }

    static void input() {
        n = scan.nextInt();
        m = scan.nextInt();

        edge = new ArrayList<>();

        for (int i = 0; i < m; i++) {
            int from = scan.nextInt();
            int to = scan.nextInt();
            int cost = scan.nextInt();

            edge.add(new Node(from, to, cost));
        }
    }

    static class Node {
        int from;
        int to;
        int cost;

        public Node(int from, int to, int cost) {
            this.from = from;
            this.to = to;
            this.cost = cost;
        }
    }

    static class FastReader {
        .
        .
    }
}
```

- FastReader

    [백준 자바 입출력 템플릿](https://ajroot5685.github.io/posts/%EB%B0%B1%EC%A4%80-%EC%9E%90%EB%B0%94-%EC%9E%85%EC%B6%9C%EB%A0%A5-%ED%85%9C%ED%94%8C%EB%A6%BF/)

<br>

## 벨만 포드를 사용하는 문제 리스트
- [1865 - 웜홀 (골드 3)](https://www.acmicpc.net/problem/1865)
- [1219 - 오민식의 고민 (플레 5)](https://www.acmicpc.net/problem/1219)

> 1219 문제의 경우 도착 도시라는 추가 조건으로 인해 음수 사이클이 있더라도 경로 속에 있는지를 판별해야 한다.
>
> 벨만 포드를 좀 더 깊게 이해할 수 있으므로 꼭 풀어보는 것을 추천한다.

<br>

## 누군가가 물어본다면
<div class="spotlight1" markdown="1">
음수 가중치가 있는 그래프에 적용하지 못하는 `다익스트라`와 달리 `벨만 포드`는 음수 가중치가 있더라도 최단 거리를 구할 수 있고, 음수 사이클의 존재 여부를 알 수 있습니다.
</div>