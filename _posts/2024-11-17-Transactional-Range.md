---
title: 스프링 Transactional 애노테이션 사용 주의
date: 2024-11-17 21:09:00 +09:00
categories: [spring]
tags:
  [
    spring,
    transaction,
    osiv,
    jackson
  ]
---

## 서론

스프링의 `@Transational` 은 AOP 기술을 활용하여 개발자가 별도의 DB 커넥션 설정, 롤백 처리 등의 로직을 신경쓰지 않아도 알아서 자동으로 처리해준다.
<br>
하지만 편리함에 익숙해지고 아무렇게나 사용하다 보면 다양한 오류 상황을 겪을 수 있다.
<br>
이 글에서는 `@Transational` 작동 방식을 짧게 짚어보고, 트랜잭션 범위가 얼마나 되는지, 발생 가능한 오류에는 무엇이 있는지 살펴보겠다.

<br>

## @Transational 작동 방식

```java
protected Object invokeWithinTransaction(Method method, @Nullable Class<?> targetClass,
                                         final InvocationCallback invocation) throws Throwable {
    TransactionAttribute txAttr = getTransactionAttributeSource().getTransactionAttribute(method, targetClass);
    PlatformTransactionManager tm = determineTransactionManager(txAttr);

    TransactionStatus status = tm.getTransaction(txAttr); // 트랜잭션 시작
    try {
        Object result = invocation.proceedWithInvocation(); // 실제 비즈니스 메서드 호출
        tm.commit(status); // 커밋
        return result;
    } catch (Throwable ex) {
        tm.rollback(status); // 롤백
        throw ex;
    }
}
```
> 실제 코드는 훨씬 복잡한데, 동기 및 비동기 트랜잭션 매니저를 지원하고, 코루틴을 지원하고, 다양한 트랜잭션 전략을 지원하기 때문이다.

<br>

1. 프록시 생성
- `@Transactional`이 설정된 메서드를 호출하면, `Spring AOP` 가 동작하여 메서드 호출을 가로채는 프록시 객체가 생성된다.

    <img src="/assets/img/241117/before_proxy.png" alt="before_proxy" width="400">
    <img src="/assets/img/241117/after_proxy.png" alt="after_proxy" width="600">

2. 트랜잭션 시작
- 프록시 객체는 메서드 호출을 가로채고 `TransactionInterceptor` 를 통해 트랜잭션 관리자를 호출하여 트랜잭션을 시작한다.
    > `TransactionAspectSupport`의 `invokeWithinTransaction`에서 확인할 수 있다.

3. 비즈니스 로직 실행
- 프록시는 실제 메서드를 호출하여 비즈니스 로직을 실행한다.

4. 트랜잭션 종료
- 예외가 발생하지 않으면, 커밋을 수행한다.
- 예외가 발생하면, 롤백을 수행한다.

5. 트랜잭션 설정 해제
- 커넥션을 풀에 반환하고, 관련 리소스를 정리한다.

<br>

## 트랜잭션의 범위

그렇다면 트랜잭션이 적용되는 범위는 어디일까? 아래 예시 코드에서 어느 시점에 세션이 끊길지 생각해보자

```java
@RestController
@RequestMapping("/users")
@RequiredArgsConstructor
public class UserController {

    private final UserService userService;

    @GetMapping("/{id}")
    public User getUser(@PathVariable Long id) {
        return userService.getUserById(id);
    }
}

---

@Service
@RequiredArgsConstructor
public class UserService {

    private final UserRepository userRepository;

    public User getUserById(Long id) {
        return userRepository.findById(id).orElse(null);
    }
}

---

@Repository
public interface UserRepository extends JpaRepository<User, Long> {
}
```

<br>

- Controller에서 값을 반환받을 때 세션이 종료된다.

<br>

## 주의 - Open Session in View

`OSIV` 는 **HTTP 요청 범위**에서 세션을 열어둔다.
<br>
즉, 스프링 부트에서는 Lazy 로딩이 가능하도록 컨트롤러와 뷰 렌더링 단계에서도 세션을 유지한다는 것이다.
<br>
이는 성능에 영향을 미치므로, `properties` 설정을 통해 끄는 경우도 있다.

```yaml
spring.jpa.open-in-view=false
```

#### [오류]

`lazy loading`의 잘못된 설정과 함께 OSIV 설정에 따라 서로 다른 오류를 만날 수 있다.
<br>
어떤 오류가 발생할 수 있는지 알아보겠다.

> 엔티티의 연관 관계를 `lazy loading`으로 설정하고, **`@JsonIgnore` 미사용** 등으로 인해 **순환 참조가 발생**하고 있는 환경이라고 가정한다.

#### [OSIV - true]

기본적으로 OSIV는 `true`가 디폴트값이므로, 초보들이 순환 참조를 처리하지 않았을 때 자주 접할 수 있는 오류이다.

```json
{
  "timestamp": "2024-11-17T10:00:00.000+00:00",
  "status": 500,
  "error": "Internal Server Error",
  "message": "Infinite recursion (StackOverflowError)",
  "path": "/users/1"
}
```

위 응답은 컨트롤러가 받은 데이터를 body 데이터로 직렬화해주는 `Jackson` 라이브러리가 무한 루프를 탐지하여 `StackOverflowError` 라는 오류를 내뱉었다.
<br>
스프링을 조금 해본 사람들이라면, 오류 메세지를 통해 순환 참조 처리를 안했다는 것을 금방 깨달을 수 있을 것이다.

<br>

#### [OSIV - false]

우리 팀에서는 데이터베이스 연결 자원을 조금이라도 확보하기 위해 OSIV 설정을 껐다.
> 솔직히 성능 차이가 얼마나 나는진 모르겠다.

```json
{
  "id": 1,
  "name": "root",
  "posts": 
}{
  "timestamp": "2024-11-17T10:00:00.000+00:00",
  "status": 500,
  "error": "Internal Server Error",
  "message": "could not initialize proxy - no Session",
  "path": "/users/1"
}
```

그리고는 아까와는 다르게 **성공 응답 1개, 실패 응답 1개가 같이 왔다.**
<br>
응답 데이터를 보고 눈치챘는가?
<br>
위와 같은 기현상은 다음과 같은 절차로 발생되었다.

1. 컨트롤러가 응답 데이터를 정상적으로 받아 반환한다.
2. Jackson 라이브러리는 정상 응답을 작성하면서 lazy 객체(`posts`)를 조회한다.
3. OSIV 설정을 껐기에 세션은 종료된 상태이므로, Jackson이 조회할 수 없게된다.
4. Jackson은 정상 응답 작성에 실패하여 실패 응답을 반환한다.
5. 이전에 처리되던 정상 응답이 버퍼에 남아있는 상태로, 스프링은 이를 그대로 반환한다.

<br>

## 프록시 관련 오류

이외에도 `@Transactional` 은 프록시를 만들어서 트랜잭션 설정을 한다는 것을 놓치고 메서드를 분리하는 과정에서 오류가 발생했다.

```java
@Service
public class ServiceA {

    @Transactional
    public void outerMethodA() {
        // 외부 작업
        .innerMethod();
    }

    @Transactional
    public void innerMethodB() {
        // 트랜잭션 작업
    }
}
```

A를 호출했을 때 B의 트랜잭션이 A의 트랜잭션에 포함되어 정상 수행되는 것을 기대하지만, 실제로는 트랜잭션 연결 자체가 되지 않는다.
> `REQUIRES_NEW` 로도 새로운 트랜잭션이 생성되지 않는다.

<br>
외부에서 호출할 때에는 프록시 객체를 통해 호출되므로 트랜잭션이 정상 작동한다.
<br>
그러나 같은 클래스 내의 다른 메서드를 호출하게 되면 프록시 객체가 아닌 진짜 객체의 메서드를 호출하기 때문에 트랜잭션이 정상적으로 수행되지 않는다.
<br><br>
더욱 문제인 것은 컴파일 시점에 트랜잭션이 끊기는지 확인할 수 없기 때문에 의도와 다르게 동작할 수 있고, 실제 프로덕션 환경에서 심각한 데이터 정합성 오류로도 이어질 수 있기에 이 내용을 잘 기억하고 실수를 저지르지 않도록 주의해야 한다.