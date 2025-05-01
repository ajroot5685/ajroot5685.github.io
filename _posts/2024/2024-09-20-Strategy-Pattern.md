---
title: 잘못된 전략 패턴으로 인한 학습과 고찰
date: 2024-09-20 00:04:00 +09:00
categories: [design pattern]
tags:
  [
    design pattern,
    spring,
    strategy
  ]
---

## 전략 패턴(Strategy Pattern)이란?

<div class="spotlight1" markdown="1">
실행(런타임) 중에 알고리즘 전략을 선택하여 객체 동작을 실시간으로 바꾸는 **행위(행동) 디자인 패턴**
</div>

<img src="/assets/img/24/09/20/structure.png" alt="구조" width="600">

- 전략을 사용할 Context에서 interface에 의존함으로써 전략을 쉽게 바꿔 낄 수 있다.

<br>

## 사용 예시

```java
// 추상화된 전략
interface Strategy {
    void doSomething();
}

// 전략 A
class StrategyA implements Strategy {
    public void doSomething() {}
}

// 전략 B
class StrategyB implements Strategy {
    public void doSomething() {}
}
```

```java
class Context {
    Strategy strategy;

    void setStrategy(Strategy strategy) {
        this.strategy = strategy;
    }

    void logic() {
        System.out.println("다른 추가 로직");
        this.strategy.doSomething();
    }
}
```
- 전략을 설정하고, 실행시키는 전략 관리 클래스 `Context`가 있으면 클라이언트에서는 다음과 같이 활용할 수 있다.

```java
class Client {
    public static void main(String[] args) {
        Context c = new Context();

        c.setStrategy(new StrategyA());
        c.logic();

        c.setStrategy(new StrategyB());
        c.logic(); 
    }
}
```

<br>

## 스프링에서 좀 더 깔끔하게 전략 패턴을 사용하기
- 책임에 따라 클래스들을 나눈것은 좋았다.
- 그러나 `Context`의 `logic()` 메서드는 다른 비즈니스 로직도 같이 넣자니 책임이 너무 많아져 버리고, 전략의 메서드만 호출하자니 너무 의미없는 클래스가 되는 것 같다.
- 그렇다고 `Strategy` 필드를 바깥으로 빼자니 Client가 전략 설정의 책임까지 지니게 된다. 해결할 수 없을까?

<br>

#### [스프링 DI를 이용하여 주입받기]
- 서비스 레이어에서 전략에 따른 로직을 수행하기 위해 전략 구현체를 모아놓는 `Map<String, 전략>`을 선언한다.
- 구현체들을 빈으로 등록하고 `@RequiredArgsConstructor` 등을 이용하면 **key가 구현체 빈 이름**인 구현체 클래스가 주입된다.

```java
public interface Strategy {}

// StrategyNames : static final로 정의된 상수
@Component(StrategyNames.STRATEGY_A)
public class StrategyA implements Strategy {}

@Component(StrategyNames.STRATEGY_B)
public class StrategyB implements Strategy {}
```

```java
@Service
@RequiredArgsConstructor
public class ServiceLayer {

    private final Map<String, Strategy> strategyMap;

    public void doSomething(Member member) {

        // member 필드 값에 따라 전략 설정
        String strategyKey = member.getMeType().equals("A") ? StrategyNames.STRATEGY_A : StrategyNames.STRATEGY_B;

        Strategy strategy = strategyMap.get(strategyKey);

        strategy.doSomething();
        System.out.println("다른 비즈니스 로직");
        .
        .
        strategy.doSomething2();
    }
}
```

<br>

#### [DI를 사용함으로써 얻은 이점]
1. 객체 관리 위임
- 객체 생명 주기와 스코프를 스프링 컨테이너가 관리하므로 직접 관리할 수고가 줄었다.

2. 결합도 감소
- 인터페이스를 통해 주입받으므로 직접 전략 객체를 참조할 필요가 없다.
- 기존 `Client` 코드에서는 전략 객체를 직접 참조하여 결합도가 높은 상태였다.

3. 유연성 증가
- 새로운 전략이 추가되어도 빈만 추가해주면 된다.
- 자동으로 주입된 `Map` 에서 꺼내쓰기만 하면 된다.

<br>

## 불편했던 전략 패턴
- 사실 회사에서도 여러 방식으로 전략 패턴을 쓴 것을 봤었는데 하나같이 답답하고 코드가 지저분해지는 느낌을 받았다.
- 전략 패턴을 이해하지 못한 것이라 생각하며 넘어갔었는데, 이번에 공부해보니 무엇이 잘못되었는지 깨달았다.

<br>

#### [회사 전략 패턴의 문제점]
- 가장 답답했던 것은 **중복 로직**이 발생했다는 것이다. 그것도 아주 많이.
- **중복 로직**으로 인해 관련 기능을 수정할 때마다 전략 객체 개수만큼 수정이 필요했었다.
- 뿐만 아니라 하나의 로직을 이해하려면 관련된 모든 전략 객체 로직을 봐야했는데, 이때에도 **중복 로직**이 많았기 때문이다.
- 이런 **중복 로직**으로 인해 전략 패턴을 혐오하는 단계까지 이르렀다.

<img src="/assets/img/24/09/20/omg.png" alt="omg" width="700">
- 위 그림이 뭔가 잘못되었다고 느껴지는가?
- 자주 사용하는 기본적인 구조인 3계층 구조에서 **Strategy 계층이 하나가 더 추가가 된 셈**이었다.
- 나는 괜히 계층만 하나 더 추가해서 복잡하게 만들 바에야, 컨트롤러를 분리시키는 것이 낫다고 생각했다.

<br>

#### [내가 생각하는 제약사항]
- 전략 패턴은 행동 디자인 패턴이다. 위 그림에서 저지른 잘못은 행동에 대해 전략화를 한게 아니라, **행동을 하는 객체 자체를 전략화**를 시켜버린 것이다.
    - 때문에 코드의 복잡성이 증대되었고, 유지보수 과정에 피로감이 커지게 되었다.
- **중복 로직을 포함한 행동을 전략화해서는 안되고, 행동이 달라지는 로직에 대해서만 전략화를 시켜야 한다.**

> 항상 느끼는 것이지만, 디자인 패턴은 목적과 사용방법을 잘 이해하고 올바르게 사용하는 것이 중요한 것 같다.