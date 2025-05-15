---
title: 이펙티브 자바
date: 2025-05-15 15:23:00 +09:00
categories: [book]
tags:
  [
    book,
    개발철학
  ]
---

## 들어가며

**이펙티브 자바**는 자바 개발자라면 반드시 읽어야 할만큼 자바를 어떻게 *효율적*으로 *올바르게* 사용할 수 있는지를 소개하는 책이다.
<br>
처음 자바를 배울 때만 하더라도 익숙하지 않은 객체지향 개념 때문에 포기하다시피 놔버렸는데 자바/스프링을 사용하는 지금, **이펙티브 자바**를 통해 다시 한번 자바를 배울 수 있었다.
<br>
이 포스팅에서는 감명 깊은(?) 아이템들을 간단하게 소개한다.

<br>

## 아이템 1. 생성자 대신 정적 팩터리 메서드를 고려하라

생성자는 여러 단점을 가진다.
- 이름이 없으므로 어떤 행동을 하는지 알기 어렵다
- 인자들이 많아지면 틀린 순서로 인자를 사용해 오류를 일으킬 수 있다
- 반드시 해당 클래스의 객체를 반환해야 한다. 하위 타입을 반환할 수 없다

정적 팩터리 메서드는 이런 단점을 극복할 수 있는 방법이다
<br>
비즈니스 로직에서의 생성 메서드를 숨겨 복잡성을 완화하기 위해 엔티티 안에다 정적 팩토리 메서드를 사용했었다.
<br>
그땐 new 키워드가 뭔가 보기 싫었고, 메서드명으로 생성하는 로직임을 명시하기 위해 썼었지만 복잡성 완화 외에도 여러 장점들이 있다는게 신기했다.

```java
@Getter
@Builder
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@AllArgsConstructor(access = AccessLevel.PRIVATE)
public class Match extends BaseEntity {

    @Id
    private String id;
    @Indexed
    private String groupId;
    private List<MatchResult> matches;

    @Override
    public void generateNewId() {
        this.id = generateFirestoreId();
    }

    // 엔티티를 생성하는 정적 팩터리 메서드
    public static Match create(String groupId, List<MatchResult> matches) {
        if (matches == null || matches.size() < 2) {
            throw new CustomException(ErrorCode.MATCH_NOT_VALID);
        }

        return Match.builder()
                .id(generateFirestoreId())
                .groupId(groupId)
                .matches(matches)
                .build();
    }
}
```

<br>

## 아이템 7. 다 쓴 객체 참조를 해제하라

보통 자바는 가비지 컬렉터가 다 알아서 해줄거라 알고 있지만, 100% 다 해주는 것은 아니다.

```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        return elements[--size]; // 반환했지만 여전히 스택에는 연결되어 있다.
    }

    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```

스택에서 pop으로 꺼낸 객체는 `다 쓴 참조`로서 더이상 쓸모없다.
<br>
하지만 스택에는 연결되어 있기 때문에 다른 객체로 덮어씌어지기 전까지는 가비지 컬렉터가 참조를 해제할 수 없다.

```java
public Object pop() {
    if (size == 0)
        throw new EmptyStackException();
    Object result = elements[--size];
    elements[size] = null; // 다 쓴 참조 해제
    return result;
}
```

따라서 명시적으로 `null` 처리를 해주어야 한다.
<br><br>
이 외에도 객체 참조를 캐시에 넣고 놔둘 때에도 문제가 생기고, 리스터/콜백에서도 문제는 발생할 수 있다.

<br>

## 아이템 24. 멤버 클래스는 되도록 static으로 만들라

중첩 클래스란 다른 클래스 안에 정의된 클래스를 말한다.
<br>
중첩 클래스는 자신을 감싼 바깥 클래스에서만 쓰여야 하며, 그 외의 쓰임새가 있다면 톱레벨 클래스로 만들어야 한다.
<br><br>
평소에도 같은 생각을 가지고 있었는데, DTO가 매우 많아지는 상황에서 DTO 파일 구조가 너무 지저분해진다는 느낌을 받았다.
<br>
응집도를 생각해서 DTO를 중첩 클래스로 구분하는 것도 괜찮은 것 같다.

<br>

## 5장 제네릭

제네릭은 자주 쓰이진 않지만, 종종 프로젝트의 구조를 개선하는 데 기여한다.
<br>
5장에서 제네릭에 대해 자세하게 소개해주기 때문에 꼭 봐야할 부분이다.

<br>

## 아이템 39. 명명 패턴보다 애너테이션을 사용하라

테스트 프레임워크인 JUnit은 버전 3까지 테스트 메서드 이름을 test로 시작하게끔 했다
<br>
이는 다음과 같은 단점들이 존재했다.

1. 오타가 나면 안 된다. test로 시작하지 않으면 해당 메서드는 무시된다
2. 올바른 프로그램 요소에서만 사용되리라 보증할 방법이 없다
  - 클래스 이름이 test로 시작하더라도 메서드 이름이 test로 시작하지 않으면 수행되지 않는다
3. 프로그램 요소를 매개변수로 전달할 마땅한 방법이 없다

애너테이션이 도입되고 나서, 이 단점들은 모두 해결되었다

<br>

## 아이템 46. 스트림에서는 부작용 없는 함수를 사용하라

`forEach` 연산은 종단 연산 중 기능이 가장 적고 가장 덜 스트림답다.
<br>
스트림 계산 결과를 보고할 때만 사용하고, 계산하는 데는 쓰지 말아야 한다.

<br>

## 아이템 61. 박싱된 기본 타입보다는 기본 타입을 사용하라

```java
public static void main(String[] args) {
		Long sum = 0L;
		for (long i = 0; i <= Integer.MAX_VALUE; i++) {
				sum += i;
		}
		System.out.println(sum);
}
```

Long과 long을 혼용해서 쓰지 말자
<br>
박싱과 언박싱이 반복해서 일어나므로 체감될 정도로 성능이 느려진다