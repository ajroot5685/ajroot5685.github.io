---
title: 자바는 Call by value 일까? Call by reference 일까?
date: 2024-04-01 12:53:00 +09:00
categories: [java]
tags:
  [
    java,
    call by value,
    call by reference
  ]
---

## Call by Value
- 함수에 인자를 전달할 때 인자의 실제 값이 `복사`되어 매개변수에 저장된다.
- 이렇게 복사된 값은 함수 내에서 지역 변수로 사용된다.
    - 함수 내에서 이 값을 변경해도 원래 변수의 값은 영향을 받지 않는다.

- 장점
    - 원본 데이터가 변경될 위험이 없어 안전하다.
    - 함수가 자신만의 데이터 복사본을 가지므로 부작용이 없다.
- 단점
    - 큰 데이터를 복사할 때 성능과 메모리 사용량이 증가할 수 있다.

<br>

## Call by Reference
- 함수에 인자를 전달할 때 인자의 메모리 주소가 전달된다.
- 함수 내에서 해당 주소를 통해 직접 원본 데이터를 변경할 수 있다.

- 장점
    - 큰 데이터를 복사할 필요가 없어 성능과 메모리 효율성이 높아진다.
    - 함수를 통해 원본 데이터를 쉽게 수정할 수 있어 유용하다.
- 단점
    - 함수가 원본 데이터를 변경할 수 있기 때문에 예기치 않은 부작용이 발생할 수 있다.
    - 프로그램의 흐름을 추적하기 어렵다.

<br>

## 자바
- 자바는 `Call by Value` 이다.
- `Primitive Type`(기본 자료형)과 `Reference Type`(참조 자료형) 모두 `Call by Value` 다.

<br>

#### 기본 자료형

```java
public class Test {
    public static void main(String[] args) {
        int n = 10;
        System.out.println(n); // 10
        plus(10);
        System.out.println(n); // 10
    }

    public static void test(int n) {
        n += 1;
        System.out.println(n); // 11
    }
}
```
- 기본 자료형값이 바뀌지 않는걸 볼 수 있다.

<br>

#### 참조 자료형
- 자바에서 배열과 객체는 메모리의 특정 위치에 저장되어 있고, 할당된 변수는 그 메모리 위치를 가리키는 주소를 저장한다.

```java
public class Test {
    public static void main(String[] args) {
        int[] arr = {1, 2, 3};
        for (int num : arr) System.out.print(num + " "); // 1 2 3
        System.out.println();

        test(arr);

        for (int num : arr) System.out.print(num + " "); // 10 20 30
    }

    public static void test(int[] a) {
        for (int i = 0; i < a.length; i++) {
            a[i] *= 10;
        }
    }
}
```
- 참조 자료형의 경우 주소값이 전달되었기 때문에 원본이 영향을 받는다는 것을 볼 수 있다.
- `call by reference` 라고 헷갈릴 수 있지만 배열, 객체의 주소값의 `복사본`이 전달되므로 엄밀히 따지면 `call by value` 다.

<br>

#### call by reference 인 C++ 예시

```c++
#include <iostream>
using namespace std;

void swap(int &x, int &y) {
    int temp = x;
    x = y;
    y = temp;
}

int main() {
    int a = 10;
    int b = 20;
    
    cout << "Before swap - a: " << a << ", b: " << b << endl; // Before swap - a: 10, b: 20
    
    swap(a, b);
    
    cout << "After swap - a: " << a << ", b: " << b << endl; // After swap - a: 20, b: 10
    
    return 0;
}
```
- `&` 를 이용하여 **변수의 주소값**을 전달하고 있다.
    > 복사본이 아니다.

<br>

## 누군가가 물어본다면
<div class="spotlight1">
<code class="language-plaintext highlighter-rouge">Call by Value</code> 는 함수에 인자를 전달할 때 인자의 실제 값이 복사되어 전달되는 방식입니다.
<br><br>
<code class="language-plaintext highlighter-rouge">Call by Reference</code> 는 함수에 인자를 전달할 때 인자의 메모리 주소 값이 전달되는 방식입니다.
<br><br>
자바는 <code class="language-plaintext highlighter-rouge">Call by Value</code> 입니다.
</div>