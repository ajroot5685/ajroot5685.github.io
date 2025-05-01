---
title: stack, queue, deque
date: 2024-03-17 19:20:00 +09:00
categories: [data structure]
tags:
  [
    data structure,
    java,
    stack,
    queue,
    deque
  ]
---

## Stack 이란?
- `쌓아놓은 더미` 를 뜻한다.
- Last-In First-Out (LIFO:후입선출) 이다.
- 가장 늦게 들어온 데이터가 가장 먼저 나간다.
- 자료의 출력 순서가 입력 순서의 역순으로 이루어질 때 사용한다.

<br>

**예시**
- 총기의 탄창에 들어있는 탄알
- 스마트폰에서 뒤로가기를 누를 때 이전에 수행되던 앱이 나오는 경우
- 프링글스

<br>

## Stack의 구조 및 사용

![stack](/assets/img/24/03/17/stack.png)

상단(stack top) : 스택에서 입출력이 이루어지는 부분

하단(stack bottom): 반대쪽 바닥 부분

요소(element): 스택에 저장되는 것

공백 스택(empty stack): 공백 상태의 스택 (= size 0)

포화 스택(full stack): 포화 상태의 스택

<br>

## 자바의 Stack
자바의 Stack 클래스는 Vector 클래스를 상속받기에 Thread-safe 하다.

**Thread-safe**
- 여러 쓰레드가 동시에 같은 Stack 인스턴스에 접근하더라도 한 쓰레드가 작업을 완료할 때까지 다른 쓰레드는 대기한다.

![stack-vector](/assets/img/24/03/17/stack-vector.png)

<br>

**stack 사용법**

- 자바는 java.util.Stack 클래스로 제공한다.

|메서드|설명|
|--|--|
|`boolean empty()`|Stack이 비어있는지 알려준다.|
|`Object peek()`|Stack의 맨 위에 저장된 객체를 반환한다.<br>pop과 달리 Stack에서 객체를 꺼내지는 않는다.|
|`Object pop()`|Stack의 맨 위에 저장된 객체를 꺼낸다.|
|`Object push(Object item)`|Stack에 객체(item)를 저장한다.|
|`int search(Object o)`|Stack에서 주어진 객체(o)를 찾아서 그 위치를 반환한다.<br>못 찾을 경우 -1을 반환한다.<br>배열과 달리 위치는 0이 아닌 1부터 시작한다.|

<br>

## stack 예시
```java
Stack<Number> stack = new Stack<>();
stack.push(1);
stack.push(2);
stack.push(3);

System.out.println(stack); // [1, 2, 3]

System.out.println(stack.pop()); // 3
System.out.println(stack); // [1, 2]

System.out.println(stack.peek()); // 2
System.out.println(stack); // [1, 2]

while(!stack.isEmpty()){
  stack.pop();
}

System.out.println(stack); // []

stack.push(4);
System.out.println(stack.size()); // 1
```

<br>

## Stack 클래스는 deprecaated 되었다.
Vector 클래스가 오래되었고 그만큼 취약점도 많기 때문에 이를 상속받은 Stack 도 쓰지 않기를 권장하고 있다.

자바 공식 문서에 따르면 Stack을 쓰지말고 Deque 클래스를 사용하여 구현할 것을 권장하고 있다.

<br>

## Queue 란?
- `꼬리` 를 뜻한다.
- First-In First-Out (FIFO:선입선출) 이다.
- 가장 먼저 들어온 데이터가 가장 먼저 나간다.
- 자료의 출력 순서가 입력 순서로 이루어질 때 사용한다.

**예시**
- 티켓 판매부스에서 줄을 서서 기다리는 사람들
- 한줄로 나란히 가야하는 차들

<br>

## queue의 구조 및 사용

![queue](/assets/img/24/03/17/queue.png)

후방(back) : 큐에서 입력이 이루어지는 부분

전방(front) : 큐에서 출력이 이루어지는 부분

<br>

## 자바의 Queue
자바의 Queue 인터페이스는 Collections 클래스를 확장한다.

Vector와는 다른 자료구조를 사용하여 Thread-safe 하지 않지만, ConcurrentLinkedQueue는 Thread-safe 하다.

**Queue 인터페이스를 구현하는 구현체들**
- LinkedList
- PriorityQueue
- ArrayDeque

<br>

## queue 예시(LinkedList)
```java
import java.util.LinkedList;
import java.util.Queue;

Queue<String> queue = new LinkedList<>();

// add : 용량 초과 시 exception 발생
queue.add("Apple");
System.out.println(queue); // [Apple]

// offer : 용량 초과 시 false 반환
queue.offer("Banana");
System.out.println(queue); // [Apple, Banana]

// remove : 제거할 데이터가 없으면 exception 발생
queue.remove();
System.out.println(queue); // [Banana]

// poll : 제거할 데이터가 없으면 null 반환
queue.poll();
System.out.println(queue); // []
        
queue.offer("Cherry");
queue.add("Date");
System.out.println(queue); // [Cherry, Date]

// exception : 제일 앞에 데이터가 없으면 exception 발생
System.out.println(queue.element()); // Cherry

// peek : 제일 앞에 데이터가 없으면 null 반환
System.out.println(queue.peek()); // Cherry

queue.clear();
System.out.println(queue); // []
        
queue.add("Elderberry");
System.out.println(queue); // [Elderberry]
        
System.out.println(queue.size()); // 1
        
System.out.println(queue.contains("Elderberry")); // true
        
System.out.println(queue.isEmpty()); // false
```

<br>

## Deque 란?
- Double-Ended Queue의 약자로 `양방향 큐` 를 뜻한다.
- 스택이나 큐보다 더 유연한 연산이 가능하다.
- 스택, 큐와 똑같은 사용할 수 있다.

<br>

## 자바에서 deque의 구조 및 사용

- deque는 queue와 마찬가지로 인터페이스로 구성되어 있다.

- 자세한 메서드는 아래 사이트 참고바란다.

  [[Java(자바)] Deque(덱/데크) 자료구조](https://soft.plusblog.co.kr/24)