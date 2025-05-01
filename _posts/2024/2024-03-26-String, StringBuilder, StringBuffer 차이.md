---
title: String, StringBuilder, StringBuffer 차이
date: 2024-03-26 13:09:00 +09:00
categories: [java]
tags:
  [
    java,
    string,
    stringbuilder,
    stringbuffer
  ]
---

## String 이란?
- Immutable(불변)하기 때문에 `+` 등의 concat 연산 시 원본을 변경하지 않고 새로운 String 객체를 생성한다.
    - 이로 인해 메모리 공간의 낭비가 발생하고 성능이 떨어진다.
- 불변 객체이기 때문에 멀티 쓰레드 환경에서 동기화를 신경쓰지 않아도 된다.(Thread-safe)
- 문자열 연산이 적고, 조회가 많은 상황에서 쓰기 좋다.

<br>

## StringBuilder, StringBuffer
- 공통점
    - Mutable(가변)하다.
    - 가변 객체이므로 문자열 연산 시 새롭게 객체를 생성하지 않고, 처음에 만든 객체에 연산, 크기 변경으로 문자열을 변경한다.
    - 따라서 문자열 연산이 자주 발생하는 상황에서 성능적으로 유리하다.

- 차이점
    - `StringBuilder` : Thread-safe 하지 않다. 즉, 멀티 쓰레드를 지원하지 않는다.
    - `StringBuffer` : Thread-safe 하다. 즉, 멀티 쓰레드를 지원한다.
- Thread-safe 하다는 것은 동기화 관련 처리를 추가적으로 해줘야 하기 때문에 **성능이 상대적으로 떨어질 수 밖에 없다.**
- 따라서 동기화가 필요한 상황이라면 `StringBuffer`, 필요하지 않다면 `StringBuilder` 가 유리하다.

> 때문에 코딩테스트와 같이 멀티 쓰레드가 요구되지 않는 환경에서 `StringBuilder` 가 자주 쓰인다.

<br>

## JDK 1.5 이후 String 연산
- JDK 1.5 이후부터는 컴파일 타임에 StringBuilder로 변경한다.
- 그러나 모든 경우에 적용되는 것은 아니다.
- 단일 문장에서의 문자열 연결이 이루어질 때 적용된다.
    ```java
    String str = "Hello, " + name + "!";
    ```
    - 이 경우에 컴파일러가 StringBuilder를 자동으로 사용하여 문자열 연결을 최적화한다.
- 다음의 경우 적용되지 않는다.
    - ex. 단일 문장이 아닌 반복문 내에서 문자열을 `+` 연산자를 사용하여 연결하는 경우
    ```java
    String result = "";
    for(int i=0;i<10;i++){
        result += strarr[i];
    }
    ```
    - 이 경우 자동으로 최적화가 이루어지지 않으므로 직접 `StringBuilder` 를 사용하는 것이 좋다.
    ```java
    StringBuilder sb = new StringBuilder();
    for(int i=0;i<10;i++){
        sb.append(strarr[i]);
    }
    String result = sb.toString();
    ```

<br>

## 누군가가 물어본다면
<div class="spotlight1">
String 은 불변하여 concat 연산 시 새로운 String 객체를 생성하고 이로 인해 성능이 떨어집니다.
<br><br>
반면 StirngBuilder와 StringBuffer 는 가변하여 문자열 연산이 자주 발생하는 상황에서 좋은 성능을 보입니다.
<br><br>
StringBuilder는 Thread-safe 하지 않고, StringBuffer는 Thread-safe 하여 동기화가 필요한 상황에서는 StringBuffer가 유리합니다.
</div>