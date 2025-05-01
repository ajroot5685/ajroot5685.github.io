---
title: Java의 equals와 hashCode
date: 2024-03-29 13:17:00 +09:00
categories: [java]
tags:
  [
    java,
    equals,
    hashCode
  ]
---

## 객체의 값 비교
- 객체끼리의 값을 비교할 때 우리는 객체 안의 값들이 다 같으면 `true`가 되기를 기대하지만 실상은 그렇지 않다.

```java
static class Person {
    public String name;
    public int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
}

public static void main(String[] args) {
    Person a = new Person("mike", 20);
    Person b = new Person("mike", 20);

    boolean isSame = a == b;  // a == b : false

    System.out.println("a == b : " + isSame);
}
```

- 위 코드와 같이 `==` 을 통한 연산에서 필드값들이 서로 같음에도 `false` 가 출력된다.
- 객체는 클래스로부터 `new` 연산자를 통해 생성되고 생성된 객체들은 메모리 상에 각각 저장되게 된다.
- 즉, 객체 a와 b는 저장된 위치가 다르기 때문에 다른 객체라고 인식되는 것이다.

<br>

## 객체의 값 비교가 원하는대로 동작하게 하려면
- 객체의 상위 클래스 `Object` 에 정의되어있던 `equals()` 와 `hashCode()` 메서드를 수정해야 한다.

```java
public class Object {
  .
  .
  public native int hashCode();
  public boolean equals(Object obj) {
    return (this == obj);
  }
}
```

<br>

## equals() 란?
- `equals()` 메서드는 두 객체가 동등한지 비교하는데 사용된다.
- 기본 구현은 `==` 연산자와 같은 동작을 하기에 두 객체의 내용을 비교하고자 할 때는 오버라이딩을 해야 한다.

<br>

## hashCode() 란?
- 객체의 해시 코드를 반환한다.
- 해시코드는 객체를 나타내는 유일한 값이다.
- 해시 기반의 컬렉션(ex. `HashMap`, `HashSet`)이 객체를 효율적으로 저장하고 검색하는데 사용하는 정수 값이다.

<br>

## equals와 hashCode 관계
- 동일한 객체는 동일한 메모리 주소를 갖는다는 것을 의미하므로, 동일한 객체는 동일한 해시코드를 가져야 한다.
- 그렇기 때문에 `equals()` 메서드를 오버라이딩 하면, `hashCode()` 메서드도 함께 오버라이딩 해야 한다.

1. 두 객체가 `equals()` 로 비교했을 때 같다면, 두 객체의 `hashCode()` 는 반드시 같은 값을 반환해야 한다.
2. 두 객체의 해시코드가 같다고 해서 두 객체가 `equals()` 로 비교했을 때 반드시 같을 필요는 없지만, 해시코드가 다르면 두 객체는 `equals()` 로 비교했을 때 반드시 다르다고 판단된다.

  ```java
  // obj1.hashCode()와 obj2.hashCode()가 같다면
  - obj1.equals(obj2) == true // OK, equals가 true 여도 된다.
  - obj1.equals(obj2) == false  // OK, equals가 false 여도 된다.

  // obj1.hashCode()와 obj2.hashCode()가 같지 않다면
  - obj1.equals(obj2) == true // not OK, equals가 true면 안된다.
  - obj1.equals(obj2) == false // OK, equals가 false 여도 된다.
  ```

<ol start="3">
<li>
<code class="language-plaintext highlighter-rouge">equals()</code> 메서드를 오버라이딩 할 때는 항상 <code class="language-plaintext highlighter-rouge">hashCode()</code> 메서드도 함께 오버라이딩 해야 한다.
</li>
</ol>

<br>

## equals() 오버라이딩
- 위에서 정의했던 `Person` 클래스에 `equals()` 를 오버라이딩 해보자

```java
@Override
public boolean equals(Object obj) {
    if (obj == null) {
        return false;
    }
    if (obj == this) {
        return true;
    }
    if (getClass() != obj.getClass()) {
        return false;
    }

    Person p = (Person) obj;
    return this.getName().equals(p.getName()) && this.getAge() == p.getAge();
}

---

boolean isSame = a.equals(b);

System.out.println("a == b : " + isSame); // true
```

- 두 객체의 이름과 나이가 모두 같으면 true를 반환하도록 설계했다.

<br>

## hashCode() 오버라이딩이 필요한 이유
- 이제 equals에 의한 문제는 해결된 것처럼 보인다.
- 하지만 `HashSet`과 같은 자료구조에 저장하려고 하면 문제가 생긴다.

```java
Person a = new Person("mike", 20);
Person b = new Person("mike", 20);

System.out.println(a.equals(b)); // true

Set<Person> persons = new HashSet<Person>();
persons.add(a);
persons.add(b);

System.out.println(persons.size()); // 2
```

- Hash 자료구조는 중복 원소를 허용하지 않는다.
- 따라서 위 코드의 `persons` 에는 같은 객체 2개를 넣었으므로 하나만 존재해야 하지만 두 개가 존재한다.
- 각 객체가 생성될 때 서로 다른 메모리 주소에 할당되면서 `hashCode` 도 다르게 할당되었기 때문이다.

<br>

## hashCode() 오버라이딩
- 위 문제를 해결하기 위해 `hashCode()` 오버라이딩을 해보자

```java
@Override
public int hashCode() {
  return Objects.hash(name, age);
}

---

System.out.println(persons.size()); // 1
```

- `Objects.hash()` 메서드를 사용하여 name과 age 의 필드값을 기반으로 해시 코드를 생성하도록 오버라이딩했다.
- 이를 통해 Hash 자료구조에도 정상적으로 동작하는 것을 볼 수 있다.

<br>

## 누군가가 물어본다면
<div class="spotlight1">
자바에서 객체의 값 비교를 원하는대로 동작시키기 위해서는 <code class="language-plaintext highlighter-rouge">equals()</code> 메서드를 오버라이딩 해야 합니다.
<br><br>
뿐만 아니라 Hash 자료구조에서 논리적으로 알맞게 동작시키기 위해서 <code class="language-plaintext highlighter-rouge">hashCode()</code> 메서드의 오버라이딩도 필요합니다.
</div>