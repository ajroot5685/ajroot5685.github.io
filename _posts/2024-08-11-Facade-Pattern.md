---
title: Facade 패턴의 적용과 고찰
date: 2024-08-11 21:30:00 +09:00
categories: [design pattern]
tags:
  [
    design pattern,
    spring,
    facade
  ]
---

## 파사드 패턴 도입 이유
- 기존의 프로젝트 구조는 하나의 서비스에서 다른 여러 개의 서비스를 의존하는 구조였다.
- 이는 의존하는 서비스의 변경으로 어떤 파급 효과가 일어날지 알기 힘든 구조가 되었다.
- 따라서 `Controller` 계층과 `Service` 계층 사이에 여러 개의 서비스를 사용하는 `Facade` 계층을 추가하는 **파사드 패턴을 도입**하였다.

<br>

## 파사드 패턴이란?

<div class="spotlight1">
복잡한 서브시스템을 간단하게 사용할 수 있도록 인터페이스를 제공하는 디자인 패턴
</div>

<br>

## 목적과 패턴 도입 예시

#### [1. 클라이언트 로직 단순화]
- 여러 개의 서비스를 호출하는 로직을 파사드 클래스에 캡슐화하여 클라이언트의 로직이 단순화되고, 가독성이 향상된다.

<div style="display: flex; justify-content: space-between; align-items: center;">
    <img src="/assets/img/240811/facade1-1.png" alt="facade1-1" width="300">
    <img src="/assets/img/240811/facade1-2.png" alt="facade1-2" width="400">
</div>

<br>

> 코드는 최대한 간소화하였다.

**파사드 패턴 도입 전**
```java
public class DashBoardController {

  private final BoardCountService boardCountService;
  private final ChartService chartService;
  private final UserService userService;
  private final NotificationService notificationService;
  private final LogService logService;

  @GetMapping("/api/dashboard")
  public DashboardData getDashboardData(Long userId) {
    int boardCount = boardCountService.getBoardCount();
    ChartData chartData = null;
    UserProfile userProfile = userService.getUserProfile(userId);
    List<Notification> notifications = notificationService.getUserNotifications(userId);
    
    if (userProfile.isPremiumUser()) {
      chartData = chartService.getPremiumChartData();
    } else {
      chartData = chartService.getChartData();
    }

    logService.logAccess(userId);

    return new DashboardData(boardCount, chartData, userProfile, notifications);
  }
}
```

<br>

**파사드 패턴 도입 후**
- 컨트롤러의(클라이언트)의 책임이 분리되어 애매하게 존재하던 비즈니스 로직이 사라졌다.
- 덕분에 컨트롤러의 가독성이 향상되었다.

```java
public class DashBoardController {

  private final DashBoardFacade dashBoardFacade;

  @GetMapping("/api/dashboard")
  public ResponseEntity<DashboardData> getDashboardData(Long userId) {
    DashboardData dashboardData = dashBoardFacade.getDashboardData(userId);
    return ResponseEntity.ok(dashboardData);
  }
}

public class DashBoardFacade {

  private final BoardCountService boardCountService;
  private final ChartService chartService;
  private final UserService userService;
  private final NotificationService notificationService;
  private final LogService logService;

  public DashboardData getDashboardData(Long userId) {
    int boardCount = boardCountService.getBoardCount();
    ChartData chartData;
    UserProfile userProfile = userService.getUserProfile(userId);
    List<Notification> notifications = notificationService.getUserNotifications(userId);
    
    if (userProfile.isPremiumUser()) {
      chartData = chartService.getPremiumChartData();
    } else {
      chartData = chartService.getChartData();
    }

    logService.logAccess(userId);

    return new DashboardData(boardCount, chartData, userProfile, notifications);
  }
}
```

<br>

#### [2. 순환 참조 방지]
- 계층을 추가하여 얻는 이점으로는 가독성 뿐만 아니라 순환 참조도 방지할 수 있다.
- 여러 서비스를 호출하는 서비스를 두는 경우, 수많은 서비스 클래스들 간의 의존 관계를 쉽게 파악하기 힘들어 순환 참조를 일으키는 잘못된 설계를 할 수도 있다.
- 파사드 계층을 추가한다면, 하위 계층은 상위 계층에 의존하지 않는다는 제약을 통해 순환 참조되는 것을 방지할 수 있다.

- 아래 그림에서는 수십 개, 수백 개의 많은 서비스 클래스로 인해 순환 참조를 인식하지 못하고 잘못된 의존을 한 경우이다.

  <img src="/assets/img/240811/facade2-1.png" alt="facade2-1" width="600">

- 파사드 계층이 추가되면 의존 가능 여부를 쉽게 확인할 수 있어 순환 참조를 방지할 수 있다.

  <img src="/assets/img/240811/facade2-2.png" alt="facade2-2" width="600">

<br>

## 잘못된 사용
- 디자인 패턴은 제대로 사용한다면 가독성 좋고, 유지보수 좋은 코드가 완성되지만, 잘못된 방법으로 사용한다면 사용 안하는것만도 못한 결과가 발생한다.
- 회사 프로젝트에서 파사드 패턴을 도입하였지만 잘못된 사용으로 오히려 복잡성이 증가한 사례를 소개하겠다.

<img src="/assets/img/240811/facade3-1.png" alt="facade3-1" width="600">
- 여러 개의 서비스를 파사드가 사용하듯이, 여러 개의 파사드를 사용하는 `Workflow` 계층이 추가되었다.
- `컨트롤러 -> 서비스` 의 방식에서 `컨트롤러 -> 워크플로우 -> 파사드 -> 서비스` 로 확장된 것이다.

- 뭔가 잘못되었다는 생각이 들지 않는가?
- 이렇게 새로운 계층을 추가하면, 여러 개의 워크플로우를 사용하는 계층을 또 하나 추가해야 하는가?
- 이는 오히려 **가독성을 심각하게 떨어뜨리는 설계**가 된다.

<br>

## 고찰
- 이런 문제를 겪은 회사 동료가 도움을 요청하여 어디서부터 잘못됐는지 되짚어보았다.

<br>

#### [파사드 패턴을 사용하는 이유]
- 문제가 발생한 이유는 파사드 패턴을 **왜?** 사용하는지를 잊어버렸기 때문이다.
- 앞에서 왜 사용하는지 정리하면 다음과 같다.
  1. Controller와 같은 클라이언트 계층에서 너무 많은 서비스를 사용하고, 비즈니스 로직으로 인해 가독성이 떨어졌다.
  2. Service가 다른 Service 들을 참조할 때 인간의 오류로 순환 참조가 발생하는 것을 방지한다.

- 이유를 상기하며, 어떻게 사용해야 올바르게 사용할 수 있는지 알아보자.

<br>

#### [여러 개의 Facade를 사용하는 계층]
- 우선 이러한 확장은 제한을 둬야 한다.
- 가독성을 좋게 하기 위해 도입한 패턴이 수직 확장으로 인해 가독성을 떨어뜨리기 때문이다.

<br>

#### [추상화]
- 파사드의 목적은 클라이언트 로직을 추상화를 통해 단순화시키고, 파사드에서 구체적인 비즈니스 로직을 정의하여 가독성을 높이는 것이다.
- 즉, 인터페이스와 구현체의 관계와 비슷하다.

<br>

#### [가독성, 순환 참조 방지를 위해]
1. Facade는 다른 Facade를 참조할 수 없다는 제약이 필요하다.
  > 당연히 클라이언트인 Controller 또는 Scheduler 등은 다른 곳에서 의존하면 안된다.
2. 억지로 Facade를 재사용 하려하면 파사드 패턴을 도입한 2가지 이유가 희석된다.
- Service처럼 재사용성까지 가지려하면 너무 많은 책임을 갖게 되어 Service와 다를바 없는 계층이 되고 만다.
  > 그 결과 위와 같은 계층 증식 문제가 발생하였다.
3. 재사용성의 연장선으로 클라이언트에서 사용하는 Facade는 하나만 사용할 것을 권장한다.
- 즉, Facade 간의 중복 로직이 발생하더라도 새로운 Facade를 재정의하여 사용해야한다.
- Facade는 클라이언트 로직을 단순화하는 인터페이스 역할에 충실해야 하기 때문이다.

- 결론적으로 **Service의 재사용성은 늘리되, Facade의 재사용성은 줄여야 한다.**

<br>

## 디자인 패턴
- 아키텍처와는 다르게 디자인 패턴은 프로젝트에 미치는 영향이 적고, 최소한의 책임만 가져 응집도가 높다.
- 특정 목적을 위해 설계된 디자인 패턴에 과도한 책임을 부여한다면, 위에서 언급된 문제들을 비롯한 다양한 문제들이 발생할 것이다.
- 항상 목적이 무엇인지 상기하고, 올바른 사용법이 무엇인지를 알고 디자인 패턴을 적용하자.