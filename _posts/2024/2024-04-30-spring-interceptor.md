---
title: Spring Security 이해를 위한 Spring Interceptor 예제 살펴보기
date: 2024-04-30 23:54:00 +09:00
categories: [spring, security]
tags:
  [
    spring,
    interceptor,
    AOP
  ]
---

## 이 포스팅에서는
> 스프링을 사용하여 사용자의 인증 여부를 확인하기 위해 보통 `Spring Security`를 사용합니다.
>
> 하지만 이 프레임워크는 거대하기 때문에 처음부터 이해하기는 힘듭니다.
>
> 이 포스팅에서는 비슷한 동작 방식을 가진 `스프링 인터셉터`를 통해 사용자의 인증 여부를 확인하는 간단한 예제를 알아보겠습니다.
>
>> 등장하는 여러 개념들은 깊게 다루지 않는다는 점 참고해주세요.

<br>

## AOP
- OOP를 보완하는 개념입니다. 즉, OOP를 더 잘 사용하기 위해 도입된 개념입니다.
- 어떤 클래스들의 공통 관심사를 효율적으로 처리합니다.
    - 핵심 기능과 부가 기능을 분리합니다.

<br>

#### [AOP 예시 - 시간을 측정하는 로직을 추가해야 할 때]

```java
public void join(JoinRequest joinRequest) {
    memberRepository.save(joinRequest.toMember());
}
```

- 회원가입을 하는 기능입니다.
- 이때, 회원가입 기능의 시간을 측정하는 요구사항이 들어왔다고 합시다.

- 아주 간단하게 다음과 같이 코드를 추가할 수 있을 것입니다.

```java
public void join(JoinRequest joinRequest) {
    StopWatch stopWatch = new StopWatch();
    stopWatch.start();

    try {
        memberRepository.save(joinRequest.toMember());
    } finally {
        stopWatch.stop();
        log.info("join spend {} ms", stopWatch.getLastTaskTimeMillis());
    }
}
```
- 시간을 재는 로직을 추가하여 try 구문 안에 회원가입 로직을 넣어 해결할 수 있습니다.

- 그러나 다음과 같은 문제점이 있습니다.
    1. 모든 서비스에 다 같은 형식으로 구현해달라는 요청을 받으면?
        - 서비스가 몇백개가 되는 경우 일일이 구현하는 것은 굉장히 비효율적입니다.
    2. 시간을 재는 것은 서비스에서 핵심 기능이 아닌 부가기능입니다.

<br>

#### [AOP 예시 - Spring AOP 적용]
- Spring AOP는 부가 기능을 정의하고, 타겟을 설정, 부가기능 실행하는 시점 등을 설정하여 반복되는 코드를 줄여줍니다.
- 실행 순서
    - `스톱워치 시작 - 타겟 메서드 실행 - 실행 완료 후 스톱워치 종료`

> AOP를 완전히 이해하기 위해 프록시 개념도 알아야 하지만 여기서 자세히 다루지는 않고, 아래 예시 코드 정도만 참고하시기 바랍니다.

```java
// PerformanceCheck 애노테이션을 가진 메서드를 타겟으로 정합니다.
@Around("@annotation(PerformanceCheck)")
public Object stopWatch(ProceedingJoinPoint joinPoint) throws Throwable {
    StopWatch stopWatch = new StopWatch();

    try {
        stopWatch.start();
        return joinPoint.proceed(); // 타겟 메서드 실행
    } finally {
        stopWatch.stop();
        log.info("request spent {} ms", stopWatch.getLastTaskTimeMillis());
    }
}
```

```java
@PerformanceCheck
public void join(JoinRequest joinRequest) {
    memberRepository.save(joinRequest.toMember());
}
```

<br>

## 스프링 인터셉터
<div class="spotlight2">
웹과 관련된 공통 관심사를 처리해주는 AOP 메커니즘입니다.
</div>

> 위의 Spring AOP로도 회원 인증/인가를 처리할 수는 있지만 인터셉터가 HTTP의 헤더나 URL 정보들을 편리하게 제공해주기 때문에 개발 생산성이 좋습니다.

- 스프링 인터셉터의 대상은 웹 요청과 응답으로, 이를 가로채서 로깅, 인증, 인가, 데이터 전처리 및 후처리 등의 작업을 수행합니다.
- 정확한 흐름은 다음과 같습니다.
    - `HTTP 요청 → WAS → 필터 → 서블릿 → 스프링 인터셉터 → 컨트롤러`

<br>

#### [스프링 인터셉터 인터페이스]
```java
public interface HandlerInterceptor{

	default boolean preHandle(HttpServletRequest request, HttpServletResponse response,
		Object handler) throws Exception {}

	default void postHandle(HttpServletRequest request, HttpServletResponse response,
		Object handler, @Nullable ModelAndView modelAndView) throws Exception {}

	default void afterCompletion(HttpServletRequest request, HttpServletResponse response,
		Object handler, @Nullable Exception ex) throws Exception {}
}
```

<img src="/assets/img/24/04/30/interceptor-flow.png" alt="interceptor-flow" width=700>

- 스프링은 preHandle, postHandle, afterCompletion 3가지의 메서드를 사용할 수 있습니다. 하나씩 간단하게 알아보겠습니다.

- `preHandle`
    - 컨트롤러 호출 전에 호출됩니다.
    - preHandle의 응답값이 true이면 다음(다음 인터셉터 혹은 컨트롤러)으로 진행하고, false면 더이상 진행하지 않습니다.
- `postHandle`
    - 컨트롤러 호출 후에 호출됩니다.
- `afterCompletion`
    - 뷰가 렌더링 된 이후에 호출됩니다.

<div class="spotlight1" markdown="1">
**postHandle vs afterCompletion**
- 차이는 핸들러(컨트롤러)에서 발생하는데 핸들러에서 예외가 발생하면 `postHandle` 은 호출되지 않지만, `afterCompletion` 은 호출됩니다.
</div>

<br>

## Session
- 예제에 들어가기에 앞서 회원정보는 세션으로 저장합니다.
- 세션에 대해 알고 있다면 이 부분은 건너뛰셔도 좋습니다.

<br>

#### [세션의 작동 방식]
1. 세션 생성
- 클라이언트가 웹 애플리케이션에 처음 접속할 때, 서버는 해당 클라이언트를 식별하기 위한 고유 세션 ID를 생성합니다.
- 이 ID는 서버에 저장되고, 클라이언트의 브라우저에는 이 세션 ID를 포함하는 쿠키가 전송됩니다.
- 이 쿠키는 보통 `JSESSIONID` 라는 이름을 가집니다.

2. 세션 유지
- 클라이언트가 웹 애플리케이션 내에서 다른 페이지로 이동하거나, 추가 요청을 할 때마다 브라우저는 세션 ID가 포함된 쿠키를 서버로 전송합니다.
- 서버는 이 세션 ID를 사용하여 해당 클라이언트의 세션을 찾고, 클라이언트의 상태 정보에 접근합니다.

3. 세션 만료
- 클라이언트가 일정 시간 동안 활동이 없거나, 로그아웃을 하면 세션은 서버에서 만료됩니다.
- 세션의 만료 시간은 서버 설정에 따라 다르며, 세션 만료 후 클라이언트는 다시 로그인해야 합니다.

<img src="/assets/img/24/04/30/session.png" alt="session" width=500>

<br>

#### [세션 획득 메서드]
```java
HttpSession session = request.getSession();
```

- 클라이언트의 쿠키에 해당하는 세션 ID로 서버에서 해당 세션을 얻습니다.

<br>

#### [Member 객체 획득 메서드]
```java
session.getAttribute("login");
```

- 얻은 세션에서 key 값(`login`)에 해당하는 value 를 획득합니다.
- 예제에서는 `login` key에는 `Member` 객체를 저장합니다.

<br>

#### [세션 저장소 예시]
- 이해를 돕기위한 세션 저장소의 데이터의 예시입니다.
- `Map<세션ID, Map<key, value>>` 으로 저장되어 있다고 생각하시면 됩니다.

|sessionId|Map<key, value>|
|:--:|:--:|
|yyzz...|{login : Member1}|
|zxab...|{login : Member2}|
|zxev...|{login : Member3}|

<br>

## 예제
- 저희는 요청을 낚아채서 해당 요청의 인증 정보가 올바른지 확인하겠습니다.
- 이를 위해 인터셉터의 3가지 메서드 중 `preHandle`을 사용합니다.

<br>

#### [Interceptor 생성]
- 먼저 HandlerInterceptor를 상속받는 LoginCheckInterceptor 를 만들겠습니다.

```java
@Slf4j
public class LoginCheckInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        log.info("preHandle 실행");

        HttpSession session = request.getSession();
        if (session == null | session.getAttribute(SessionConst.LOGIN_MEMBER) == null) {
            return false;
        }
        return true;
    }
}
```

- session 저장소가 없거나, 저장소에 Member 객체가 없으면 false를 반환합니다.

<br>

#### [Interceptor 등록]
- 생성한 인터셉터를 설정 파일에 등록해줍니다.

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LoginCheckInterceptor())
                .order(1)
                .addPathPatterns("/**")
                .excludePathPatterns("/", "/post/detail/**", "/signup", "/login", "/logout", "/admin/login", "/css/**", "/*.ico", "/error");
    }
}
```

- `order`
    - 여러 개의 인터셉터가 등록될 때 순서를 지정하는 메서드입니다.
    - 숫자가 낮을수록 먼저 실행됩니다.
- `addPathPatterns`
    - 인터셉터가 적용될 URL 패턴을 지정합니다.
    - 여기서는 `/**`를 지정하여 모든 URL에 이 인터셉터가 적용됩니다.
- `excludePathPatterns`
    - 인터셉터를 적용시키지 않을 URL 패턴을 지정합니다.
    - `addPathPatterns` 보다 우선권을 가집니다.

<br>

#### [인증 실패 시 응답 로직 추가]
- `preHandle` 에서 단순히 false 만 반환하면 클라이언트는 빈화면에서 멈춰버립니다.
- 따라서 클라이언트에게 루트 경로로 돌아가도록 redirect 응답을 보내겠습니다.

```java
@Slf4j
public class LoginCheckInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        log.info("preHandle 실행");

        HttpSession session = request.getSession();
        if (session == null | session.getAttribute(SessionConst.LOGIN_MEMBER) == null) {
            response.sendRedirect("/");
            return false;
        }
        return true;
    }
}
```

<br>

## 마치며
- 코드로밖에 못보여드리지만 직접 간단한 CRUD를 구축하셔서 적용해보시면 큰 도움이 되실 것 같습니다.
- 다음은 Spring Security로 회원 인증/인가를 다뤄보겠습니다.