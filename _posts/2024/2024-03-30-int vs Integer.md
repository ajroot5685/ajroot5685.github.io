---
title: int vs Integer
date: 2024-03-30 15:23:00 +09:00
categories: [java]
tags:
  [
    java,
    int,
    Integer,
    wrapper class
  ]
---

## int
- 정수를 저장한다.
- `Primitive type` (기본 자료형) 이다.
    > 기본 자료형은 스택 메모리에 값을 직접 저장하기 때문에 처리 속도가 빠르다.
- 산술 연산이 가능하며, `null` 값을 가질 수 없다.

<br>

## Integer
- 정수를 저장한다.
- `Reference type` (참조 자료형) 이다. 객체로 분류된다.
    > 참조 자료형은 힙 메모리에 객체를 저장하고, 변수는 그 객체를 가리키는 주소를 저장한다.
- `int` 값을 감싸는 `Wrapper class`로, 객체지향 프로그래밍의 이점을 제공한다.
    - null 값을 가질 수 있고, 클래스이므로 메서드를 가질 수 있다.
- `unboxing`을 하지 않으면 산술 연산이 불가능하다.
    > 그러나 자바 5부터는 자동으로 `unboxing` 이 수행되므로 개발자가 명시적으로 수행하지 않아도 된다.

<div class="spotlight2">
<code class="language-plaintext highlighter-rouge">Wrapper class</code>는 자바의 기본 자료형을 객체로 다룰 수 있게 해주는 클래스이다.
</div>

![wrapper class](/assets/img/24/03/30/wrapper%20class.png)

<br>

## 용도
- `int`는 빠른 속도를 가지기에 숫자 연산이나 반복문 카운터 같이 성능이 중요한 상황에서 사용된다.
- `Integer`는 null 할당이 필요하거나, 컬렉션 객체에 숫자를 저장할 때 사용된다.
    - 자바의 컬렉션 프레임워크는 기본형을 직접 사용할 수 없기 때문에 `Wrapper Class` 를 사용해야 한다.

<br>

## autoboxing, unboxing
- 자바 5부터 `autoboxing`과 `unboxing`을 지원한다. 이는 int 와 Integer 사이의 변환을 자동으로 해준다.

![boxing](/assets/img/24/03/30/boxing.png)

#### Autoboxing
- 기본 자료형의 값을 해당 래퍼 클래스의 객체로 자동으로 변환하는 과정이다.
```java
Integer a = 25; // 25는 int 이지만, 자동으로 Integer 객체로 박싱된다.
```

<br>

#### Unboxing
- 래퍼 클래스의 객체를 기본 자료형의 값으로 자동으로 변환하는 과정이다.
```java
Integer a = new Integer(25);
int i = a; // a 객체가 자동으로 int 값으로 언박싱된다.
```

<br>

## 누군가가 물어본다면
<div class="spotlight1">
int 는 기본 자료형으로 성능이 좋아 반복문 카운팅 등 성능이 중요할 때 사용합니다.
<br><br>
Integer 는 wrapper class로, 기본 자료형을 참조하는 객체입니다. 컬렉션 작업을 할 때 주로 사용합니다.
<br><br>
자바 5부터 autoboxing과 unboxing이 지원되어 두 자료형 간의 변환을 개발자가 신경쓰지 않아도 됩니다.
</div>