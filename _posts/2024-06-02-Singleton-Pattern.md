---
title: 인스턴스의 개수를 1개로 제한하는 싱글톤 패턴
date: 2024-06-02 19:08:00 +09:00
categories: [design pattern]
tags:
  [
    design pattern,
    spring
  ]
---

## 싱글톤 패턴이란
- 클래스의 인스턴스가 딱 1개만 생성되는 것을 보장하는 디자인 패턴이다.

<br>

#### [여러 개의 웹 요청이 들어올 때]
- 만약 싱글톤 패턴을 사용하지 않는다면, 한 번에 여러 개의 요청이 들어올 때마다 해당 요청을 처리하기 위한 객체들이 여러 개 생성될 것이다.

    <img src="/assets/img/24/06/02/singleton1.png" alt="singleton1" width=600>

- 모든 요청에 대해 각각 객체가 생성되고 소멸되는 것은 메모리 낭비가 심하다.
    > 객체 조회 오버헤드가 1이라고 가정하면, 객체 생성에는 1000의 오버헤드가 발생한다.
- 따라서 객체를 한개만 생성하고 공유하여 해결하는 것이 **싱글톤 패턴**이다.

<br>

## 싱글톤 패턴 코드(JAVA)

#### [가장 단순하고 안전한 방법]
- 멀티스레드 환경에서 안전하지만, 클래스가 로드될 때 인스턴스를 생성하므로, 애플리케이션 시작 시 메모리를 차지하게 된다.

- `static` 키워드로 instance로 정의된 싱글톤 객체를 static 영역에 하나만 존재하도록 설정한다.
- `final` 키워드로 한번 정의되면 바뀌지 않도록 설정한다.
- `getInstance()` 로 다른 곳에서 싱글톤 객체를 가져올 수 있다.
- 생성자를 `private` 으로 정의하지 않으면 다른 곳에서 실수로 새로 생성할 수 있으므로 반드시 재정의 해야 한다.

```java
public class Singleton {

    private static final Singleton instance = new Singleton();

    public static Singleton getInstance() {
        return instance;
    }

    private Singleton() {    // 필수
    }
    
    public void logic(){
        System.out.println("싱글톤 객체 로직 호출");
    }
}
```

<br>

#### [Double-Checked Locking]
- 클래스 로딩 시가 아닌 처음으로 사용하려 할 때 객체를 생성하는 방법이다.
- 성능을 위해 두번 체크하여 동기화 블록의 범위를 최소화했다.

- `volatile` 키워드로 메모리 배치를 순서대로 강제하여, 초기화가 완료되기 전에 다른 스레드가 인스턴스에 접근하지 못하도록 한다.
    > 이 키워드를 사용하지 않으면, 초기화 전에 메모리 주소가 할당되어 다른 쓰레드가 초기화가 완료되지 않은 인스턴스를 볼 수도 있다.

```java
public class Singleton {
    private static volatile Singleton instance;

    private Singleton() {   // 필수
    }

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

<br>

## 싱글톤 패턴의 문제점
- 의존관계상 클라이언트가 구체 클래스에 의존한다.
    > `Singleton.getInstance()`를 사용해야 하므로 `DIP`를 위반한다.
- 구체 클래스에 의존하기 때문에 `OCP` 원칙을 위반할 가능성이 높다.
- `Test Driven Development`(TDD)에서 걸림돌이 된다.
    > 단위 테스트는 서로 독립적이어야 하는데, 싱글톤 패턴을 사용하면 **독립적인 인스턴스**를 만들기 어렵기 때문
- 모듈 간의 결합을 강하게 만들어, 결론적으로 유연성이 떨어진다.
- 위와 같은 이유들 때문에, 안티 패턴이라고 불린다.

<br>

## 스프링에서의 싱글톤 패턴
- 스프링에서 **스프링 컨테이너**는 빈으로 등록된 객체들을 기본적으로 싱글톤으로 관리해준다.
- 덕분에 싱글톤 패턴을 위한 추가적인 코드가 들어가지 않아도 되고, `DIP, OCP, 테스트, private 생성자`로부터 자유롭게 싱글톤을 사용할 수 있다.

<img src="/assets/img/24/06/02/singleton2.png" alt="singleton2" width=600>

<br>

## 싱글톤 방식의 주의점
- 싱글톤 객체는 상태를 무상태(`stateless`)로 설계해야한다.
    - 특정 클라이언트에 의존적인 필드가 있으면 안된다.
    - 특정 클라이언트가 **값을 변경할 수 있는 필드가 있으면 안된다.**
    - 가급적 읽기만 가능해야 한다.
    - 필드 대신에 자바에서 공유되지 않는, 지역변수, 파라미터, ThreadLocal 등을 사용해야 한다.

<br>

#### [잘못된 싱글톤 방식 예시]

```java
public class StatefulService {

    private int price;  // 상태를 유지하는 필드

    public void order(String name, int price){
        System.out.println("name = " + name + " price = " + price);
        this.price = price;
    }

    public int getPrice(){
        return price;
    }
}
```

- 웹의 각 요청들은 보통 쓰레드로 분리되어 실행되는데 이렇게 공유되는 필드가 있는 경우 `Race Condition` 이 발생할 수 있다.
- 실무에서도 종종 발생하고, 복잡한 상속 관계 등으로 인해 발견하기도 어렵다고 한다.
    > 싱글톤 방식의 주의점을 잘 인지하고 있다면 쉽게 해결하고, 방지할 수 있을 것이다.

<br>

## 누군가가 물어본다면
<div class="spotlight1" markdown="1">
싱글톤 패턴은 클래스의 인스턴스를 딱 **1개만 생성**되는 것을 보장하는 디자인 패턴입니다.
<br><br>
`객체 지향`, `테스트` 등과 관련된 문제로 **안티패턴**으로 불리지만, 스프링에서는 `스프링 컨테이너` 덕분에 문제 없이 편리하게 사용할 수 있습니다.
</div>