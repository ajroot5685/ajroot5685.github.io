---
title: 자바에서 new ArrayList<>(); 의 <>와 ()는 무엇일까?
date: 2024-04-18 16:16:00 +09:00
categories: [java]
tags:
  [
    java,
    generic,
    constructor,
    type safe
  ]
---

## 🤔 <> 는 무엇일까? - Generic
- `List<String> list = new ArrayList<>();` 에서 `<>` 는 무엇일까?
- `다이아몬드 연산자` 라고도 불리는 `<>`는 제네릭 타입을 명시할 때 사용된다.
- 제네릭(Generic) 은 클래스 또는 메서드에서 사용될 데이터 타입을 일반화하여 코드의 재사용성을 높이고 타입 안정성을 제공한다.
- 자바 7부터 도입되어 컴파일러가 문맥을 기반으로 필요한 제네릭 타입을 추론할 수 있게 해준다.
    - 위 예시에서 컴파일러는 `<>` 안에 `String`이 들어갈 것이라고 추론한다.
  
<br>

#### [제네릭의 일반화]
- ArrayList의 구현 코드를 보면 다음과 같다.
    - `public class ArrayList<E> ... {}`
    - 여기서 `E`는 Element를 의미하고 개발자가 자유롭게 정의할 수 있는 **타입 매개변수의 이름**이다.
        > 우리가 함수의 인자를 설정할 때 이름을 지정하는 것과 같다.
    - 단, 타입 매개변수는 참조 타입(클래스, 인터페이스)만을 받기 때문에 int, char와 같은 기본 데이터 타입은 사용할 수 없다.

    <div class="spotlight2" markdown="1">
    제네릭 덕분에 ArrayList는 `ArrayList<String>, ArrayList<Integer>, ...` 와 같이 각 타입에 따른 중복된 코드를 작성할 필요가 없다.
    <br><br>
    이것을 일반화라고 한다.
    </div>

    - 타입 매개변수 명명 규칙

      |타입|뜻|
      |:--:|:--:|
      |\<T>|Type|
      |\<E>|Element|
      |\<K>|Key|
      |\<V>|Value|
      |\<N>|Number|

      > 규칙일 뿐이고, 매개변수 이름은 개발자가 마음대로 정해도 된다.

<br>

#### [타입 안정성(Type Safety)]
- 프로그램 내에서 데이터 항목이 예상된 데이터 타입으로만 사용되도록 보장하는 특성
- 타입 안정성을 유지함으로써, 컴파일 단계에서 개발자가 오류를 잡을 수 있어 런타임 에러 방지와 프로그램의 신뢰성을 높이는 데 기여한다.
  > 자바스크립트는 타입 안정성이 없기 때문에 많은 사람들이 타입 안정성을 보장해주는 `TypeScript`를 사용한다.

```java
List<String> list = new ArrayList<>();
list.add(123);  // 문자열이 아닌 정수를 넣고자할 때, 컴파일 에러가 발생하여 개발자가 빠르게 오류를 고칠 수 있다.
```

<br>

## 😮 <> 를 안쓴다면? - Raw Type
- `List list = ArrayList();`
- 다이아몬드 연산자를 생략하면 자바 컴파일러는 해당 객체를 `raw type`으로 처리한다.
- raw type 은 제네릭이 도입되기 전에 사용되던 클래스와 인터페이스이다.
- 예를 들어, 제네릭 타입 `List<E>` 의 raw type은 `List` 이다.
- 그러나 레거시 프로젝트를 위해, 즉 backward compatibility 를 위해 현재도 사용할 수 있다.

<br>

#### [Raw Type의 문제점]
- 제네릭의 주요 이점인 타입 안정성이 사라진다.
- 컴파일러가 타입 체크를 수행할 수 없다.
- 다양한 타입의 객체를 추가할 수 있지만, 각 타입의 요소에 대한 타입 체크 로직이 제대로 설계되어있지 않으면 `ClassCastException` 과 같은 예외를 발생시킬 수 있다.

```java
List list = new ArrayList();
list.add(123);

String str = list.get(0); // 컴파일 에러가 아닌 런타임 에러가 발생하여 개발자가 오류를 잡기 힘들다.
```

<br>

## 🤨 () 는 무엇일까? - Constructor
- 자바에서 객체를 생성할 때 사용되는 생성자 호출을 나타낸다.
- 미리 정의된 자료구조들은 여러 방법으로 생성자를 이용할 수 있다.

<br>

#### [ArrayList 생성자 예시]

```java
// 기본 생성자
ArrayList<String> list = new ArrayList<>();

---
// 용량을 지정하는 생성자
ArrayList<String> list = new ArrayList<>(100);

---
// 원래 있던 리스트의 요소들을 복사하는 생성자
List<Stirng> list2 = Arrays.asList("Hello", "World");
ArrayList<String> list = new ArrayList<>();
```

<br>

## 누군가가 물어본다면
<div class="spotlight1" markdown="1">
`<>` 는 제네릭 타입 명시자로, 컴파일러가 필요한 제네릭 타입을 자동으로 추론합니다.
<br>
제네릭은 데이터 타입을 일반화하여 불필요한 중복 코드를 만들지 않게 하고, 타입 안정성 또한 제공합니다.
<br><br>
`()` 는 생성자 호출을 나타내어, 자바의 자료구조에 내장된 여러가지 생성자를 활용할 수 있습니다.
</div>