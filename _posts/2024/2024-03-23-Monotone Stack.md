---
title: Monotone Stack
date: 2024-03-23 15:39:00 +09:00
categories: [algorithm]
tags:
  [
    algorithm,
    BOJ,
    monotone stack
  ]
---

## Monotone Stack 이란?
- 일련의 값에서 바로 다음 큰값이나, 바로 다음 작은 값을 구해야할 때 사용할 수 있는 Stack 풀이법이다.
- 주로 배열의 요소들을 순회하면서, 각 요소의 다음에 오는 더 큰 요소나 더 작은 요소를 찾는 문제에 사용된다.
- 즉, 어떤 특정 요소를 기점으로 `처음으로` 큰 요소를 찾거나 작은 요소를 찾는 문제에 적용할 수 있다.

<br>

#### increasing
- 삽입하려는 값이 stack의 top 보다 클 때만 push
- 삽입하려는 값보다 작은 값은 모두 stack에서 pop
- 배열에서 다음 작은 값이 무엇인지 알아낼 수 있다.

<br>

#### decreasing
- 삽입하려는 값이 stack의 top 보다 작을 때만 push
- 삽입하려는 값보다 큰 값은 모두 stack에서 pop
- 배열에서 다음 큰 값이 무엇인지 알아낼 수 있다.

<br>

## BOJ 17298 - 오큰수 (골드 4)
[17209-오큰수](https://www.acmicpc.net/problem/17298)

- 주어지는 수열의 각 원소에 대해 자신보다 큰 오른쪽에 있는 수들 중 가장 왼쪽에 있는 수를 `오큰수`라고 한다.
- 각 원소에 대해 오큰수를 구해야 한다.
- 오큰수가 없는 경우 -1 로 표시한다.
- ex. 크기가 4인 수열 3, 5, 2, 7 -> 정답 : 5, 7, 7, -1

<br>

## 풀이
- monotone stack을 이용하여 수열의 왼쪽부터 차례로 스택에 넣는다.
- 이때 스택에 넣는 원소 A보다 top에 있는 원소 B가 작다면 B를 꺼내고 해당 원소의 오큰수를 A로 기록한다.
    - 스택의 top에 A보다 크거나 비어있을 때까지 반복한다.
- top에 있는 원소가 더 크다면 그냥 넣는다.
- 결과도 수열 순서대로 출력해야 하기 때문에 정수 배열에 기록을 하고 stack의 원소는 인덱스로, 값비교는 인덱스에 해당하는 배열의 원소값으로 설계한다.

```java
import java.io.*;
import java.util.Deque;
import java.util.LinkedHashMap;
import java.util.LinkedList;
import java.util.StringTokenizer;

public class Main {
    static FastReader scan = new FastReader();
    static StringBuilder sb = new StringBuilder();

    static int n;
    static Deque<Integer> deque = new LinkedList<>();
    static int[] result;

    public static void main(String[] args) {
        input();

        for(int i=0;i<n;i++){
            if (sb.length()>0){
                sb.append(" ");
            }
            sb.append(result[i]);
        }

        System.out.println(sb.toString());
    }
    static void input(){
        n = Integer.parseInt(scan.next());
        result = new int[n];

        for (int i=0;i<n;i++){
            result[i]=Integer.parseInt(scan.next());
        }

        solve();
    }
    static void solve(){
        for (int i=0;i<n;i++){
            while(!deque.isEmpty() && result[deque.peekFirst()]<result[i]){
                result[deque.pollFirst()]=result[i];
            }
            deque.offerFirst(i);
        }
        for (int i=0;i<deque.size();){
            result[deque.pollFirst()]=-1;
        }
    }
    static class FastReader {
        // 생략
    }
}
```

- FastReader 

    [백준 자바 입출력 템플릿](https://ajroot5685.github.io/posts/%EB%B0%B1%EC%A4%80-%EC%9E%90%EB%B0%94-%EC%9E%85%EC%B6%9C%EB%A0%A5-%ED%85%9C%ED%94%8C%EB%A6%BF/)

<br>

## Monotone Stack을 이용한 문제 리스트
- [6198-옥상 정원 꾸미기 (골드 5)](https://www.acmicpc.net/problem/6198)
- [2493-탑 (골드 5)](https://www.acmicpc.net/problem/2493)
- [16288-Passport Control (골드 3)](https://www.acmicpc.net/problem/16288)
- [14719-빗물 (골드 5)](https://www.acmicpc.net/problem/14719)
- [1725-히스토그램 (플레 5)](https://www.acmicpc.net/problem/1725)

<br>

## 누군가가 물어본다면
<div class="spotlight1">
monotone stack은 일련의 값에서 각 요소의 다음에 <code class="language-plaintext highlighter-rouge">처음으로</code> 크거나 작은 요소를 찾는 문제에 적합합니다.
</div>