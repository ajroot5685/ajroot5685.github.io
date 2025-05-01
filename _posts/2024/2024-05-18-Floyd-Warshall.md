---
title: 다익스트라 심화 - 플로이드 워셜(Floyd-Warshall)
date: 2024-05-18 17:48:00 +09:00
categories: [algorithm]
tags:
  [
    algorithm,
    BOJ,
    java,
    dijkstra,
    floyd warshall
  ]
---

## 다익스트라의 한계

[다익스트라(Dijkstra) 알고리즘](https://ajroot5685.github.io/posts/dijkstra/)
[벨만 포드(Bellman-Ford) 알고리즘](https://ajroot5685.github.io/posts/Bellman-Ford/)

> 이전 포스팅에서 다익스트라는 음수 가중치를 고려할 수 없다는 한계를 극복하기 위해 벨만 포드 알고리즘을 다뤘었다.
>
> 이외에 또다른 한계를 알아보자.

- 다익스트라는 하나의 점에서 모든 지점까지의 최단 경로를 구하는 알고리즘이다.
- 모든 지점에서 다른 모든 지점까지의 최단 경로를 구하려면 어떻게 해야 할까?

<br>

#### [다익스트라 n번]
- 모든 지점에서 다익스트라를 사용하여 구할 수 있다.
- 그러나 이 경우, 음수 가중치가 고려되지 않을 것이다.

<br>

#### [벨만포드 n번]
- 모든 지점에서 벨만 포드를 사용하여 구할 수 있다.
- 이 경우, 간선 수가 희소한 경우에는 더 효과적일 수 있다.
- 하지만 구현 관점에서 플로이드 워셜이 더 간단하다.

<br>

#### [알고리즘 비교]

|| Dijkstra | Bellman-Ford | Floyd-Warshall |
|:--:|:--:|:--:|:--:|
| **시간 복잡도** | O(V^2) (Array)<br>O(Elog(V)) (Heap) | O(V * E) | O(V^3) |
| **공간 복잡도** | O(V) | O(V) | O(V^2) |
| **특징** | 하나의 노드에서부터<br>모든 노드의 최단 경로를 구함<br><br>음수 가중치를 고려하지 못함 | 하나의 노드에서부터<br>모든 노드의 최단 경로를 구함<br><br>음수 가중치를 고려할 수 있음 | 모든 노드에서부터<br>모든 노드의 최단 경로를 구함<br><br>음수 가중치를 고려할 수 있음 |

<br>

## 플로이드 워셜(Floyd-Warshall) 알고리즘
- 플로이드 워셜은 모든 지점에서 모든 지점까지의 최단 경로를 구하면서 음수 가중치도 고려할 수 있다.
- 구현이 간단하다.

<br>

## 플로이드 워셜 로직
1. 그래프의 인접 행렬(최단 거리 행렬)을 초기화한다.
    - i에서 j로의 직접적인 간선이 존재하면 해당 가중치로 초기화한다.
    - 자기 자신으로의 거리는 0으로 초기화한다.
    - 이외에는 무한대로 초기화한다.
        > 자바에서는 적당히 10억 정도로 초기화한다. 그래야 MAX와 계산했을 때 오버플로우가 발생하지 않기 때문이다.
2. `삼중 for문`을 수행한다. (`i -> j -> k`)
    - 가장 바깥쪽 for문 i는 경유지를 의미한다.
    - j에서 k로 가는 경로를 **기존 경로**와 **j -> i + i -> k 경로**를 비교하여 더 작은 값으로 업데이트 한다.
3. 음수 사이클을 확인하고 싶다면, 대각선 원소인 `dis[i][i]` 를 확인한다.
    - 음수라면, 음수 사이클이 존재한다.

<br>

## 플로이드 워셜 로직
> 아무래도 시간복잡도가 `O(V^3)` 인 만큼 경우의 수가 너무 많다보니 자세히 다루지는 않겠다.

- 아래 gif는 A - B - C - D - E 순서대로 경유지를 설정하여 그래프가 업데이트 되는 모습을 보여준다.

    ![floyd](https://blog.kakaocdn.net/dn/uC9In/btrX4iVZxhp/OSubywGuVPhCaZZMPJ0Ypk/img.gif)

> 출처
>
> [[알고리즘] Floyd-Warshall Algorithm : 플로이드 워셜 알고리즘이란?](https://olrlobt.tistory.com/43)

<br>

## BOJ 1613 - 역사 (골드 3)
[1613-역사](https://www.acmicpc.net/problem/1613)

- 플로이드 워셜을 사용한다.
- 최단 거리를 구하는 것이 아닌 사건들의 전후 관계를 나타나기 위해 해당 경로가 있다는 것만 나타내면 된다.

<br>

```java
import java.io.*;
import java.util.*;

public class Main {

    static FastReader scan = new FastReader();
    static StringBuilder sb = new StringBuilder();

    static int n;
    static int k;

    static int[][] table;

    public static void main(String[] args) {
        input();
    }

    static void input() {
        n = scan.nextInt();
        k = scan.nextInt();

        table = new int[n + 1][n + 1];

        for (int i = 0; i < k; i++) {
            int from = scan.nextInt();
            int to = scan.nextInt();

            table[from][to] = 1;
        }

        solve();

        int s = scan.nextInt();
        for (int i = 0; i < s; i++) {
            int front = scan.nextInt();
            int back = scan.nextInt();

            if (table[front][back] == 1) {
                sb.append(-1).append("\n");
            } else if (table[back][front] == 1) {
                sb.append(1).append("\n");
            } else {
                sb.append(0).append("\n");
            }
        }

        System.out.println(sb);
    }

    static void solve() {
        for (int i = 1; i <= n; i++) {
            for (int j = 1; j <= n; j++) {
                for (int k = 1; k <= n; k++) {
                    if (table[j][k] == 0 && table[j][i] == 1 && table[i][k] == 1) {
                        table[j][k] = 1;
                    }
                }
            }
        }
    }

    static class FastReader {
        .
        .
        .
    }
}
```

- FastReader

    [백준 자바 입출력 템플릿](https://ajroot5685.github.io/posts/%EB%B0%B1%EC%A4%80-%EC%9E%90%EB%B0%94-%EC%9E%85%EC%B6%9C%EB%A0%A5-%ED%85%9C%ED%94%8C%EB%A6%BF/)

<br>

## 플로이드 워셜을 사용하는 문제 리스트
- [11404 - 플로이드 (골드 4)](https://www.acmicpc.net/problem/11404)
- [11403 - 경로 찾기 (실버 1)](https://www.acmicpc.net/problem/11403)
- [1389 - 케빈 베이컨의 6단계 법칙 (실버 1)](https://www.acmicpc.net/problem/1389)
- [2610 - 회의준비 (골드 2)](https://www.acmicpc.net/problem/2610)

> 플로이드 워셜을 풀다보면 특징이 보인다.
>
> 시간 복잡도가 세제곱이나 되기 때문에 정점의 개수가 1000개 이하로 주어진다.
>
> 이 점을 기억하면 코테에서도 쉽게 유형을 파악할 수 있을 것이다.

<br>

## 누군가가 물어본다면
<div class="spotlight1" markdown="1">
`플로이드 워셜`은 모든 정점에서 모든 정점의 최단 경로를 구하는 알고리즘입니다.
<br>
벨만 포드와 마찬가지로 음수 가중치를 고려할 수 있고, 음수 사이클 존재를 판별할 수 있습니다.
<br>
또한 **구현이 간단**하다는 장점이 있습니다.
</div>