---
title: 병렬 스트림(ParallelStream) 간단하게 알아보기
date: 2024-04-09 12:00:00 +09:00
categories: [java]
tags:
  [
    java,
    stream,
    parallelstream,
    fork/join framework,
    work-stealing
  ]
---

## 🤔 병렬 스트림(ParallelStream)
- 자바 8에서 등장한 stream의 기능으로 병렬 처리를 쉽게 처리할 수 있는 메서드이다.
- 개발자가 직접 쓰레드 혹은 쓰레드풀을 생성/관리 할 필요 없이 `parallelStream()`, `parallel()` 만 사용하면 병렬 처리가 된다.
- `Fork / Join Framework` 관리 방식을 이용한다.

<br>

## 😮 Fork / Join Framework
- 자바 7에 도입되어 병렬 실행을 용이하게 하는 도구이다.
- 데이터를 작은 부분으로 나누고, 각 부분을 병렬로 처리한 다음, 결과를 결합하는 방식이다.
    > 분할 기법과 유사하다.
- `Work-Stealing 메커니즘`을 사용한다.

<br>

#### [주요 구성 요소]
- `ForkJoinPool`
    - task의 실행을 관리하는 병렬 처리용 쓰레드 풀이다.
    - 사용자가 실행하고자 하는 task들을 보관한다.
- `Worker Thread`
    - 실제 작업을 수행하는 쓰레드이다.
    - ForkJoinPool로부터 task를 가져와(take) 처리한다.
    - 각 worker thread 들은 자신만의 work deque 를 가지고 있다.

<br>

#### [Work-Stealing 메커니즘]

![work-stealing](/assets/img/24/04/09/work-stealing.png)

1. 사용자가 task를 등록한다.(submit)
2. inbound queue에 task가 들어가고, worker thread A와 B가 task를 가져가 처리한다.(take)
3. A와 B는 자신만의 work deque 를 가지고 있으며, 자신의 task가 없다면 다른 worker thread의 task를 가져와 처리한다.
    > 이 work-strealing 메커니즘으로 CPU 자원이 놀지 않고 최적의 성능을 낼 수 있게 된다.

<br>

#### [상세하게 살펴보기]
- worker thread는 기본적으로 **한번에 하나의 task만 가져와서 처리**한다.
- task를 처리하는 과정에서 **task 분할**이 일어날 수 있고 하나의 큰 task가 여러개의 작은 task로 쪼개져서 worker thread의 deque에 추가된다.
- 때문에 한번에 하나씩 처리하더라도 task 분할로 인해 자신의 deque에 여러 개의 task가 등록될 수 있고 다른 쓰레드가 가져갈 수 있는 task가 생기는 것이다.

<br>

## 🚨 주의사항

#### [Thread Pool 공유]
- parallelStream은 별도의 설정이 없다면 내부적으로 하나의 Thread Pool을 모든 parallelStream이 공유한다.
- 즉, parallelStream 별로 Thread Pool을 만드는 것이 아니다.
- 만약 I/O 작업으로 모든 쓰레드를 계속 점유하고 있으면 요청은 queue에 계속 쌓이기만 하여 처리하지 못하는 상황이 발생한다.
> Custom Thread Pool을 생성하여 별도의 쓰레드 풀을 생성해서 해결할 수 있다.

<br>

#### [Collection 별 성능 차이]
- parallelStream은 분할되는 작업의 단위가 균등하게 나누어져야 하며, 나누어지는 작업의 비용이 높지 않아야 순차적 방식보다 효율적으로 이루어질 수 있다.
  - array, arrayList와 같이 전체 사이즈를 알 수 있는 경우에는 분할 처리가 빠르고 비용이 적게 든다.
  - 하지만 linkedList와 같이 사이즈를 정확히 알 수 없는 데이터 구조는 분할되지 않고 순차 처리를 하므로 성능 효과를 보기 어렵다.

<br>

> 출처
> 
> [[Java] 병렬 스트림(ParallelStream) 사용 방법 및 주의사항](https://dev-coco.tistory.com/183)

<br>

## 누군가가 물어본다면
<div class="spotlight1">
parallelStream() 은 편리하게 병렬 처리를 사용하게 하지만, 무조건 더 나은 결과를 보장할 수는 없습니다.
<br><br>
분할이 잘 이루어질 수 있는 데이터 구조 혹은 작업이 독립적이면서 CPU 사용이 높은 작업(CPU bound)에 적합합니다.
</div>