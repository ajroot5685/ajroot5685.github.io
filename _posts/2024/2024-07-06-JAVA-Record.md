---
title: 불변 데이터를 간단명료하게 정의할 수 있는 JAVA의 Record 기술
date: 2024-07-06 16:22:00 +09:00
categories: [java]
tags:
  [
    java,
    record
  ]
---

## Record란?
- 자바 14버전에 처음 등장하여 16버전에서 정식 스펙이 되었다.
- 데이터 전송 객체(DTO)나 값 객체(VO) 같은 불변 데이터를 간단하고 명확하게 정의할 수 있게 해준다.

<br>

## Record의 등장 배경
- 기존 클래스 사용에 있어서 반복적이고 귀찮은 코드인 `보일러 플레이트 코드`를 줄이기 위해 도입되었다.

<br>

#### [보일러 플레이트 코드]
- 최소한의 변경으로 여러 곳에서 재사용되며 반복적으로 비슷한 형태를 가지고 있는 코드를 말한다.
- 자바에서는 `getter`, `setter`, `equals`, `hashCode`, `toString` 등이 해당된다.

<br>

#### [기존 클래스 코드 예시]
- 아래 코드를 보면 필드가 3개밖에 없는 간단한 클래스임에도 **보일러 플레이트 코드**들로 인해 가독성이 떨어지는 것을 확인할 수 있다.

```java
public class Employee {
    private String name;
    private int age;
    private String department;

    public Employee(String name, int age, String department) {
        this.name = name;
        this.age = age;
        this.department = department;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getDepartment() {
        return department;
    }

    public void setDepartment(String department) {
        this.department = department;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Employee employee = (Employee) o;
        return age == employee.age &&
               Objects.equals(name, employee.name) &&
               Objects.equals(department, employee.department);
    }

    @Override
    public int hashCode() {
        return Objects.hash(name, age, department);
    }

    @Override
    public String toString() {
        return "Employee{" +
               "name='" + name + '\'' +
               ", age=" + age +
               ", department='" + department + '\'' +
               '}';
    }
}
```

<br>

## Record 적용 예시
- 먼저 위의 코드를 record를 사용했을 때에는 어떻게 바뀌는지 살펴보고 특징을 알아보겠다.
- class 대신 record를 사용하였고, 필드들은 인자값의 형태로 담기게 된다.
- `레코드명(헤더), {바디}` 의 구조로, 헤더에 나열되는 필드를 **컴포넌트** 라고 부른다.
- 단 1줄의 코드로 `getter`, `toString` 등의 다양한 기능들을 자동으로 사용할 수 있게 된다.

```java
public record Employee(String name, int age, String department) {}
```

<br>

## 특징

#### [1. 불변성]
- record는 불변 객체로 `abstract` 로 선언할 수 없고, 암시적으로 `final` 로 선언된다.
- 즉, 한 번 값이 정해지면 setter로 값을 변경할 수 없다.
- record 내 각 필드들은 `private final` 로 정의된다.
- 불변성이라는 특징 덕분에 `Thread-Safe` 하고, 안정성, 효율성, 간결성 등을 보장받게 되었다.

<br>

#### [2. 상속 불가]
- 불변성이라는 특징으로 인해, 상속이 불가능하다.
- 하지만 인터페이스로는 구현이 가능하다.

- 사실 상속이 불가능하다는 특징은 기술 선택에 있어 꽤 치명적이라고 할 수 있다.
- DTO를 사용할 때에는 보통 여러 개를 사용하고, 공통 필드가 생기기에 상속으로 부모 DTO 클래스를 만드는 경우가 종종 있다.
- 이러한 상황에서 record는 공통 필드를 묶기가 어렵다.
    > 인터페이스로 묶을 수는 있지만, 상대적으로 불편한 형태가 된다.

<br>

#### [3. 보일러 플레이트 코드 자동 생성]
- 위의 예시에서 볼 수 있듯 record를 사용하기만 하면 필드 정의, 생성자, `eqluals`, `hashCode`, `toString` 메서드 등을 자동으로 생성해준다.

<br>

## (번외) Lombok
- 자바의 프레임워크인 스프링을 사용해본 사람들이라면 `Lombok` 라이브러리로 충분히 해결될 수 있는 문제 아니냐는 의문이 들 수 있다.
- 실제로 lombok의 `@Data` 어노테이션을 사용하면 별도로 구현할 필요없이 간단하게 사용할 수 있다.
    > 이 어노테이션은 대부분의 보일러 플레이트 코드들을 자동으로 생성해주는 기능을 수행한다.

```java
import lombok.Data;

@Data
public class Employee {
    private String name;
    private int age;
    private String department;
}
```

<br>

## Record를 꼭 사용해야 할까?
- 위의 lombok을 보면 알 수 있듯이 사실 **보일러 플레이트 코드**를 줄이는 기술은 이미 존재한다.
- 게다가 record는 **상속을 하지 못한다**는 특징 때문에 DTO의 개수가 많아져도 공통 필드를 묶지 못한다.
- 그럼에도 record는 온전한 자바의 기술이기 때문에 자바 기술이 발전함에 따라 사용성이 증가할거라는 기대가 있으므로 자바 유저라면 알아두자.

<br>

## 누군가가 물어본다면
<div class="spotlight1" markdown="1">
`Record`는 자바16의 기술로, 변경이 적고 반복되는 코드를 줄여주는 기술입니다.
<br><br>
하지만 불변성이라는 특징으로 상속이 불가능하여, 사용하기 애매하다는 점이 있습니다.
</div>