---
title: Fixture Monkey 라이브러리로 Fixture 자동 생성하기
date: 2024-09-08 16:02:00 +09:00
categories: [backend]
tags:
  [
    backend,
    test,
    fixture monkey
  ]
---

## Fixture, Dummy Data, Mock, Monkey Testing
- 라이브러리를 소개하기 전에 Fixture와 자주 쓰이는 테스트 용어, Monkey라는 이름은 어디서 유래한건지 알아보자.

<br>

#### [Fixture]
<div class="spotlight2" markdown="1">
테스트를 위해 미리 준비된 상태나 데이터
</div>

- 테스트에서 **검증**할 핵심 데이터를 제공하며, 테스트 성공 여부를 결정하는 중요한 데이터이다.

<br>

#### [Dummy Data]
<div class="spotlight2" markdown="1">
테스트에서 필요하지만 실제 로직에는 사용되지 않는 데이터
</div>

- 테스트를 성공적으로 실행하기 위한 **최소한**의 값만 제공하며, 중요한 데이터가 아닌 단순히 자리를 채우기 위한 용도로 사용된다.

<br>

#### [Mock]
<div class="spotlight2" markdown="1">
테스트에서 특정 객체의 동작을 시뮬레이션하는 가짜 객체
</div>

- 외부 API 호출, 데이터베이스 연결과 같은 의존성을 끊고 `Mock 객체`가 대신하여 그 행동을 정의한다.

<br>

#### [Monkey Testing]
<div class="spotlight2" markdown="1">
임의의 입력을 무작위로 시스템에 넣어 예기치 않은 동작이나 오류를 발견하는 데 중점을 두는 테스트 기법
</div>

- 기법의 형태가 **원숭이가 타자기를 마구잡이로 두들기는 것**에 비유하여 `Monkey Testing`이라는 이름이 붙었다고 한다.

<br>

## 기존 테스트에서의 Fixture 구성
- 테스트를 작성하기 위해 각각의 테스트 케이스에 테스트용 객체를 일일이 구성하는 것은 매우 번거로운 일이다.
- 그래서 가장 흔한 형태로, Test Data Builder 패턴과 Object Mother 패턴으로 Fixture를 구성한다.
- 그러나 이 방식은 다음과 같은 불편한 점이 존재한다.
    1. 코드가 중복될 수 있다.
    2. 새로운 객체 상태가 필요할 때마다 새로운 메서드를 추가해야 하므로 **유연성이 떨어진다.**
    3. 직접 테스트 객체의 상태를 추가하다 보니 미처 커버하지 못하는 엣지 케이스가 발생할 수 있다.

<br>

## Fixture Monkey
<img src="/assets/img/24/09/08/fixture monkey.png" alt="fixture monkey" width="600">

[Fixture Monkey 웹페이지](https://naver.github.io/fixture-monkey/v1-0-0/)

- Fixture Monkey는 NAVER의 Platform Labs에서 `Next 페이 프로젝트`에서 활용하면서 발전시킨 테스트 라이브러리이다.
- 몇년 간의 개선 후에 2023.11.10 에 정식 1.0.0 버전이 릴리즈되었다.
- Fixture와 Monkey Testing에서 따온 Monkey를 합쳐 Fixture Monkey 라는 이름으로 탄생한 이 라이브러리는 사진처럼 여러 장점을 내세우고 있다.

<img src="/assets/img/24/09/08/homepage.png" alt="homepage" width="600">

**재사용 가능하고 복잡한 임의의 테스트 Fixture를 자동으로 생성해 주는 Java&Kotlin 라이브러리**
> 아쉽지만 스프링을 타겟으로 나온 라이브러리이다.

<br>

#### [Simplicity]
- 미리 정의된 Fixture 메서드를 통해 1줄의 코드만으로 원하는 만큼 테스트 객체를 만들 수 있다.
- 직접 정의하는 Fixture 메서드는 `Builder 패턴`을 통해 쉽게 만들어낼 수 있다.

<br>

#### [Reusability]
- 한 번의 Fixture 메서드를 정의하여 굉장히 다양한 형태의 테스트 객체를 만들어 낼 수 있어 재사용성이 높다.

<br>

#### [Randomness]
- 랜덤성이 들어가 놓칠 수 있는 엣지 케이스까지 빠짐없이 테스트 할 수 있다.

<br>

## 사용 방법
- 간단한 사용방법을 먼저 보고 초기 세팅 과정과 도입 결과를 뒤에서 다루도록 하겠다.

<br>

**build.gradle 추가**
```groovy
// Fixture Monkey
testImplementation 'com.navercorp.fixturemonkey:fixture-monkey-starter:1.0.23'
```

<br>

**객체 생성**
- 핵심 클래스인 FixtureMonkey 객체를 통해 다양한 객체를 생성할 수 있다.

```java
FixtureMonkey fixtureMonkey = FixtureMonkey.builder().build();
User user = fixtureMonkey.giveMeOne(User.class);  // User 객체 생성
```

<br>

**필드 커스터마이징**
- 객체의 특정 필드 값을 고정하거나 특정 범위로 설정할 수 있다.

```java
// age가 30인 User 객체 생성
User customUser = fixtureMonkey.giveMeBuilder(User.class)
    .set("name", "이름")
    .set("age", 30)
    .sample();

// age가 20~50 사이인 User 객체 생성
User customUser = fixtureMonkey.giveMeBuilder(User.class)
    .set("name", "이름")
    .set("age", Arbitraries.integers().between(20, 50)) // 20~50 사이의 랜덤 값 설정
    .sample();
```

<br>

**다양한 설정**
- 설정하지 않은 값은 기본적으로 `null` 로 설정된다.
- 그러나 `defaultNotNull(true)` 로 설정하면, null이 아닌 해당 필드 타입의 랜덤 값으로 설정된다.

```java
FixtureMonkey fixtureMonkey = FixtureMonkey.builder()
    .defaultNotNull(true) // 모든 필드를 기본적으로 null이 아닌 값으로 설정
    .build();

User customUser = fixtureMonkey.giveMeBuilder(User.class)
    .set("name", "이름")
    //.set("age", 30) 미설정 -> defaultNotNull(true) 이므로 랜덤 값으로 설정됨
    .sample();
```

<br>

**컬렉션 객체**
- `giveMe(Class<T> clazz, int size)` 메소드로 원하는 크기의 컬렉션 객체도 생성할 수 있다.
    > 각 객체의 값은 랜덤하게 설정된다.

```java
// 10개의 랜덤 User 객체 리스트 생성
List<User> users = fixtureMonkey.giveMe(User.class, 10);
```

<br>

**객체 그래프 생성**
- 객체가 다른 객체를 포함하는 경우에도, 자동으로 생성하여 테스트 데이터를 구성할 수 있다.
- 순환 참조가 되는 객체에 대해서도 Fixture Monkey는 이를 인식하여 **순환 참조 깊이를 제한**하는 등의 제어가 가능하다.

```java
// Order 객체와 그 안에 포함된 User, Product 등도 자동 생성
Order order = fixtureMonkey.giveMeOne(Order.class);

Order order = fixtureMonkey.giveMeBuilder(Order.class)
    .set("shippingAddress.city", "New York")  // 하위 객체의 필드도 설정 가능
    .sample();
```

```java
import com.navercorp.fixturemonkey.customizer.CircularDependencyCustomizer;

FixtureMonkey fixtureMonkey = FixtureMonkey.builder()
    .defaultCircularReferenceDepth(3)  // 순환 참조 깊이를 3으로 설정
    .build();

// 객체 순환 참조가 있는 경우 깊이 3까지만 데이터를 채움
Employee employee = fixtureMonkey.giveMeOne(Employee.class);
```

<br>

## 데이터 기본 생성 전략
- Fixture Monkey의 기본 생성 방식은 `BeanArbitraryIntrospector` 이다.

<br>

**BeanArbitraryIntrospector**
- 빈 프로퍼티 규약을 따라 리플렉션과 Setter 메서드를 사용하여 객체를 생성한다.
- 해당 클래스에 기본 생성자와 Setter가 있어야 한다.

<br>

**ConstructorPropertiesArbitraryIntrospector**
- 생성자를 이용해 객체를 생성한다.
- 해당 클래스에 생성자가 정의되어 있어야 한다.
    - ex. `@AllArgsConstructor`

<br>

**FieldReflectionArbitraryIntrospector**
- 리플렉션 방식을 이용하여 생성한다.
- 필수 조건이 없으나 final 필드에 값을 설정할 수 없고, 특정 환경이나 설정에 의해 리플렉션이 제한될 수 있다.

<br>

**BuilderArbitraryIntrospector**
- 빌더 방식을 이용하여 생성한다.
- `@Builder` 나 `builder`/`build` 로 이름을 갖는 메서드를 만들어야 한다.

<br>

**FailoverArbitraryIntrospector**
- 테스트 코드를 작성하다 보면 작성된 코드의 객체 생성 방식이 모두 달라 단일로는 생성되지 않는 경우가 발생할 수 있다.
- `FailoverArbitraryIntrospector` 는 여러 개의 생성 방식을 지정한다.

```java
public static FixtureMonkey fixtureMonkey = FixtureMonkey.builder()
    .objectIntrospector(new FailoverIntrospector(
        Arrays.asList(
            BeanArbitraryIntrospector.INSTANCE
            FieldReflectionArbitraryIntrospector.INSTANCE
            BuilderArbitraryIntrospector.INSTANCE
        ))
        )
    .build();
```

<br>

## 재사용성을 고려한 Fixture Monkey 유틸 클래스
- 공통으로 사용하는 `FixtureMonkey` 객체와 범위 설정 로직들은 `FixtureCommon` 클래스에 두고, 각 도메인에 맞는 Fixture 클래스를 만들었다.

<br>

- `.set()` 은 같은 필드에 대해 여러번 사용하더라도 가장 마지막에 넣은 설정이 적용된다.
- 따라서 `ArbitraryBuilder<Object>` 를 반환하는 기본 함수를 두고 테스트 코드에서 필요할 때마다 `.set()`으로 추가 설정을 하거나 메서드 자체를 추가할 수 있다.

```java
public class FixtureCommon {

    public static FixtureMonkey fixtureMonkey = FixtureMonkey.builder()
        .objectIntrospector(new FailoverIntrospector(
            Arrays.asList(
                BeanArbitraryIntrospector.INSTANCE
                FieldReflectionArbitraryIntrospector.INSTANCE
                BuilderArbitraryIntrospector.INSTANCE
            ))
            )
        .defaultNotNull(true)
        .build();

    /**
     * 아래는 특정 데이터 형식을 가져오기 위한 함수들이다.
     */
    public static Arbitrary<String> generateRandomEmail() {
        return Arbitraries.strings()
            .withCharRange('a','z')
            .ofLength(5)
            .map(id -> id + "@gmail.com");
    }

    public static Arbitrary<String> generateRandomPhoneNumber() {
        return Arbitraries.integers().between(1000, 9999)
            .flatMap(firstPart -> 
                Arbitraries.integers().between(1000, 9999)
                    .map(secondPart -> "010-" + firstPart + "-" + secondPart));
    }

		.
		.
}
```

```java
// 예시. Member 객체에 대한 Fixture 클래스
public class FixtureMember {

    private static ArbitraryBuilder<Member> member() {
        return fixtureMonkey.giveMeBuilder(Member.class);
    }

	public static ArbitraryBuilder<Member> memberArbitrary() {
        return member()
            .set("meIdx", null)
            .set("meUuid", UUID.randomUUID().toString()) // UUID
            .set("meHp", generateRandomPhoneNumber()) // 회원 전화번호
            .set("meLastLogin", generateRandomPastLocalDateTime()) // 마지막 로그인 시간
            .set("meIsUse", Arbitraries.integers().between(0, 1)) // 미사용, 사용
            .set("meIsVerify", Arbitraries.integers().between(0, 1)) // 인증안됨, 인증됨
            .set("meVerifyDate", generateRandomLocalDateTime()); // 인증일자
    }

    // 기본 객체
    public static Member getMember() {
        return memberArbitrary(null, null)
            .sample();
    }

    // meIsUse 설정을 덮어쓴 객체
    public static Member getMeIsUseMember(String meId, String mePassword) {
        return memberArbitrary(null, null)
            .set("meIsUse", 1)
            .set("meId", meId)
            .set("mePassword", mePassword)
            .sample();
    }

    // 객체 리스트
    public static List<Member> getMemberList(int count) {
        return memberArbitrary(null, null)
            .sampleList(count);
    }
}
```

<br>

## 변화된 테스트 코드 로직
- Fixture Monkey 라이브러리 도입을 통해 지저분했던 테스트 로직이 어떻게 바뀌었는지 보겠다.

<br>

- 기존 테스트 로직
    - 일일이 여러 개의 객체를 넣으면서 코드 길이 때문에 최소한의 개수로 데이터를 추가하였다.

```java
@Test
void search_에_성공하여_meIsUse_와_meIsVerify_가_1인_데이터들을_가져온다() {
    // given
    MemberSearchReq req = new MemberSearchReq();

    fakeMemberRepository.deleteAll();
    fakeMemberRepository.save(Member.builder()
        .meIdx(1L)
        .meId("member1")
        .meIsUse(1)
        .meIsVerify(1)
        .build());
    fakeMemberRepository.save(Member.builder()
        .meIdx(2L)
        .meId("member2")
        .meIsUse(1)
        .meIsVerify(1)
        .build());
    fakeMemberRepository.save(Member.builder()
        .meIdx(3L)
        .meId("member3")
        .meIsUse(0)
        .meIsVerify(1)
        .build());
    fakeMemberRepository.save(Member.builder()
        .meIdx(4L)
        .meId("member4")
        .meIsUse(1)
        .meIsVerify(0)
        .build());

    // when
    ListResult<MemberSearchRes> search = memberService.search(req);
    List<MemberSearchRes> list = search.getList();

    // then
    assertThat(list)
        .allMatch(res -> res.getMeIsUse() == 1 && res.getMeIsVerify() == 1);
}
```

<br>

- 변화된 테스트 로직

```java
@Test
void search_에_성공하여_meIsUse_와_meIsVerify_가_1인_데이터들을_가져온다() {
    // given
    MemberSearchReq req = validMemberSearchReq();
    req.setPaging(true);

    // 한 줄의 코드로 서로 필드값이 다른 20개의 객체리스트가 만들어진다.
    List<Member> memberList = FixtureMember.getMemberList(20);
    memberList.forEach(member -> fakeMemberRepository.save(member));

    // when
    ListResult<MemberSearchRes> search = memberService.search(req);
    List<MemberSearchRes> list = search.getList();

    // then
    assertThat(list)
        .allMatch(res -> res.getMeIsUse() == 1 && res.getMeIsVerify() == 1);
}
```

- 차이가 느껴지는가?
- 기존에는 `meIsUse==1 && meIsVerify==1` 인 리스트만 가져오는지 확인하기 위해 일일이 객체를 다르게 생성해줬습니다.
- 하지만 `FixtureMonkey`를 사용하였더니 기존 로직을 재사용하고 개수도 원하는만큼 생성할 수 있게 되었다.

<br>

## Fixture Monkey를 좀 더 활용하기
- [이전 포스트](https://ajroot5685.github.io/posts/Test-Last-Development/) 에서 **각 테스트 케이스는 해당 메서드의 정책이 된다** 라고 했다.
- 회사에서 기존 프로젝트를 유지보수 하면서 Service 로직의 이해가 어려웠던 것 말고도, 엔티티의 필드가 무슨 역할을 하는지, 어떤 값들이 들어갈 수 있는지 이해하기 힘들었다.

<br>

- 이를 토대로 Fixture 클래스의 기본 `ArbitraryBuilder` 메서드에 엔티티 필드의 정책을 넣으면 어떨까 라고 생각했다.
- 예시는 다음과 같다.

```java
public static ArbitraryBuilder<Member> memberArbitrary() {
    return member()
        .set("meIdx", null) // Member 인덱스
        .set("meId", Arbitraries.strings()) // 회원 id
        .set("mePassword", Arbitraries.strings()) // 회원 비밀번호
        .set("meUuid", UUID.randomUUID().toString()) // UUID
        .set("meName", Arbitraries.strings()) // 회원 이름
        .set("meHp", generateRandomPhoneNumber()) // 회원 전화번호 - 010-1234-5678
        .set("meLastLogin", generateRandomPastLocalDateTime()) // 마지막 로그인 시간 - 2024-01-01T00:00:00
        .set("meIsUse", Arbitraries.integers().between(0, 1)) // 0:미사용, 1:사용
        .set("meIsVerify", Arbitraries.integers().between(0, 1)) // 0:인증안됨, 1:인증됨
        .set("meVerifyDate", generateRandomLocalDateTime()) // 인증일자 - 2024-01-01T00:00:00
}
```

- 이렇게 각 필드 설정 로직 옆에 주석으로 정보를 달아놓으면, 무슨 필드인지, 어떤 값이 들어갈 수 있는지를 한눈에 볼 수 있다.

<br>

## 마치며
- 유지보수를 할 때 가장 힘들었던 것은 정책이 계속 변경되면서 제대로 정리도 안되어 있어 찾기 어려웠다는 점이었다.
- 테스트 코드를 도입하면서 '어떻게 하면 가성비 있게 테스트 코드를 넣을 수 있을까?' 라는 질문에 대해 `Fixture Monkey` 라이브러리를 접하게 되었고, 이를 컨벤션으로 정하면 높은 생산성을 기대할 수 있다고 생각한다.
- 이것저것 여러가지 시도를 통해서 불편했던 유지보수 과정을 반드시 개선해볼 것이다.