---
title: 팩토리 패턴의 종류와 스프링이 사용하는 팩토리 패턴
date: 2024-06-10 19:22:00 +09:00
categories: [design pattern]
tags:
  [
    design pattern,
    spring
  ]
---

## 팩토리 패턴(Factory Pattern)
- 팩토리 패턴은 객체 생성 로직을 별도의 클래스나 메서드로 분리하는 패턴이다.
- 객체 생성 로직을 숨기고, 로직의 변경이 클라이언트에 영향을 주지 않아 코드를 유연하고 확장 가능하게 한다.
- 팩토리 패턴에는 크게 3가지로, `단순 팩토리 패턴`, `팩토리 메서드 패턴`, `추상 팩토리 패턴`이 있다.

<br>

## 단순 팩토리 패턴(Simple Factory Pattern)
- 객체를 생성하는 팩토리 클래스를 만든다.
- 이름 그대로 단순하게 객체를 생성하는 로직을 분리시킨 것이다.

<br>

#### [장점]
- 매우 간단하고 직관적이다.
- 객체 생성 로직을 캡슐화했기에 어느정도의 가독성과 유지보수성을 챙길 수 있다.

<br>

#### [단점]
- 새로운 객체 유형을 추가하거나 생성 로직을 수정할 때마다 팩토리 클래스의 코드가 변경된다.
- 이는 좋은 객체지향 설계원칙의 `RIP`, `OCP`를 위배하게 된다.

<br>

#### [예시]
- `Animal` 객체를 `new` 연산자가 아니라 `SimpleFactory` 클래스의 `createAnimal` 메서드를 통해 생성한다.

```java
public class SimpleFactory {
    public static Animal createAnimal(String type) {
        if (type.equals("Dog")) {
            return new Dog();
        } else if (type.equals("Cat")) {
            return new Cat();
        }
        return null;
    }
}

public interface Animal {
    void speak();
}

public class Dog implements Animal {
    @Override
    public void speak() {
        System.out.println("멍");
    }
}

public class Cat implements Animal {
    @Override
    public void speak() {
        System.out.println("냐옹");
    }
}

public class Main {
    public static void main(String[] args) {
        Animal dog = SimpleFactory.createAnimal("Dog");
        dog.speak();

        Animal cat = SimpleFactory.createAnimal("Cat");
        cat.speak();

        // 멍
        // 냐옹
    }
}
```

<br>

## 팩토리 메서드 패턴(Factory Method Pattern)
- **단순 팩토리 패턴**에서는 팩토리 클래스 안에 객체를 어떻게 만들지 정의했었다.
- **팩토리 메서드 패턴**에서는 객체 생성의 책임을 서브클래스에 위임한다.
- 즉, 팩토리 메서드는 추상 클래스나 인터페이스로 정의되고, 서브클래스(or 구현체)에서 구체적인 객체 생성 로직을 정의한다.

<br>

#### [장점]
- 새로운 객체 유형을 추가할 때 기존 코드를 수정하지 않고 새로운 서브클래스를 추가하면 된다.
- 클라이언트 코드가 구현체가 아닌 인터페이스나 추상 클래스에 의존하여 **결합도가 낮아진다.**
- 따라서 `RIP`, `OCP`, `DIP` 등을 준수할 수 있게 된다.

<br>

#### [단점]
- 단순 팩토리 패턴에 비해 클래스가 많아져 복잡해진다.
- 이에 따른 설계 작업도 늘어난다.

<br>

#### [예시]
- `AnimalFactory`의 책임을 `DogFactory`, `CatFactory`가 나눠가진다.
- 추가 객체 유형이 생길 때에도 `AnimalFactory`의 코드는 유지된다.

```java
public abstract class AnimalFactory {
    public abstract Animal createAnimal();
}

public class DogFactory extends AnimalFactory {
    @Override
    public Animal createAnimal() {
        return new Dog();
    }
}

public class CatFactory extends AnimalFactory {
    @Override
    public Animal createAnimal() {
        return new Cat();
    }
}

public interface Animal {
    void speak();
}

public class Dog implements Animal {
    @Override
    public void speak() {
        System.out.println("멍");
    }
}

public class Cat implements Animal {
    @Override
    public void speak() {
        System.out.println("냐옹");
    }
}

public class Main {
    public static void main(String[] args) {
        AnimalFactory dogFactory = new DogFactory();
        Animal dog = dogFactory.createAnimal();
        dog.speak();

        AnimalFactory catFactory = new CatFactory();
        Animal cat = catFactory.createAnimal();
        cat.speak();

        // 멍
        // 냐옹
    }
}
```

<br>

## 추상 팩토리 패턴(Abstract Factory Pattern)
- **팩토리 메서드 패턴**은 한 종류의 객체를 생성하는데 집중한 반면, **추상 팩토리 패턴**은 여러 종류의 객체를 생성하는 데 사용한다.
- 즉, 여러 공장들을 모은 `공장 단지`라고 볼 수 있다.
- 유지보수성을 위해 되도록이면 연관관계 혹은 의존관계가 있는 공장들을 모아놓는 것이 좋다.

<br>

#### [장점]
- 서로 관련 있는 객체들을 그룹으로 묶어 **일관성을 유지**할 수 있다.
- 팩토리 메서드 패턴의 장점들을 모두 포함한다.

<br>

#### [단점]
- 설계와 구현이 **가장 복잡**하여 유지보수 오버헤드가 증가한다.

<br>

#### [예시]
- 서로 다른 객체지만 연관관계가 존재하는 `Animal`과 `Habitat`을 그룹으로 묶어 생성 일관성을 유지한다.
- 그러나 코드의 양이 배로 늘어났다.

```java
public interface Animal {
    void speak();
}

public interface Habitat {
    void describe();
}

public class Lion implements Animal {
    @Override
    public void speak() {
        System.out.println("크엉");
    }
}

public class Shark implements Animal {
    @Override
    public void speak() {
        System.out.println("첨벙");
    }
}

public class Savannah implements Habitat {
    @Override
    public void describe() {
        System.out.println("사자가 서식하는 사바나입니다.");
    }
}

public class Ocean implements Habitat {
    @Override
    public void describe() {
        System.out.println("상어가 서식하는 바다입니다.");
    }
}

public interface AbstractFactory {
    Animal createAnimal();
    Habitat createHabitat();
}

public class LandAnimalFactory implements AbstractFactory {
    @Override
    public Animal createAnimal() {
        return new Lion();
    }

    @Override
    public Habitat createHabitat() {
        return new Savannah();
    }
}

public class WaterAnimalFactory implements AbstractFactory {
    @Override
    public Animal createAnimal() {
        return new Shark();
    }

    @Override
    public Habitat createHabitat() {
        return new Ocean();
    }
}

public class Main {
    public static void main(String[] args) {
        AbstractFactory landAnimalFactory = new LandAnimalFactory();
        Animal lion = landAnimalFactory.createAnimal();
        lion.speak();
        Habitat savannah = landAnimalFactory.createHabitat();
        savannah.describe();

        AbstractFactory waterAnimalFactory = new WaterAnimalFactory();
        Animal shark = waterAnimalFactory.createAnimal();
        shark.speak();
        Habitat ocean = waterAnimalFactory.createHabitat();
        ocean.describe();

        // 크엉
        // 사자가 서식하는 사바나입니다.
        // 첨벙
        // 상어가 서식하는 바다입니다.
    }
}
```

<br>

## 스프링에서 사용하는 팩토리 패턴
- 스프링은 등록된 빈들을 싱글톤 패턴으로 관리한다.
    > 각 빈은 스프링 컨테이너 내에서 단 하나의 인스턴스로 존재한다.
- 싱글톤 패턴은 DIP, OCP 등을 위반하여 안티 패턴으로 불리기도 한다.
- 스프링은 이러한 문제를 팩토리 패턴과 애노테이션으로 해결을 했다.

<br>

#### [객체 생성]
- 스프링에서는 별도의 객체 생성 로직이 존재하지 않고 `@Bean`이나 `@Component` 애노테이션을 통해 자동으로 스프링 컨테이너 안에 등록이 된다.
- 스프링 컨테이너가 주체가 되어 빈 객체를 생성한다.

```java
@Configuration
public class AppConfig {
    @Bean
    public MyService myService() {
        return new MyServiceImpl();
    }
}
```

<br>

#### [객체 조회]
- 스프링 컨테이너 내에서 관리되는 싱글톤 빈들은 `ApplicationContext` 나 `BeanFactory` 를 통해 요청할 때마다 동일한 인스턴스를 반환한다.
- 즉, 팩토리를 통해 빈 객체를 가져오는 팩토리 패턴이 적용되었다.

<br>

#### [싱글톤 패턴의 문제를 해결]
- 이렇게 객체 생성 로직을 추상화하고, 객체의 관리를 스프링 컨테이너가 전담하면서, DIP, OCP 등의 문제들이 해결되었다.
- 스프링 컨테이너라는 팩토리가 객체지향 설계를 위한 핵심 역할을 수행하면서 스프링이라는 프레임워크가 각광받게 되었다고 볼 수 있다.

<br>

## 누군가가 물어본다면
<div class="spotlight1" markdown="1">
팩토리 패턴은 객체의 생성 로직을 분리하여 코드를 유연하고 확장 가능하게 합니다.
<br><br>
팩토리 메서드 패턴과 추상 팩토리 패턴에서는 객체 생성의 책임을 서브클래스로 위임하여 `RIP`, `OCP`, `DIP`를 지키게 되었습니다.
<br><br>
**스프링**에서는 빈 객체들을 기본적으로 싱글톤 패턴으로 관리함과 동시에 팩토리 패턴을 적용하여 객체지향 설계원칙을 지키도록 설계가 되었습니다.
</div>