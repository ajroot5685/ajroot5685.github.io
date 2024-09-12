---
title: 시간이 촉박한 환경에서 가성비 있는 테스트 도입하기
date: 2024-08-30 23:28:00 +09:00
categories: [backend]
tags:
  [
    backend,
    test,
  ]
---

## TDD는 무엇인가?
- TLD를 논하기 전에 TDD가 무엇인지 간단하게 살펴보자.

<img src="/assets/img/240830/tdd.png" alt="tdd" width="600">

- `Test-Driven Development`는
    1. RED - 테스트를 먼저 작성한 뒤
    2. GREEN - 테스트를 통과하는 최소한의 코드를 작성하고
    3. REFACTOR - 작성한 로직을 개선하는 방법론이다.
- 기존에 로직부터 짜고 봤던 대부분의 사람들은 이러한 방법에 익숙하지 않다.
- 하지만 TDD에 익숙해진다면 초기 단계에서 버그를 빠르게 발견하고 최종적인 개발 시간이 줄어드는 효과를 볼 수 있다.

<br>

#### [TDD의 문제점]
- TDD는 빠른 시간 내에 개발하는 것을 요구하는 `eXtreme Programming`의 실천 방안 중 하나인만큼 생산성 증대의 효과를 맛볼 수 있다.
- 하지만 이 말에 혹해 많은 사람들이 TDD를 잘못 알고 사용하여 이도저도 아닌 상황이 발생한다.
- 그 이유는 다음과 같다.

<br>

1. 테스트를 먼저 작성하는 방식은 익숙하지 않다.
- 개발을 배우는 초기 단계에서는 프로젝트의 규모도 크지 않은 경우가 많아 수동 테스트를 진행하는 경우가 많고 불편함도 크게 느끼지 못한다.
- 이렇게 비즈니스 로직을 먼저 작성하는 습관에 길들여져 TDD를 처음 도입하면 **개발 시간이 최소 30% 증가**되는 부작용이 발생할 수 있다.

2. 가장 중요한 단계는 `REFACTOR` 단계다.
- 개인적으로 리팩토링하는 과정이 가장 중요하다고 생각한다.
- RED, GREEN 단계에서 최소한의 시간을 할당하고 REFACTOR 단계에서 공을 들여 코드를 고치고, 필요하다면 구조도 개선하는 작업이 동반되어야 한다.
- 하지만 많은 사람들은 시간에 쫒겨 REFACTOR 단계를 경시하는 경향이 있는 것 같다.

<br>

## 테스트가 없을 때의 문제점
- 테스트는 반드시 필요한가? 테스트가 왜 필요한지 생각해보자

#### [기존 방식의 문제점]
1. 구현/수정 여부를 **직접 확인**해야 한다.
- 수동으로 내가 만들고 수정한 기능이 문제가 없는지 확인한다.
- 가장 확실하고 안전한 방법이지만 귀찮고 시간이 오래 걸린다.

2. 수정으로 이어지는 **추가적인 버그 발생**을 완벽히 확인할 수 없다.
- 복잡성이 점점 증가함에 따라 어떤 기능을 수정했을 때 미처 생각하지 못한 부분에서 오류가 발생할 수 있다.
- 1번과 이어지는 내용으로, 테스트 로직이 필요한 이유가 된다.

3. 기존 코드를 이해하기 힘들다.
- 유지보수의 관점에서 한창 진행되던 프로젝트를 처음부터 이해하기는 힘들다.
- 작성되어 있는 테스트 코드를 통해 비즈니스 로직의 이해를 쉽게 할 수 있다.

4. 설계의 질이 떨어진다.
- 기능 구현에만 급급했던 결과, 중복 로직이 발생하고 **좋지 않은 설계 형태**가 만들어진다.
- 테스트 코드들이 추가된다면, 안정성 뿐만 아니라 테스트를 잘 작성하기 위한 좋은 설계를 하도록 유도된다.

<br>

#### [우리 회사의 특징]
- 파견은 없지만 SI 형태로서 짧은 기간 안에 프로젝트를 끝내야 한다. 한번에 여러 프로젝트를 진행하는 경우도 파다하다.
- 시간이 없는 것은 개발 부서 뿐만 아니라, 기획 부서에도 마찬가지여서 요구사항이 수시로 바뀌는 경우가 많다.
    > 수시로 바뀌는 요구사항은 개발팀 전체에게 공유되지 않고, 소수만 알고 넘어가게 된다.

<br>

- 🧨 시간이 촉박한 상황에서 TDD를 잘못 도입하거나, 심지어 테스트 자체를 도입하는 데에도 개발시간이 매우 길어질 수 있다.

<br>

## 절충안
- 그렇다고 테스트 없이 프로젝트를 진행하기엔 끝이 없던 유지보수의 기억이 떠나지 않는다.. 방법을 찾아보자

<br>

#### [TDD가 아닌 TLD 도입]
- 나는 TLD를 회사에 도입해보고자 한다.
- TLD는 `Test-Last-Development`로, 기존의 방식처럼 로직을 먼저 짜고 그에 해당하는 테스트들을 추가하는 방법론이다.
- 개발 방식의 변화로 개발 시간의 증가로 이어질 수 있는 TDD 도입 대신 기존의 개발 방식을 유지하며 테스트를 추가하는 TLD를 선택했다.

<br>

#### [Service 레이어에 대한 테스트만 진행]
- Controller는 웹 요청, 응답에 대한 책임만 존재하고 Repository는 JPA에 종속적이다.
- 이미 대부분의 처리를 `Spring MVC`와 `Spring Data JPA` 가 맡고 있고, 심지어는 대부분의 오류가 컴파일 단계에서 잡힌다.
- 때문에 서비스의 핵심 비즈니스 로직이 집중되어 있는 Service 레이어만 테스트를 진행해도 서비스의 80%를 커버한다고 생각한다.
- 자취생의 입장에서 한정된 돈으로 가장 가성비 있게 식사를 해야하는 것처럼, 시간이 촉박한 상황에서 가장 가성비 있게 테스트를 수행해야 한다.

<br>

#### [단위(소형) 테스트만 수행]

<img src="/assets/img/240830/test.png" alt="test" width="600">

> 실제 UI를 통해 기능을 테스트하는 UI 테스트는 논외로 하겠다.

- 통합 테스트는 서로 다른 모듈이나 계층 간 테스트가 수행된다.
- 때문에 더 많은 의존성을 띄고 특히 데이터베이스가 포함된 경우 커넥션 작업 등에 의해 테스트의 시간이 매우 증가한다.
- 더 넓은 범위의 오류를 발견할 수 있으나, 테스트 실행 시간이나 테스트 작성 시간이 크기 때문에 가성비가 좋지 않다고 할 수 있다.
- 따라서 단위 테스트만 수행해야 한다고 생각했다.

<br>

## 최종 개발 프로세스
- 위 절충안을 적용한 개발 프로세스는 다음과 같다.

1. 기존처럼 Controller, Service, Repository 에 대한 실제 로직을 먼저 작성한다.
2. `Service`의 의존성을 끊기 위한 **Mock 설정**, **Fake 클래스 작성**, 더미 데이터 등을 넣는다.
3. 짜여진 로직들을 바탕으로 테스트 케이스들을 작성하고, 위의 예시와 동일하게 `Green-Refactor 단계`를 수행한다.
4. 모든 테스트가 통과한다면 마지막으로 간단하게 API가 작동되는지 수동 테스트를 진행한다.

<br>

## 반드시 필요한 제약사항
- 항상 초심을 잃으면 더 안좋은 결과가 발생한다.
- 게다가 최대한 가성비 있게 테스트를 도입하는 것이므로, 최소한 이것들만은 꼭 지켜야 테스트의 의미를 잃지 않는다고 생각한다.

<br>

#### [100% 커버리지 달성을 강제한다.]

<img src="/assets/img/240830/coverage.png" alt="coverage" width="600">

- 100%가 정답인 것은 아니고, 불필요한 테스트도 여럿 추가될 수 있지만 100%로 강제하지 않으면 점점 안일해지고 커버리지가 낮아진다.
- 따라서 여지를 주는 것 보다 100%로 강제하는 것이 좋다고 생각했다.
> 위 이미지는 100%를 채우지 않았지만, 반드시 100%로 채우자

<br>

#### [테스트는 주석을 적극 이용한다.]
- 위에서 기획이 수시로 바뀌고 수정된 내용이 공유가 잘 안된다고 언급했었다.
- 그렇다면 테스트 케이스 하나를 정책으로 두고 그에 대한 내용을 주석으로 자세하게 적어놓으면 어떻까?
- 개발자는 항상 코드를 봐야하므로 정책서를 매번 보면서 수정 내용을 찾기 힘들다.
- 하지만 코드에 내용이 적혀 있으면, 굳이 정책서의 특정 기능을 찾아갈 필요도 없고 **깃 버전 관리에 따른 업데이트 내용이 바로 파악**되므로 훨씬 편하게 수정사항을 알아차릴 수 있을 것이다.
- `테스트 케이스들은 곧 정책이 된다.`

<br>

#### [반드시 Refactor 단계를 거친다.]
- Refactor 단계를 통해 처음 본 사람들도 빠르게 이해할 수 있는 좋은 코드가 되고, 장기적으로 디버깅 시간이 감소하게 된다.
- 테스트 작성이 어렵다면 실제 코드의 리팩토링이 필요하다는 것이다.
  - 사실상 테스트 작성의 핵심이다.
  - 아무리 테스트를 잘 작성해봤자 코드가 엉망이여서 오류 찾기가 쉽지 않다면, 그것은 테스트의 의미가 없어진다.
  - 반드시 리팩토링 과정을 거쳐야 한다.
> 리팩토링을 하면서 좋은 코드를 작성하는 법을 익히니 더 좋지 않은가?

<br>

## 마무리
- 사실 회사에 테스트를 도입하기 위해 쭉 생각한 것들을 한번 더 정리한 것인데도 뭔가 난잡한 것 같다.
- 그만큼 테스트를 도입하는 것은 간단한 일이 아니기 때문일 것이다.
- 앞으로도 계속 고민하면서 최대한 가성비 있는 테스트를 어떻게 짤 수 있을지 고민해 볼 예정이다.