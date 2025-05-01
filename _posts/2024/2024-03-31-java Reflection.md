---
title: java Reflection
date: 2024-03-31 17:40:00 +09:00
categories: [java]
tags:
  [
    java,
    reflection,
    리플렉션
  ]
---

## 리플렉션 (Reflection)
- JVM은 클래스 정보를 클래스 로더를 통해 읽어와서 해당 정보를 JVM 메모리에 저장한다.
- 그렇게 저장된 클래스에 대한 정보가 마치 거울에 투영된 모습과 닮아있어, **리플렉션**이라는 이름을 가지게 되었다.
- 리플렉션을 사용하면 생성자, 메서드, 필드 등 클래스에 대한 정보를 아주 자세히 알아낼 수 있다.
- 컴파일 시간이 아닌 런타임 시간에 동적으로 특정 클래스의 정보를 추출한다.

<br>

#### 사용 예시
- 대표적으로 스프링 프레임워크에서 사용하는 어노테이션이 리플렉션을 사용한 예시이다.
- 어노테이션은 그 자체로는 아무 역할도 하지 않지만, 리플렉션 덕분에 스프링에서 `@Component`, `@Bean` 과 같은 어노테이션을 사용할 수 있는 것이다.
- 이외에도 인텔리제이와 같은 IDE에서 `Getter`, `Setter`를 자동으로 생성해주는 기능도 리플렉션을 사용하여 필드 정보를 가져와 구현한다고 한다.

<br>

#### 여기서는
- 리플렉션을 사용하여 객체의 생성자, 메서드, 필드를 가져오는 예시를 보면서 정리한다.
- 다음과 같은 `Animal` 객체와 하위 클래스 `Lion` 객체를 사용한다.
```java
static class Animal {
    String name = "동물";
    int age = 5;

    private Animal() {}

    public Animal(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public void hello() {
        System.out.println("저는 " + name + "입니다.");
    }
    private int getAge() {
        return age;
    }
}
```
```java
static class Lion extends Animal {
    @Override
    public void hello() {
        System.out.println("나는 " + name + "다.");
    }

    public Lion(){
        this.name = "사자";
        this.age = 12;
    }
}
```

<br>

## Class 클래스
- 리플렉션의 핵심은 `Class` 클래스 이다.
- 어떻게 특정 클래스의 `Class` 인스턴스를 획득할 수 있을까?

<br>

## Class 객체 획득 방법
```java
Class<Animal> animal1 = Animal.class;    // 1
System.out.println(animal1);    // class org.example.Main$Animal

Animal ani = new Lion();    // Lion은 Animal을 상속받은 클래스
Class<? extends Animal> animal2 = ani.getClass();   // 2
System.out.println(animal2);    // class org.example.Main$Lion

Class<?> aClass = Class.forName("org.example.Main$Animal");
System.out.println(aClass);    // class org.example.Main$Animal
```

1. `class` 프로퍼티를 통해 획득하는 방법이다.
    - 컴파일러가 타입을 확인할 수 있기 때문에 타입 안정성이 보장된다.
    - 리플렉션을 사용하지 않고 컴파일 타임에 접근한다.
2. 인스턴스의 `getClass()` 메서드를 사용한다.
    - 인스턴스가 하위 클래스의 객체인 경우, 해당 하위 클래스의 `Class` 객체가 반환된다.
    > `<? extends Animal>` 은 반환되는 클래스가 `Animal` 의 하위 타입임을 나타내는 제네릭 타입이다.
3. `Class` 클래스의 `forName()` 정적 메서드에 FQCN(Fully Qualified Class Name) 을 전달하여 해당 경로와 대응하는 클래스에 대한 `Class` 클래스의 인스턴스를 얻는 방법이다.
    - 주어진 이름의 클래스를 찾을 수 없는 경우 `ClassNotFoundException` 이 발생한다.

> `Class`의 객체는 public 생성자가 존재하지 않아 직접 생성할 수 없다. 대신 JVM이 객체를 자동으로 생성해준다.

<br>

## getXXX(), getDeclaredXXX()
- `getXXX()`
    - 상속받은 클래스와 인터페이스를 포함하여 모든 public 요소를 가져온다.
    - ex. `getMethods()` 는 해당 클래스가 상속받거나 구현한 인터페이스에 대한 모든 **public 메서드**를 가져온다.
- `getDeclaredXXX()`
    - 상속받은 클래스와 인터페이스를 제외하고 해당 클래스에 직접 정의된 내용만 가져온다.
    - 접근 제어자와 상관없이 요소에 접근할 수 있다.
    - ex. `getDeclaredMethods()` 는 해당 클래스에만 직접 정의된 private, protected, public **메서드를 전부** 가져온다.

<br>

## Constructor 클래스
- `Class` 를 사용해서 생성자를 `Constructor` 타입으로 가져올 수 있다.
- `Constructor` 는 클래스 생성자에 대한 정보와 접근을 제공한다.

```java
Constructor<?> constructor = animal1.getDeclaredConstructor();  // 생성자 가져오기
System.out.println(constructor);    // private org.example.Main$Animal()

Object object = constructor.newInstance();  // 인스턴스 생성
System.out.println(object); // org.example.Main$Animal@8bcc55f

Animal animal = (Animal) constructor.newInstance(); // 타입 캐스팅을 사용한 객체 생성
System.out.println(animal); // org.example.Main$Animal@58644d46
```

- `getConstructor()` 를 사용하여 `Constructor` 를 획득할 수 있다.
    - 파라미터가 없으면 기본 생성자를 찾는다.
    - `getDeclaredXXX()` 를 사용했으므로 `private` 으로 설정된 기본 생성자를 가져올 수 있다.
- `newInstance()` 메서드를 사용하여 객체를 직접 생성할 수 있다.
    - 타입 캐스팅을 사용하지 않으면 `Object` 타입으로 받아와진다.

- 생성자에 파라미터가 있다면 생성자 파라미터에 대응하는 타입을 전달하면 된다.
```java
Constructor<?> allArgsConstructor = animal1.getDeclaredConstructor(String.class, int.class);
System.out.println(allArgsConstructor); // public org.example.Main$Animal(java.lang.String,int)
```

- 아래와 같이 생성자를 가져와서 생성자를 사용하듯이 객체를 생성할 수 있다.
```java
Animal ani2 = (Animal) allArgsConstructor.newInstance("호랑이", 14);
```

- 만약 생성자가 `private` 이라면 `java.lang.IllegalAccessException` 예외가 발생하므로 접근을 허용 해주는 코드가 필요하다.
```java
constructor.setAccessible(true);
Animal animal = (Animal) constructor.newInstance();
```

<br>

## Field
- `Field` 타입의 오브젝트를 획득하여 객체 필드에 직접 접근할 수 있다.

```java
Class<Animal> aClass = Animal.class;
Animal member = new Animal("호랑이", 14);

for (Field field : aClass.getDeclaredFields()) {
    field.setAccessible(true);
    String fieldInfo = field.getType() + ", " + field.getName() + " = " + field.get(member);
    System.out.println(fieldInfo);
}

/*
    class java.lang.String, name = 호랑이
    int, age = 14
 */
```

- `set()` 메서드를 사용하여, Setter 없이도 강제로 객체의 필드 값을 변경할 수도 있다.

```java
Class<Animal> aClass = Animal.class;
Animal animal = new Animal("호랑이", 14);

Field name = aClass.getDeclaredField("name");
name.setAccessible(true);
name.set(animal, "타조"); // animal 객체의 name 필드값을 타조로 변경

System.out.println("animal = " + animal);
// animal = Animal{name='타조', age=14}
```

<br>

## Method
- `Method` 타입의 오브젝트를 획득하여 객체 메서드에 직접 접근할 수 있다.
- `invoke()` 를 사용하여 메서드를 직접 호출할 수 있다.

```java
Class<Animal> aClass = Animal.class;
Animal animal = new Animal("표범", 3);

Method hello = aClass.getDeclaredMethod("hello");
hello.invoke(animal);
// 저는 동물입니다.
```

<br>

## 어노테이션 가져오기
```java
Class<Member> aClass = Member.class;

Entitiy entityAnnotation = aClass.getAnnotation(Entitiy.class);
String value = entityAnnotation.value();
System.out.println("value = " + value);
// 멤버
```

- `getAnnotation()` 메서드에 직접 어노테이션 타입을 넣어주면, 클래스에 붙어있는 어노테이션을 가져올 수 있다.
- 어노테이션이 가지고 있는 필드에도 접근할 수 있다.

<br>

## 리플렉션의 단점
- 일반적으로 메서드를 호출한다면, 컴파일 시점에 분석된 클래스를 사용하지만 리플렉션은 **런타임에 클래스를 분석**한다.
- JVM을 최적화할 수 없기 때문에 속도가 느리다.
- 런타임에 클래스를 분석하므로 타입 체크가 컴파일 타임에 불가능하다.
- 또한 객체의 추상화가 깨진다는 단점도 존재한다.

<br>

- 따라서 일반적인 웹 애플리케이션 개발자는 사실 리플렉션을 사용할 일이 거의 없다.
- 보통 라이브러리나 프레임워크를 개발할 때 사용된다.
- 정말 필요한 곳에만 리플렉션을 한정적으로 사용해야한다.

<br>

## 누군가가 물어본다면
<div class="spotlight1">
리플렉션은 구체적인 클래스 타입을 알지 못하더라도 그 클래스의 메서드, 타입, 변수들에 접근할 수 있도록 해주는 자바 API를 말합니다.
<br><br>
컴파일 시간이 아닌 실행 시간에 동적으로 특정 클래스의 정보를 추출할 수 있는 프로그래밍 기법입니다.
</div>