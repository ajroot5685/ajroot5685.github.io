---
title: 우선순위 큐(Priority Queue) 알아보기
date: 2024-04-17 10:30:00 +09:00
categories: [data structure]
tags:
  [
    data structure,
    priority queue,
    우선순위 큐
  ]
---

## 🤔 우선순위 큐란?
- 먼저 들어오는 데이터가 먼저 나가는 `FIFO`인 Queue와 달리, 우선순위 큐는 우선순위가 높은 데이터가 먼저 나가는 자료구조이다.
- 큐의 예시가 은행업무를 기다리는 대기줄이라고 하면, 우선순위 큐의 예시는 병원의 응급 환자를 생각할 수 있다.

|자료 구조|먼저 나가는 요소|
|:--:|:--:|
|스택(Stack)|가장 최근에 들어온 데이터(LIFO)|
|큐(Queue)|가장 먼저 들어온 데이터(FIFO)|
|우선순위 큐(Priority Queue)|가장 우선순위 가 높은 데이터|

<br>

## 😎 우선순위 큐의 구현
- 여러 방법으로 구현을 할 수 있지만 힙(Heap) 자료구조를 이용하는 것이 가장 효율적이다.
- 오름차순이면 최소힙, 내림차순이면 최대힙을 사용한다.
    ![최대힙과 최소힙](/assets/img/24/04/14/최대%20힙,%20최소%20힙.png)

- [힙(heap) 자료구조 알아보기](https://ajroot5685.github.io/posts/Heap/)

<br>

## 😉 우선순위 큐 사용(JAVA)
- 자바에서는 내부적으로 우선순위 큐가 구현되어 있고 기본적으로 `최소힙`을 사용한다.
- 큐와 동일하게 `add()`, `peek()`, `poll()` 등의 메서드를 사용할 수 있다.

#### [Integer 오름차순]
```java
PriorityQueue<Integer> queue = new PriorityQueue<>();
queue.add(5);
queue.add(1);
queue.add(3);
System.out.println(queue.peek()); // 1
```

<br>

#### [Integer 내림차순]
- `Comparator` 구현체를 생성자에 넣으면 사용자 정의 우선순위를 구현할 수 있다.
- 아래 예시는 `Comparator.reverseOrder()`로 내림차순으로 만든 예시이다.

```java
PriorityQueue<Integer> queue = new PriorityQueue<>(Comparator.reverseOrder());
queue.add(5);
queue.add(1);
queue.add(3);
System.out.println(queue.peek()); // 5
```

<br>

#### [사용자 정의 클래스에 Comparable 구현]
- `Comparable`을 상속하여 `compareTo`를 클래스에 정의한다면 우선순위 큐는 해당 정렬 기준을 바탕으로 데이터를 정렬한다.

```java
PriorityQueue<Node> queue = new PriorityQueue<>();

queue.add(new Node(1, 4));
queue.add(new Node(2, 2));
queue.add(new Node(3, 5));

while (!queue.isEmpty()) {  // 큐에서 요소를 하나씩 꺼내어 출력
    Node node = queue.poll();
    System.out.println(node);
}
// to : 2, weight : 2
// to : 1, weight : 4
// to : 3, weight : 5

---

class Node implements Comparable<Node>{
    int to;
    int weight;

    public Node(int to, int w) {
        this.to = to;
        this.weight = w;
    }

    @Override
    public int compareTo(Node n) {  // 오름차순
        return this.weight - n.weight;
    }

    @Override
    public String toString() {
        return "to : " + to + ", weight : " + weight;
    }
}
```
> 자바 정렬에 대해 자세히 알아보기
>
> [자바의 여러가지 정렬 방법](https://ajroot5685.github.io/posts/%EC%9E%90%EB%B0%94-%EC%A0%95%EB%A0%AC/)

<br>

## 누군가가 물어본다면
<div class="spotlight1">
우선순위 큐는 우선순위가 높은 데이터가 먼저 나가는 자료구조입니다.
<br><br>
자바에서는 기본적으로 최소힙으로 구현되어 있고, 사용자가 정렬 기준을 설정해서 사용자 정의 우선순위를 적용시킬 수도 있습니다.
</div>