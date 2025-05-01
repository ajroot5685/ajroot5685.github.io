---
title: 객체 생성을 유연하게 만들어주는 빌더 패턴
date: 2024-06-29 19:26:00 +09:00
categories: [design pattern]
tags:
  [
    design pattern,
    spring
  ]
---

## 빌더 패턴(Builder Pattern)
- `빌더 패턴`은 복잡한 객체를 단계별로 생성할 수 있도록 하는 생성 패턴(`Cretional Pattern`)이다.
- 객체의 생성 과정을 유연하게 만들어줘 다양한 표현으로 객체를 생성할 수 있게 해준다.
- 즉, 객체의 생성 과정을 **가독성이 좋은 형태**로 만들어준다.

> [객체의 생성 책임을 분리하는 팩토리 패턴](https://ajroot5685.github.io/posts/Factory-Pattern-and-Spring/)과 다르다는 것에 유의하자.

<br>

## 장단점

#### [장점]
- 생성 로직이 캡슐화되어 **쉽게 객체 생성이 가능**하다.
- 각 단계들이 명확하게 분리되어 **객체 생성의 일관성이 보장**된다.
- 동일한 빌더로 **서로 다른 구성의 객체**를 만들 수 있어 유연하다.
    > ex. a, b, c 필드를 가진 객체를 만들 수도, a, c 필드를 가진 객체를 만들 수도 있다.

<br>

#### [단점]
- 여느 디자인 패턴처럼 복잡성이 증가한다.
- 여러 단계를 거치므로 생성자보다는 성능이 떨어진다.
- 빌더 클래스를 잘못 수정하면 전체 객체 생성 로직에 영향을 줄 수 있다.

<br>

## 기존 방식의 문제점

```java
public class Product {
    private String partA;
    private String partB;
    private String partC;

    public Product(String a) {
        this.partA = a;
    }

    public Product(String a, String b) {
        this.partA = a;
        this.partB = b;
    }

    public Product(String a, String b, String c) {
        this.partA = a;
        this.partB = b;
        this.partC = c;
    }
}

public static void main(String[] args) {
    Product product1 = new Product("A");
    Product product2 = new Product("A", "B");
    Product product3 = new Product("A", "B", "C");

    // 만약 필드가 10개가 넘어간다면..
    Product complexProduct = new Product("A", "B", "C", 0, 4, 3, 5, 0, 0, null, "complete", true);
}
```

1. 경우에 따라 생성자를 계속 만들어야 한다.
    - 항상 객체의 모든 필드를 사용해서 생성하지 않으므로 다른 조합으로 객체 생성이 필요할 때마다 생성자를 만들어줘야 한다.
    - 또는 사용하지 않는 필드에 `0`이나 `null`을 넣어주어야 할 것이다.
2. 필드가 많을수록 순서가 헷갈릴 수 있다.
    - 생성자에 명시된 인자들의 순서가 지켜줘야 원하는대로 객체가 생성된다.
    - 때문에 객체를 생성할 때도 순서를 착각하여 쉽게 알아차리기 힘든 논리적인 오류가 발생할 수 있다.

<br>

## setter를 사용하는 경우

```java
public class Product {
    private String partA;
    private String partB;
    private String partC;

    public Product() {}

    public void setPartA(String partA) {
        this.partA = partA;
    }

    public void setPartB(String partB) {
        this.partB = partB;
    }

    public void setPartC(String partC) {
        this.partC = partC;
    }
}

public static void main(String[] args) {
    Product product = new Product();
    product.setPartA("A");
    product.setPartB("B");
    product.setPartC("C");

    Product product2 = new Product();
    product2.setPartA("A");
    product2.setPartC("C");
}
```

- `setter` 를 사용하면 필드 당 하나씩만 만들면 되고, setter 메서드의 이름이 다르므로 순서도 상관없으며, 내가 사용하는 필드만 세팅하면 된다.
- 하지만 setter는 **객체가 완전히 설정되지 않은 상태에서 사용될 수 있다**는 단점도 있지만 더욱 큰 단점이 있다. 바로 불변성이 깨진다는 것이다.

<br>

#### [불변성 유지의 중요성]
- 불변 객체는 상태가 변하지 않으므로 상태 변화에 따른 **side effect**(부작용)를 걱정하지 않아도 된다.
- 또한 여러 쓰레드에서 동시에 접근해도 매번 같은 값을 읽으므로 `Thread-Safety` 이다.

<br>

#### [올바른 Setter 사용]
- 실제로는 불변 객체보다 가변 객체를 쓸 일이 더 많을 것이다.
- 따라서 setter를 아예 쓰지 말자는 것이 아니라, **객체를 변화할 수 있는 방법을 최소화**하는 것이 가장 중요하다.
> 글이 길어지므로, 여기서 끊고 나중에 좀 더 자세히 다루겠다.

<br>

## 빌더 패턴 적용 - 순수 자바 코드

```java
public class Product {
    private String partA;
    private String partB;
    private String partC;

    private Product(Builder builder) {
        this.partA = builder.partA;
        this.partB = builder.partB;
        this.partC = builder.partC;
    }

    public static class Builder {
        private String partA;
        private String partB;
        private String partC;

        public Builder partA(String partA) {
            this.partA = partA;
            return this;
        }

        public Builder partB(String partB) {
            this.partB = partB;
            return this;
        }

        public Builder partC(String partC) {
            this.partC = partC;
            return this;
        }

        public Product build() {
            return new Product(this);
        }
    }
}

public static void main(String[] args) {
    Product product = new Product.Builder()
        .partA("A")
        .partB("B")
        .partC("C")
        .build();

    Product product = new Product.Builder()
        .partA("A")
        .partC("C")
        .build();
}
```
- 필드와 생성자가 `private`으로 설정되어 있으므로 객체 안의 `Builder` 클래스만을 통해 객체를 생성할 수 있다.
- 메서드명을 필드명과 같게 설정하여 헷갈릴 일이 없다. 즉, 가독성이 좋아졌다.
- 사용하고자 하는 필드만 설정할 수도 있고, 순서가 바뀌어도 상관없다.

- 다만, 빌더 클래스가 생성됨에 따라 객체를 정의하는 코드가 지저분해졌다.
- 이는 `Lombok` 라이브러리를 사용하여 더 깔끔하게 압축이 가능하다.

<br>

## 빌더 패턴 적용 - Lombok 라이브러리

```java
@Builder
public class Product {
    private String partA;
    private String partB;
    private String partC;
}

public static void main(String[] args) {
    Product product1 = Product.builder()
        .partA("A")
        .partB("B")
        .partC("C")
        .build();

    Product product2 = Product.builder()
        .partA("A")
        .partC("C")
        .build();

    System.out.println(product1);
    System.out.println(product2);
}
```
- 순수 자바 코드로 구현했던 빌더 패턴이 바로 롬복 라이브러리의 `@Builder` 어노테이션이다.
- 라이브러리를 통해서 디자인 패턴의 고질적인 문제인 **복잡성 증가**도 해결이 되었다.

> 앞으로도 코드를 어떻게 해야 잘 짤수 있을지 고민하면서, 혁명적인 라이브러리 사용으로 깔끔한 코드 작성을 해보자.

<br>

## 누군가가 물어본다면
<div class="spotlight1" markdown="1">
빌더 패턴은 객체의 생성 과정을 유연하고 가독성이 좋게 만들어주는 생성 패턴입니다.
</div>