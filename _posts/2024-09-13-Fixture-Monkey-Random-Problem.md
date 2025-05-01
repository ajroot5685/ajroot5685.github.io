---
title: Fixture Monkey 라이브러리 - 랜덤으로 값이 할당되지 않는 문제
date: 2024-09-13 00:11:00 +09:00
categories: [backend]
tags:
  [
    backend,
    fixture monkey
  ]
---

## 서론
- [이전 글](https://ajroot5685.github.io/posts/Fixture-Monkey/)에서 객체의 필드값에 랜덤으로 값을 할당했었는데 실제로 테스트 해보니 랜덤이 아니었던 문제가 있었다.
- 이는 `sampleList()` 사용할 때 발생하였으며, 원인을 정확히 파악하지는 못했지만 라이브러리를 뜯어보며 나름의 가설을 세워보았다.
- 이전 글에 추가하려 했으나 점점 길어져서 따로 작성하게 되었다.
    > 라이브러리 뜯는게 끝이 없는 작업임을 깨닫게 되었다.

## 관련 코드

**수정 전 코드**

```java
public static String generateRandomPhoneNumber() {
    return "010-" +
        Arbitraries.integers().between(1000, 9999).sample() + "-" +
        Arbitraries.integers().between(1000, 9999).sample();
}
```

**수정 후 코드**

```java
public static Arbitrary<String> generateRandomPhoneNumber() {
    return Arbitraries.integers().between(1000, 9999)
        .flatMap(firstPart -> 
            Arbitraries.integers().between(1000, 9999)
                .map(secondPart -> "010-" + firstPart + "-" + secondPart));
}
```

**문제가 발생했던 코드**
- 관련 없는 필드들은 없애고 핵심 코드만 보겠다.

```java
public class FixtureMember {

    private static ArbitraryBuilder<Member> member() {
        return fixtureMonkey.giveMeBuilder(Member.class);
    }

	public static ArbitraryBuilder<Member> memberArbitrary() {
        return member()
            .set("meHp", generateRandomPhoneNumber()) // 회원 전화번호
    }

    public static List<Member> getMemberList(int count) {
        return memberArbitrary(null, null)
            .sampleList(count);
    }
}
```

## 반환 타입 변경
- *수정 전 코드*가 더 깔끔하고 알아보기도 쉽지만, 굳이굳이 `flatMap`, `map` 메서드를 써가며 복잡하게 `Arbitrary` 객체를 반환해야만 랜덤값이 적용되었다.
- 실제 테스트해본 결과 `sampleList()` 메서드로 여러 개의 객체 값을 생성할 때 *수정 전 코드*에서는 참조 객체가 모두 같았다.

#### [수정 전 코드로 테스트한 결과]
- 0번째 객체 `meHp`의 주소값과 1번째 객체 `meHp`의 주소값이 같다.

<br>

- 0번째 객체

<img src="/assets/img/24/09/13/random1.png" alt="random1" width="600">

- 1번째 객체

<img src="/assets/img/24/09/13/random2.png" alt="random2" width="600">

#### [수정 후 코드로 테스트한 결과]
- 0번째 객체 `meHp`의 주소값과 1번째 객체 `meHp`의 주소값이 달라졌다.

<br>

- 0번째 객체

<img src="/assets/img/24/09/13/random3.png" alt="random3" width="600">

- 1번째 객체

<img src="/assets/img/24/09/13/random4.png" alt="random4" width="600">


<br>

#### [Arbitrary 코드 뜯어보기]
- `sample()` 메서드

```java
@API(status = MAINTAINED, since = "1.3.0")
default T sample() {
    return this.sampleStream()
        .map(Optional::ofNullable)
        .findFirst()
        .orElseThrow(() -> new JqwikException("Cannot generate a value"))
        .orElse(null);
}
```

- `sampleStream()` 메서드

```java
@API(status = MAINTAINED, since = "1.3.0")
default Stream<T> sampleStream() {
    return ArbitraryFacade.implementation.sampleStream(this);
}
```

- 또한 메서드를 이렇게 설명하고 있다.
<img src="/assets/img/24/09/13/sample.png" alt="sample" width="600">

- 여기서 눈에 띄는건 값을 1000개까지 생성하면서, 클래스에 `Random` 객체를 주입받는다고 한다.
- 주입받는 시점에서 동일한 시드값을 통해 주입받으면 똑같은 값이 나올 수 있다.
- 하지만 발생한 문제는 똑같은 값이 아니라 똑같은 객체를 참조하고 있는게 문제였다.

<br>

#### [Fixture Moneky 내부 코드 뜯어보기]
- 나는 `jqwik` 라이브러리에만 집중을 한 나머지 `sampleList()` 를 놓치고 말았다.
- `sampleList()` 는 Fixture Monkey 라이브러리에서 구현한 메서드로서 아래와 같이 구현되어 있었다.
    > `ArbitraryBuilder`의 구현체인 `DefaultArbitraryBuilder` 의 일부이다.

```java
public Arbitrary<T> build() {
    ArbitraryBuilderContext buildContext = context.copy();

    return Arbitraries.ofSuppliers(() -> (T)resolveArbitrary(buildContext).combined());
}

public T sample() {
    return (T)resolveArbitrary(context).combined();
}

public Stream<T> sampleStream() {
    return this.build().sampleStream(); // 마지막의 sampleStream은 jqwik의 sampleStream이다.
}

public List<T> sampleList(int size) {
    return this.sampleStream().limit(size).collect(toList());
}
```
- 사실 `combined` 라던지, `ofSuppliers` 라던지 핵심 구현 로직은 나오지 않았으나 너무 글이 길어지기 때문에 넘어가겠다.

<br>

## 그래서 원인은?

- 보통 우리는 여러 개의 객체들을 생성할 때 for문과 같은 반복문을 이용해서 직접 생성한다.
- 그래서 `sampleList()` 또한 반복문으로 생성되며, `generateRandomPhoneNumber()` 같은 로직도 반복문을 통해 서로 다른 값들이 반복해서 생성될 것이라 생각했다.
- 하지만 반복이 아니라 `combined()`로 '무언가를 엮는구나' 라고 깨닫게 되었다.

<br>

#### [가설 1]
- 한번 `sample()`로 만들어진 ***객체***는 임시 `Arbitrary` 저장소(`ArbitraryBuilderContext`라고 추측한다.)에 보관된다.
- `sampleList()`는 `sample()` 로 매번 랜덤으로 값을 생성하는 대신에 저장되어 있는 ***객체*** 들을 꺼내씀으로써 성능을 최적화시킨다.
    > 캐시와 비슷한 메커니즘이다.

<br>

#### [가설 2]
- `sampleList()`는 반복문으로 객체 리스트를 생성한 게 아니었다.
- 때문에 한번 실행되는 `generateRandomPhoneNumber()` 도 고정된 문자열로 반환이 되었던 것이다.
- 무조건 Arbitrary 객체로 반환해야 값이 고정되지 않고 서로 다른 값을 가진 객체 리스트가 만들어지는 것이다.
- 실제로 `MonkeyManipulatorFactory`의 메서드 일부에는 다음과 같이 구현되어 있다.

```java
private NodeManipulator convertToNodeManipulator(int sequence, @Nullable Object value) {
    .
    .
    else if (value == Values.NOT_NULL) { // null 이 아닌 일반 객체인 경우
        return new NodeNullityManipulator(false);
    }
    .
    .
    else if (value instanceof Arbitrary) { // Arbitrary 객체인 경우
        // ArbitraryBuilderContext 에 등록..?
        return new NodeSetLazyManipulator<>(
            sequence,
            traverser,
            decomposedContainerValueFactory,
            LazyArbitrary.lazy(() -> ((Arbitrary<?>)value).sample()) // 이제서야 랜덤한 값을 생성?
        );
    }
    .
    .
}
```

## 마무리
- 라이브러리를 뜯으면 뜯을수록 이해는 할 수 없었지만 깔끔하게 잘 구현해놨다는 느낌은 받았다.
- 시간복잡도 `n`에서 어떻게든 `log n` 으로 성능을 최적화해내는 실력이 경이롭다.
    > 역시 네이버인가?