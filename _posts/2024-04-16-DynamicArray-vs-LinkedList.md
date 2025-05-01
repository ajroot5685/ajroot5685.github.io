---
title: Dynamic Array vs Linked List
date: 2024-04-16 10:18:00 +09:00
categories: [data structure]
tags:
  [
    data structure,
    array,
    dynamic array,
    linked list
  ]
---

## 🤔 Array
- 입력된 데이터들이 메모리 공간에서 연속적으로 저장되어 있는 자료구조
- 메모리상에서 연속적으로 저장되어 있는 특징을 갖기 때문에, index를 통한 접근이 용이하다.
- 배열의 크기는 처음 생성할 때 정하며, 이후에는 변경할 수 없다.

![array](/assets/img/24/04/16/array.png)

<br>

#### [Array 시간복잡도]
- 탐색 : `O(1)`
    - 접근하고자 하는 인덱스를 알고 있을 때, 해당 메모리 주소로 바로 접근할 수 있다.
- 삽입 및 삭제 : `O(n)`
    - 배열의 끝에서 작업이 수행되면 O(1)이지만 보통의 경우 중간에 삽입/삭제된다.
    - 이에 따른 뒤의 데이터들도 옮겨주는 작업을 해야하므로 `O(n)`이 소요된다.

<br>

#### [Array의 한계]
1. 생성 시에 길이가 정해지므로 길이를 바꾸는 데에 비효율적이다.
    - 새로운 길이의 배열을 메모리에 따로 할당하고, 데이터를 복사한 후, 기존의 배열을 삭제하는 세 단계를 거쳐야한다.
    - 길이를 늘리는 작업 자체가 지원되지 않으므로 개발자가 직접 로직을 설계해야 한다.
2. `Inner Fragmentation`
    - 데이터가 없어도 메모리를 차지하기 때문에 원소가 삭제되어도 빈자리(null)가 남게 된다.
3. `Outer Fragmentation`
    - array는 메모리의 연속된 공간을 필요로 하기 때문에 수용할 수 있는 충분한 공간이 있음에도 연속되지 않다면 저장할 수 없다.

    > 아래 그림에서 만약 배열이 50KB의 연속된 메모리 공간을 필요로 한다면 메모리 공간이 다 나뉘어져 있으므로 저장할 수 없다.

    ![outer fragment](/assets/img/24/04/16/outer%20fragmentation.png)

<br>

## 😮 Dynamic Array
- Java : `ArrayList`
- C++ : `Vector`

- 고정된 길이를 갖는 Array의 한계를 극복하기 위해 나온 자료구조이다.
- 크기가 가변적으로 변하는 선형리스트이다.
    - 따라서 크기가 변하는 배열을 사용할 때나 배열의 크기를 알 수 없는 경우에 사용한다.
- 시간복잡도는 일반 array와 같다.
- 런타임 시에 자동으로 길이가 증가하지만 `Fragmentation` 문제는 해결되지 않았다.

<br>

## 😙 Linked List
- 여러 개의 노드들이 순차적으로 연결된 형태를 갖는 자료구조
- 첫번째 노드를 Head, 마지막 노드를 Tail 이라고 한다.
- 데이터를 저장하는 각 노드는 이전 노드와 다음 노드의 상태만 알고 있으면 된다.
    - 배열과는 다르게 메모리를 연속적으로 사용하지 않는다.

<br>

#### [LinkedList 시간복잡도]
- 탐색 : `O(n)`
    - 처음부터 노드를 순회하여 탐색하므로 `O(n)` 이다.
- 삽입 및 삭제 : `O(1)`
    - 불필요한 데이터의 복사가 없어 데이터의 삽입, 삭제 시 유리하다.
    - 그러나 삽입, 삭제를 하기 위한 노드를 찾는 시간은 `O(n)` 이 걸린다.

<br>

## 😨 할당 영역
- 일반적으로 배열은 stack 에 할당된다.
- 그러나 `new` 연산자로 생성되는 경우 heap 에 할당된다.
- JAVA는 모두 `new` 연산자로 생성되므로 모두 heap 에 할당된다.
- LinkedList도 `new` 로 생성하므로 heap에 할당된다.

```c++
int stackArray[5];  // stack 에 할당된다.
int* array = new int[10];   // heap 에 할당된다.
```

```java
int[] array = new int[10];  // heap 에 할당된다.
```

<br>

## 🤗 자료구조 선택
- 데이터의 검색이 주가 되는 경우 `Dynamic Array`를 사용한다.
- 데이터의 삽입, 삭제가 빈번하다면 `LinkedList`를 사용한다.

<br>

#### [백준 문제를 통한 자료구조 차이 체험]
- [16235 - 나무 재테크 (골드 3)](https://www.acmicpc.net/problem/16235)

- 삼성 기출 문제의 구현 문제이다.
- 시간 제한이 0.3초로 짧게 설정되어 있고 삭제 연산이 빈번하게 발생한다.
- 따라서 `Dynamic Array`를 사용하면 시간초과가 발생하므로 `LinkedList`를 사용해야 한다.

<br>

## 누군가가 물어본다면
<div class="spotlight1">
Dynamic Array는 Array의 크기 변경 불가라는 단점을 보완하기 위해 만들어졌습니다. 하지만 <code class="language-plaintext highlighter-rouge">Fragmentation</code> 문제는 해결되지 않았습니다.
<br><br>
검색이 자주 발생하는 경우 Array를, 삽입 및 삭제가 자주 발생하는 경우 LinkedList를 사용하는 것이 더 유리합니다.
</div>