---
title: JVM(Java Virtual Machine)
date: 2024-03-19 16:19:00 +09:00
categories: [java]
tags:
  [
    java,
    JVM,
    Class Loader,
    Execution Engine,
    Garbage Collector,
    Runtime Data Area
  ]
---

## 누군가가 물어본다면
<div class="spotlight1">
JVM은 자바 애플리케이션이 운영체제에 관계없이 실행될 수 있게 바이트코드를 기계어로 변환하고 관리하는 가상 머신입니다.
<br>
Garbage Collection 모듈이 존재하여 메모리 관리를 대신 해줍니다.
</div>

<br>

## JVM
- Java Virtual Machine
- 스택 기반의 자바 가상 머신
- 자바 애플리케이션을 클래스 로더로 통해 읽어 들여 자바 API와 함께 실행한다.
- Java와 OS(운영체제) 사이에서 중개자 역할을 수행하여 Java가 OS에 구애받지 않고 독립적으로 작동이 가능하다.
- Garbage collection(메모리 관리)를 수행한다.
- 자바 바이트 코드를 실행할 수 있는 주체이다.

<br>

#### 가상머신
- 프로그램을 실행하기 위해 물리적 머신과 유사한 머신을 소프트웨어로 구현한 것

<br>

#### 자바 API
- 개발자가 사용할 수 있는 미리 구현된 클래스와 인터페이스의 집합, 즉 `자바 표준 라이브러리` 이다.
- `java.lang`, `java.util` 등 import 하여 사용하는 기능들이 자바 API이다.

<br>

#### Gabage Collection(GC)
- 프로그램이 동적으로 할당한 메모리 영역 중에서 더 이상 사용하지 않는 영역을 자동으로 찾아내어 해제하는 작업이다.
- 이로인해 C언어에서는 개발자가 직접 메모리 할당, 해제, 관리를 해줘야 하지만 Java에서는 이 일들을 JVM의 GC 모듈이 수행한다.

<br>

## JVM을 알아야 하는 이유
- 한정된 메모리를 효율적으로 사용하여 최고의 성능을 내기 위해서이다.
- 동일한 기능의 프로그램이더라도 메모리 관리에 따라서 성능이 좌우되기 때문에 JVM이 하는 역할을 이해하고 메모리를 효율적으로 사용하여 최고의 성능을 낼 수 있다.

<br>

## 자바 프로그램 실행 과정
1. 프로그램이 실행되면 JVM은 OS로부터 이 프로그램이 필요로 하는 메모리를 할당받는다. JVM은 이 메모리를 용도에 따라 여러 영역으로 나누어 관리한다.
2. 자바 컴파일러(`javac`)가 자바 소스 코드를 읽어들여 자바 바이트 코드(`.class`)로 변환시킨다.
3. Class Loader를 통해 class 파일들을 JVM으로 로딩한다.
4. 로딩된 class 파일들은 Execution Engine을 통해 해석된다.
5. 해석된 바이트 코드는 Runtime Data Area에 배치되어 실질적인 수행이 이루어지게 된다. 이러한 실행 과정 속에서 JVM은 필요에 따라 Thread Synchronization과 GC 같은 관리 작업을 수행한다.

![](/assets/img/240319/JVM.png)

<br>

## 각각의 역할

#### Class Loader(클래스 로더)
- 자바 프로그램은 `.java` 로 작성되며 이는 자바 컴파일러에 의해 바이트코드라는 중간 형태의 `.class` 파일로 변환된다.
- Runtime 시에 JVM 내로 클래스(`.class` 파일)를 로드하고 링크를 통해 배치하는 작업을 수행한다.
  > Runtime : 클래스를 처음으로 참조하는 시점
- 사용하지 않는 클래스들은 메모리에서 삭제한다.
- 동적 로드를 담당한다.

<br>

#### Execution Engine(실행 엔진)
- 클래스를 실행시키는 역할이다.
- 클래스 로더가 JVM 내의 Runtime Data Area에 바이트 코드를 배치시키고 이것은 실행 엔진에 의해서 실행된다.
- 실행 엔진은 바이트 코드를 실제로 JVM 내부에서 기계가 실행할 수 있는 형태로 변환한다.
  > 자바 바이트 코드는 비교적 인간이 보기 편한 형태로 기술된 것이다.
- 실행 엔진은 자바 바이트 코드를 명령어 단위로 읽어서 실행한다.
- 최초의 JVM은 인터프리터 방식이었기 때문에 속도가 느리다는 단점이 존재했지만, JIT 컴파일러 방식을 통해 이 점을 보완했다.

<br>

1. Interpreter(인터프리터)
- 자바 바이트 코드를 명령어 단위로 읽어서 실행한다.
- 한 줄씩 실행하기 때문에 느리다.

2. JIT(Just-In-Time) Compiler
- 인터프리터 방식의 단점을 보완했다.
- 인터프리터 방식으로 실행하다가 적절한 시점에 바이트 코드 전체를 컴파일하여 네이티브 코드로 변경하고 이후에는 더 이상 인터프리팅 하지 않고 네이트브 코드로 직접 실행한다.
- 네이티브 코드는 캐시에 보관하기 때문에 한 번 컴파일된 코드는 빠르게 수행된다.
- 한 번만 실행되는 코드라면 JIT 컴파일러가 컴파일하는 게 인터프리팅하는 것보다 오래 걸리므로 인터프리팅하는 것이 유리하다.
- 따라서 해당 메소드가 얼마나 자주 수행되는지 체크하고 일정 정도를 넘을 때만 컴파일을 수행하는 게 좋다.

<br>

#### Garbage Collector
- GC를 수행하는 모듈이 존재한다.

<br>

#### Runtime Data Area
- JVM이 프로그램을 수행하기 위해 OS로부터 할당받은 메모리 공간이다.
- 이 공간은 용도에 따라 여러 영역으로 나누어 관리한다.

![](/assets/img/240319/Runtime%20Data%20Area.png)

<br>

1. PC Register
- Thread가 시작될 때, 각각의 Thread 별로 생성되는 공간으로 현재 수행 중인 JVM 명령어 주소를 가지게 된다.
- CPU에 직접 접근하지 않고 Stack에서 주소를 뽑아서 가져온다.
- 현재 어떤 명령을 실행해야 할 지에 대한 기록을 담당한다.

2. JVM 스택 영역(Stack Area)
- 프로그램의 실행 과정에서 임시로 할당되었다가 메소드를 빠져나가면 바로 소멸되는 특성의 데이터를 저장하기 위한 영역이다.
- 메소드의 매개변수, 지역 변수 등 메소드의 정보를 저장한다.

3. Native Method Stack
- Java 외의 언어로 작성된 네이티브 코드를 위한 영역이다.
  > 네이티브 코드 : C나 C++ 같은 low-level 언어로 작성된 코드
- 네이티브 메소드 호출의 매개변수, 지역변수, 실행 결과 등을 관리한다.
- Java Native Interface(JNI) 를 통해 네이티브 코드를 호출한다.
- 자바 애플리케이션에서 시스템 호출이나 하드웨어 수준의 기능이 필요할 때 네이티브 코드를 사용한다.
  > ex. 시스템의 특정 정보 가져오기, 특정 하드웨어 장치를 직접 제어해야 하는 경우
- native method에서 사용되는 메모리는 GC가 관리하지 않으므로 메모리 누수 문제가 발생하지 않도록 주의해야 한다.

<div class="spotlight2">
Native Method Stack은 자바가 운영 체제나 하드웨어 수준의 기능과 상호작용할 수 있게 하는 중요한 메커니즘을 제공하지만, 메모리 관리, 호환성 등의 문제로 주의가 필요하다.
</div>

<ol start="4">

<li>
Method Area(Class Area, Static Area)
<ul>
<li>클래스 정보를 처음 메모리 공간에 올릴 때, 초기화 되는 대상을 저장하기 위한 메모리 공간</li>

<li>모든 쓰레드가 공유하는 메모리 영역이다. 클래스, 인터페이스, 메소드, 필드, Static 변수 등의 바이트 코드를 보관한다.</li>

<li>Runtime Constant Pool이라는 것이 존재하며, 이는 별도의 관리 영역으로 상수 자료형을 저장하여 참조하고 중복을 막는 역할을 수행한다. (각 클래스와 인터페이스의 상수, 메소드와 필드에 대한 모든 레퍼런스를 담고 있는 테이블이다.)</li>

<li>Java 7부터 String Constant Pool은 Heap 영역으로 변경되어 GC의 관리 대상이 되었다.</li>

</ul>
</li>

<li>
Heap(힙 영역)
<ul>
<li>객체를 저장하는 가상 메모리 공간이다.</li>

<li>런타임 시 동적으로 할당하여 사용하는 영역</li>

<li><code class="language-plaintext highlighter-rouge">new</code> 연산자로 생성된 객체와 배열을 저장한다.</li>

<li>클래스 영역에 올라온 클래스들로만 객체로 생성할 수 있으며, 세 부분으로 나눌 수 있다.</li>

<li>GC의 관리 대상에 포함된다.</li>

</ul>
</li>
</ol>

![](/assets/img/240319/heap.png)

<br>

- New/Young 영역
    - Eden : 객체들이 최초로 생성되는 공간
    - Survivor 0/1 : Eden에서 참조되는 객체들이 저장되는 공간

<br>

- Old 영역
    - New 영역에서 일정 시간 참조되고 살아남은 객체들이 저장되는 공간이다.
    - Eden 영역에서 인스턴스가 가득차게 되면 첫 번째 GC가 발생한다. (minor GC)
    - Eden 영역에 있는 값들을 Survivor 1 영역에 복사하고, 이 영역을 제외한 나머지 영역의 객체를 삭제한다.
    - Eden 영역에서 GC가 한 번 발생한 후 살아남은 객체는 Survivor 영역으로 이동한다. 이 과정을 반복하다가 살아남은 객체는 Old 영역으로 이동된다.

<br>

- Permanent Generation
    - 생성된 객체들의 주소값이 저장되는 공간이다.
    - 리플렉션을 사용하여 동적으로 클래스가 로딩되는 경우 사용된다.
    - Old 영역에서 살아남은 객체가 영원히 남아있는 곳이 아니다.
    - 이 영역에서 발생하는 GC는 Major GC의 횟수에 포함된다.