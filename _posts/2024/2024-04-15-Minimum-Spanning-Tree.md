---
title: 최소 스패닝 트리를 푸는 2가지 알고리즘 - 크루스칼, 프림
date: 2024-04-15 12:26:00 +09:00
categories: [algorithm]
tags:
  [
    algorithm,
    최소 스패닝 트리,
    크루스칼,
    프림
  ]
---

## 🤔 최소 스패닝 트리(Minimum Spanning Tree)란?
- 가능한 모든 Spanning Tree 중에서 가장 작은 가중치를 갖는 트리를 의미한다.
- 크루스칼(Kruskal)과 프림(Prim) 알고리즘이 MST를 찾는 대표적인 알고리즘이다.

<div class="spotlight2">
Spnning Tree란 주어진 그래프의 <b>모든 정점을 포함</b>하면서 <b>사이클이 없는</b> 부분 그래프를 말한다.
<br><br>
최소한의 간선으로 연결하므로, 어떤 두 정점 사이에는 정확히 하나의 경로만 존재한다.
</div>

<br>

## 😎 크루스칼 알고리즘
1. 모든 edge들을 가중치 기준으로 오름차순 정렬한다.
2. 이후 모든 edge들을 앞에서부터 순서대로 탐색하여 지금까지의 MST에 포함시킨다.
    - 이때 사이클을 형성하지 않는 edge만 MST에 포함한다.
    - 포함된 edge의 가중치를 결과값에 더하여 기록한다.

> edge가 `전체 노드 개수 - 1` 개가 되면 MST가 만들어진 것이므로 탐색을 종료시킴으로써 약간의 성능을 향상시킬 수 있다.

- 시간 복잡도 : `O(ElogE)` (E : edge 개수)
    - 크루스칼의 시간 복잡도는 edge를 정렬하는데 드는 시간에 의해 결정된다.
    - 이 시간 복잡도는 최소힙 자료구조를 사용했을 때의 시간복잡도이다.
    - 프림보다 상대적으로 밀도가 낮은, 즉 node 개수는 많지만 edge 개수는 적은 그래프에서 효율적이다.

- 사이클 형성 여부를 쉽게 파악하기 위해서 `Union-Find 알고리즘` 을 사용한다.
    > [Union-Find Algorithm (유니온-파인드)](https://ajroot5685.github.io/posts/%EC%9C%A0%EB%8B%88%EC%98%A8-%ED%8C%8C%EC%9D%B8%EB%93%9C-%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98/)

<br>

## 🥸 프림 알고리즘
1. 모든 edge가 들어있는 집합을 만든다.
2. 임의의 정점에서 시작한다. (초기 트리 크기는 1)
3. 트리 안에 속하지 않고 트리와 연결된 edge들 중 가장 작은 가중치를 가지는 edge를 추가한다.

- edge가 `전체 노드 개수 - 1` 개가 될 때까지 진행한다.(모든 노드를 방문할 때까지 진행)

- 시간 복잡도 : `O(ElogV)` (E : edge 개수, V : node 개수)
    - 이 시간 복잡도는 최소힙 자료구조를 사용했을 때의 시간복잡도이다.
    - 크루스칼보다 상대적으로 밀도가 높은, 즉 node 개수는 적고 node 당 edge 개수가 많은 그래프에서 효율적이다.

- 다익스트라와 목적은 다르지만 유사한 알고리즘을 가지고 있다.
    > 실제로 프림 알고리즘은 다익스트라와 프림이 함께 발견하여 `Prim-Dijkstra` 알고리즘으로도 불린다.

<br>


## BOJ 1197 - 최소 스패닝 트리 (골드 4)
[1197-최소 스패닝 트리](https://www.acmicpc.net/problem/1197)

- MST 그 자체를 구하는 문제이다.

<br>

#### [크루스칼 풀이]

- Node 클래스를 만들고, 크루스칼을 사용하기 위한 union, find 알고리즘을 구현했다.
- 기본적인 union 함수를 변형하여 사이클인지의 여부를 반환하도록 설계했다.
- 간선 정보는 ArrayList를 사용해도 되지만 더 익숙한 우선순위 큐를 사용하였다.

```java
import java.io.*;
import java.util.*;

public class Main {
    static FastReader scan = new FastReader();
    static StringBuilder sb = new StringBuilder();

    static int v;
    static int e;

    static int[] parent;
    static int result;
    static PriorityQueue<Node> queue = new PriorityQueue<>();

    public static void main(String[] args) {
        input();
        solve();
    }

    static void input() {
        v = scan.nextInt();
        e = scan.nextInt();

        parent = new int[v + 1];

        for (int i = 1; i < v + 1; i++) {
            parent[i]=i;
        }

        for (int i=0;i<e;i++){
            queue.add(new Node(scan.nextInt(), scan.nextInt(), scan.nextInt()));
        }
    }

    static void solve() {
        result = 0;

        while (!queue.isEmpty()) {
            Node n = queue.poll();

            if (union(n.from, n.to)) {
                continue;
            }
            result += n.weight;
        }
        System.out.println(result);
    }

    static int find(int x) {
        if (parent[x] == x){
            return x;
        }
        return parent[x] = find(parent[x]);
    }

    static boolean union(int x, int y) {
        int rootX = find(x);
        int rootY = find(y);

        if (rootX == rootY) {
            return true;
        }

        if (rootX > rootY) {
            parent[rootY] = rootX;
        } else {
            parent[rootX] = rootY;
        }
        return false;
    }

    static class Node implements Comparable<Node>{
        int from;
        int to;
        int weight;

        public Node(int from, int to, int w) {
            this.from = from;
            this.to = to;
            this.weight = w;
        }

        @Override
        public int compareTo(Node n) {
            return this.weight - n.weight;
        }
    }

    static class FastReader{
        .
        .
        .
    }
}
```

- FastReader

    [백준 자바 입출력 템플릿](https://ajroot5685.github.io/posts/%EB%B0%B1%EC%A4%80-%EC%9E%90%EB%B0%94-%EC%9E%85%EC%B6%9C%EB%A0%A5-%ED%85%9C%ED%94%8C%EB%A6%BF/)

<br>

#### [프림 풀이]

- 크루스칼과의 차이점을 꼽자면 다음과 같다.
    - Node 클래스의 멤버 변수 축소
    - 무방향 그래프이므로 edge 저장 리스트에 양방향으로 저장(단방향 2개)
    - visit 배열로 방문한 노드 기록

```java
import java.io.*;
import java.util.*;

public class Main {
    static FastReader scan = new FastReader();
    static StringBuilder sb = new StringBuilder();

    static int v;
    static int e;

    static ArrayList<Node>[] edge;
    static boolean[] visit;

    public static void main(String[] args) {
        input();
        solve();
    }

    static void input() {
        v = scan.nextInt();
        e = scan.nextInt();

        edge = new ArrayList[v + 1];

        for (int i = 1; i <= v; i++) {
            edge[i] = new ArrayList<>();
        }

        visit = new boolean[v + 1];

        for (int i = 0; i < e; i++) {
            int from = scan.nextInt();
            int to = scan.nextInt();
            int weight = scan.nextInt();

            edge[from].add(new Node(to, weight));
            edge[to].add(new Node(from, weight));
        }
    }

    static void solve() {
        long result = 0;

        PriorityQueue<Node> queue = new PriorityQueue<>();

        queue.add(new Node(1, 0));

        while (!queue.isEmpty()) {
            Node node = queue.poll();

            if (visit[node.to]) {
                continue;
            }
            visit[node.to] = true;

            result += node.weight;

            for (Node next : edge[node.to]) {
                if (!visit[next.to]) {
                    queue.add(next);
                }
            }
        }

        System.out.println(result);
    }

    static class Node implements Comparable<Node>{
        int to;
        int weight;

        public Node(int to, int w) {
            this.to = to;
            this.weight = w;
        }

        @Override
        public int compareTo(Node n) {
            return this.weight - n.weight;
        }
    }

    static class FastReader{
        .
        .
        .
    }
}
```

- FastReader

    [백준 자바 입출력 템플릿](https://ajroot5685.github.io/posts/%EB%B0%B1%EC%A4%80-%EC%9E%90%EB%B0%94-%EC%9E%85%EC%B6%9C%EB%A0%A5-%ED%85%9C%ED%94%8C%EB%A6%BF/)

<br>

## MST를 사용하는 문제 리스트
- [1922 - 네트워크 연결 (골드 4)](https://www.acmicpc.net/problem/1922)
- [4386 - 별자리 만들기 (골드 3)](https://www.acmicpc.net/problem/4386)
- [17472 - 다리 만들기 2 (골드 1)](https://www.acmicpc.net/problem/17472)
- [2887 - 행성 터널 (플레 5)](https://www.acmicpc.net/problem/2887)

<br>

## 누군가가 물어본다면
<div class="spotlight1">
<code class="language-plaintext highlighter-rouge">Minimum Spanning Tree</code>를 구하기 위해서는 대표적으로 크루스칼과 프림 2가지 알고리즘을 사용할 수 있습니다.
<br><br>
크루스칼은 <code class="language-plaintext highlighter-rouge">Union-Find</code> 알고리즘을 사용하고, 프림은 <code class="language-plaintext highlighter-rouge">Dijkstra</code> 와 유사한 알고리즘을 사용합니다.
</div>