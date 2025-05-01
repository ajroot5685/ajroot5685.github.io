---
title: 코드와 함께 간단하게 옵저버 패턴 알아보기
date: 2024-06-12 14:12:00 +09:00
categories: [design pattern]
tags:
  [
    design pattern
  ]
---

## 옵저버 패턴(Observer Pattern)
> 스타크래프트의 옵저버 유닛처럼 관찰자를 뜻하는 패턴이다.

- 옵저버(관찰자)들이 관찰하고 있는 옵저버블(대상자)의 상태가 변화가 있을 때마다 **대상자는 자신의 목록에 있는 관찰자들에게 통지**하고, 알림을 받은 **관찰자들은 그에 따른 작업을 수행**하는 행동 패턴이다.
- 객체들 사이의 일대다 의존성을 정의할 때 사용된다.
- 주로 이벤트 핸들링 시스템에서 많이 활용된다.
- `Kafka` 나 `RabbitMQ` 와 같은 `Pub/Sub 모델`로도 알려져 있다.

<br>

## 장단점

#### [장점]
- 주제의 상태 변경을 주기적으로 조회하지 않고 자동으로 감지할 수 있다.
- 코드를 변경하지 않고 새로운 구독 클래스를 도입할 수 있어 OCP를 준수한다.

<br>

#### [단점]
- 알림 순서를 제어할 수 없다.
- 코드 복잡도가 증가한다.
- 다수의 옵저버 객체를 등록하고 해지하지 않는다면 메모리 누수가 발생할 수도 있다.

<br>

## 예시
- 뉴스 피드 또는 유튜버 구독으로 쉽게 이해할 수 있다.
- A, B, C가 유튜버 Y를 구독한다고 했을 때,
- Y가 동영상을 업로드하면 A, B, C 모두에게 알림이 갈 것이다.
- 구독자들은 알림을 받고 해당 동영상을 보러가는 작업을 수행할 수 있다.

#### [간단한 예시 코드]
- Observable(Youtuber)

```java
interface Youtuber {
    void registerSubscriber(Subscriber s);
    void removeSubscriber(Subscriber s);
    void notifySubscribers();
}

class Y implements Youtuber {

    // 리스트로 구독자들을 관리한다.
    private List<Subscriber> subscribers;
    private String state;

    public Y() {
        this.subscribers = new ArrayList<>();
    }

    // 상태 변화가 있을 때마다 notifySubscribers() 함수를 호출한다.
    public void setState(String state) {
        this.state = state;
        notifySubscribers();
    }

    public String getState() {
        return state;
    }

    @Override
    // 구독자 등록 함수
    public void registerSubscriber(Subscriber s) {
        subscribers.add(s);
        s.setYoutuber(this);
    }

    @Override
    // 구독자 제거 함수
    public void removeSubscriber(Subscriber s) {
        subscribers.remove(s);
    }

    @Override
    // 등록된 모든 구독자들에게 자신의 상태를 알리는 함수
    public void notifySubscribers() {
        for (Subscriber subscriber : subscribers) {
            subscriber.update(state);
        }
    }
}
```

- Observer(Subscriber)

```java
interface Subscriber {
    void update(String state);
    void setYoutuber(Youtuber youtuber);
}

class A implements Subscriber {
    private Youtuber youtuber;

    @Override
    // 유튜버의 상태 변화에 대한 알림을 받는 함수
    public void update(String state) {
        System.out.println("A 수신 : " + state);
    }

    @Override
    public void setYoutuber(Youtuber youtuber) {
        this.youtuber = youtuber;
    }

    // 구독을 취소하는 함수
    public void unsubscribe() {
        youtuber.removeSubscriber(this);
        System.out.println("A가 구독을 취소했습니다.");
    }
}

// B, C 생략
```

- 테스트 함수

```java
public static void main(String[] args) {
    Y y = new Y();

    A a = new A();
    B b = new B();
    C c = new C();

    y.registerSubscriber(a);
    y.registerSubscriber(b);
    y.registerSubscriber(c);

    y.setState("자바 코딩 직캠!");
    // A 수신 : 자바 코딩 직캠!
    // B 수신 : 자바 코딩 직캠!
    // C 수신 : 자바 코딩 직캠!

    a.unsubscribe();
    // A가 구독을 취소했습니다.

    y.setState("스프링이 좋아요? 코틀린이 좋아요?");
    // B 수신 : 스프링이 좋아요? 코틀린이 좋아요?
    // C 수신 : 스프링이 좋아요? 코틀린이 좋아요?

    b.unsubscribe();
    // B가 구독을 취소했습니다.

    y.setState("옵저버 패턴 araboza");
    // C 수신 : 옵저버 패턴 araboza

    c.unsubscribe();
    // C가 구독을 취소했습니다.

    y.setState("살아있는 구독자 분 계세요?");
    // 구독자가 없으므로 상태 변화 알림을 받은 객체가 존재하지 않음
}
```

<br>

## 누군가가 물어본다면
<div class="spotlight1" markdown="1">
옵저버 패턴은 어떤 객체의 상태 변경에 따른 다른 객체의 동작을 트리거하고 싶을 때 사용합니다.
</div>