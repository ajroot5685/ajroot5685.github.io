---
title: 
date: 2024-12-15 19:16:00 +09:00
categories: [book]
tags:
  [
    book,
    clean code
  ]
---

## 들어가며

회사에서 레거시 코드를 유지보수하면서, 잘 정리되지 않은 코드와 시간에 쫒기는 코드로 인해 프로젝트를 이해하고 제대로 버그를 고쳐나갈 수 있기까지 꼬박 한달이 걸렸다.
<br>
이 경험으로 인해, 나는 남들이 읽기 쉬운 코드를 어떻게 짤 수 있을지 고민하고 확립되지 않은 컨벤션도 세워보려는 시도를 하였다.
<br>
또 좋은 코드를 잘 작성하기 위해 유명한 **Clean Code**를 읽어보았다.
<br>
전체 내용을 정리 및 요약하지 않고 내가 느끼기에 인상깊었던 부분만 정리해서 기록한다.
<br><br>
최근 [우테코](https://ajroot5685.github.io/posts/Woowa-Precourse/)에도 참여했었는데, 여기서 강조한 클린코드가 이 책을 기반으로 요구한다는 것을 알게 되었다.
<br>
우테코에서 제공한 클린코드는 이해되지 않는 부분들도 있었는데 역시 책을 보니 더 자세한 내용과 근거를 통해 이해할 수 있었다. **축약된 글로는 확실히 온전한 정보를 전달하기 어렵다.**[^hash]

<br>

## 좋은 코드를 사수하려 노력하라

> "아니, 잠깐만요! 상사가 시키는 대로 하지 않으면 짤린다구요!" 글쎄다. 겉으로 아닌 듯 행동해도 대다수 관리자는 진실을 원한다.<br>
> ...중략<br>
> 좋은 코드를 사수하는 일은 바로 우리 프로그래머들의 책임이다.<br>
> 비유를 하나 들겠다. 자신이 의사라 가정하자. 어느 환자가 수술 전에 손을 씻지 말라고 요구한다. 시간이 너무 걸리니까. 확실히 환자는 상사다. 하지만 의사는 단호하게 거부한다. 왜? 질병과 감염의 위험은 환자보다 의사가 더 잘 아니까. 환자 말을 그대로 따르는 행동은 (범죄일 뿐만 아니라) 전문가답지 못하니까.<br>
> 프로그래머도 마찬가지다. 나쁜 코드의 위험을 이해하지 못하는 관리자 말을 그대로 따르는 행동은 전문가답지 못하다.<br>
> *p.7*

<br>

확실히 코드를 짜는 팀장급이 아닌 `관리만 하는 관리자와 우리`는 `환자와 의사`의 관계에 들어맞는다.
<br>
그대로 따르기만 한다면 ChatGPT를 사용하지, 누가 개발자를 고용하겠는가?
<br>
그들에게 짜증을 들을 각오를 하고서라도, 좋은 코드를 유지하기 위해 상대가 누구라도 들이받을 수 있는 적극성을 지녀야 한다.
<br>
이것이 코더와 프로그래머의 가장 큰 차이가 될 것이다.

<br>

## 객체지향과 절차지향 중 하나를 고집하지 말고 유연하게 사용하라

> 시스템을 구현할 때, 새로운 자료 타입을 추가하는 유연성이 필요하면, 객체가 더 적합하다. 다른 경우로 새로운 동작을 추가하는 유연성이 필요하면 자료 구조와 절차적인 코드가 더 적합하다. 우수한 소프트웨어 개발자는 편견 없이 이 사실을 이해해 직면한 문제에 최적인 해결책을 선택한다.<br>
> *p.128*

<br>

#### [절차지향 코드]

절차지향은 함수를 중심으로 동작하며 프로그램이 순차적으로 실행된다.
<br>
가장 큰 특징은 데이터와 기능이 분리되어 있어 데이터를 함수로 전달하여 처리한다는 것이다.

```java
class BankAccount {
    int balance;
}

public class ProceduralExample {

    // 잔액 입금 함수
    public static void deposit(BankAccount account, int amount) {
        account.balance += amount;
    }

    // 잔액 출금 함수
    public static void withdraw(BankAccount account, int amount) {
        if (account.balance >= amount) {
            account.balance -= amount;
        } else {
            System.out.println("잔액 부족");
        }
    }

    public static void main(String[] args) {
        BankAccount account = new BankAccount();
        account.balance = 0;

        deposit(account, 1000); // 입금
        System.out.println("현재 잔액: " + account.balance);

        withdraw(account, 500); // 출금
        System.out.println("현재 잔액: " + account.balance);
    }
}
```

#### [객체지향 코드]

객체지향은 객체를 중심으로 동작하며 기능이 객체 내부에 캡슐화된다.
<br>
객체는 상태와 동작을 함께 포함한다.

```java
class BankAccount {
    private int balance;

    // 생성자
    public BankAccount(int initialBalance) {
        this.balance = initialBalance;
    }

    // 입금 메서드
    public void deposit(int amount) {
        balance += amount;
    }

    // 출금 메서드
    public void withdraw(int amount) {
        if (balance >= amount) {
            balance -= amount;
        } else {
            System.out.println("잔액 부족");
        }
    }

    // 잔액 확인 메서드
    public int getBalance() {
        return balance;
    }
}

public class ObjectOrientedExample {
    public static void main(String[] args) {
        BankAccount account = new BankAccount(0);

        account.deposit(1000); // 입금
        System.out.println("현재 잔액: " + account.getBalance());

        account.withdraw(500); // 출금
        System.out.println("현재 잔액: " + account.getBalance());
    }
}
```

<br>

#### [절차지향이 유리한 경우]

객체지향에 익숙해진 사람이라면, 무조건 절차지향이 안좋아보인다고 느낄 것이다.
<br>
통념적으로도 절차지향은 데이터가 노출되어 안전하지 않고, 확장성이 낮고, 유지보수가 안좋다고 알려져있다.
<br>
하지만 무조건 안좋기만 할까?
<br><br>
환급 기능이 추가되어야 한다고 가정하자.
<br>
객체지향 코드에서는 객체 내부가 바뀌지만, 절차지향 코드에서는 객체 내부의 변경 없이 외부에서 메서드를 하나 추가하기만 하면 된다.
<br>
즉, 절차지향에서는 객체의 항상성이 유지된다.(물론, 새로운 상태를 추가한다면 객체지향이 더 유리하다.)

> 그렇다고 혼합되는 구조는 피해야 한다.<br>
> ex. 데이터 전달만이 목적인 DTO에 메서드 추가
{: .prompt-warning }

<br>

## 클라이언트 코드가 특수 상황을 처리하지 않도록 설계하라

> 깨끗한 코드는 읽기도 좋아야 하지만 안정성도 높아야 한다. 이 둘은 상충하는 목표가 아니다. 오류 처리를 프로그램 논리와 분리해 독자적인 사안으로 고려하면 튼튼하고 깨끗한 코드를 작성할 수 있다. 오류 처리를 프로그램 논리와 분리하면 독립적인 추론이 가능해지며 코드 유지보수성도 크게 높아진다.<br>
> p.142

<br>

#### [try-catch]

try-catch 문은 문법 자체가 클린하지 못한 코드다. 사용하는 순간 최소 5줄의 코드가 필요하게 된다.

```java
try {
  MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
  m_total += expenses.getTotal();
} catch(MealExpensesNotFound e) {
  m_total += getMealPerDiem();
}
```

<br>

논리가 따라가기 어려운 이 코드를 다음과 같이 바꿀 순 없을까?

```java
MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
m_total += expenses.getTotal();
```

<br>

가능하다. 클라이언트가 처리할 필요 없이 내부에서 예외 상황을 처리하면 된다.

```java
public class ExpenseReportDAO {

  private final MealRepository mealRepository;

  public MealExpenses getMeals(Long id) {
    return mealRepository.findById(id)
        .orElseThrow(() -> new IllegalException("객체 조회 실패")); // 또는 기본 객체를 반환
  }
}
```

이를 `특수 사례 패턴(Special Case Pattern)`이라고 한다.
<br>
이렇게 하면 클라이언트 코드가 예외적인 상황을 처리할 필요가 없어진다.

#### [null]

```java
public void registerItem(Item item) {
  if (item != null) {
    ItemRegistry registry = persistentStore.getItemRegistry();
    if (registry != null) {
      Item existing = registry.getItem(item.getId());
      if (existing.getBillingPeriod().hasRetailOwner()) {
        existing.register(item);
      }
    }
  }
}
```

null이 발생할 수 있는 모든 케이스를 고려하여 처리하는게 과연 맞는 방법일까?
<br>
메서드에서 null을 반환하는 방식도, 인수로 null을 전달하는 방식도 나쁘다.
<br>
그렇다면, 서버 코드[^server-code]에서 선제적으로 null을 처리하는 것이 낫겠다.
<br>
만약 그럴 수 없는 상황이라면, 팀 내 컨벤션으로 null은 전달하지 않도록 정하는 것이 응당 합리적이다.
<br><br>
나의 경우에는 Facade 계층을 추가하여 Service는 특수상황을 처리하여 무조건 존재하는 객체를 전달하고, Facade는 예외 처리 없이 비즈니스 로직만 수행하도록 설계해보았다.
<br>
확실히 비즈니스 로직의 줄 수가 줄어 가독성이 향상되었다.

<br>

## 학습테스트를 수행해보자

> 학습 테스트에 드는 비용은 없다. 어쨌든 API를 배워야 하므로...<br>
> 오히려 필요한 지식만 확보하는 손쉬운 방법이다. 학습 테스트는 이해도를 높여주는 정확한 실험이다.<br>
> 학습 테스트는 공짜 이상이다. 투자하는 노력보다 얻는 성과가 더 크다. 패키지 새 버전이 나온다면 학습 테스트를 돌려 차이가 있는지 확인한다.<br>
> *p.149*

<br>

외부 라이브러리를 사용할 때 학습 테스트를 사용할 것을 권장한다.
<br>
우리는 보통 라이브러리를 사용할 때, 사용법이나 필요한 인자 등이 변하지 않는다고 가정하고 사용한다.
<br>
이런 안일한 판단은 라이브러리 업데이트로 인해 갑자기 서비스 장애가 발생하고, 원인을 파악하기도 쉽지 않게 된다.
<br><br>
외부 API를 사용하기 위해 제공된 API 문서들을 보며 코드를 짰던 적이 있다. 별도의 테스트 코드 없이 직접 실행해보며 json 응답을 직접 확인했었다.
<br>
나름 잘 정리된 API 문서임에도 맞지 않는 부분들이 있었고, 하나하나 확인해가며 작업을 수행했었다.
<br>
그런데 갑자기 개편한답시고 응답 구성이 크게 달라진다면?
<br>
나는 또다시 수많은 응답필드들을 하나하나 확인해봐야만 한다.
<br>
이때 학습테스트가 있다면 확실히 작업의 소요가 덜 걸리게 될 것이다.

<br>

## 클래스와 메서드 수를 최소로 줄여라

> 중복을 제거하고, 의도를 표현하고, SRP를 준수한다는 기본적인 개념도 극단으로 치달으면 득보다 실이 많아진다. 클래스와 메서드 크기를 줄이자고 조그만 클래스와 메서드를 수없이 만드는 사례도 없지 않다. 그래서 이 규칙은 함수와 클래스 수를 가능한 줄이라고 제안한다.<br>
> p.222

<br>

SRP를 극단적으로 지키기 위해 하나의 클래스 당 하나의 함수만 가져야된다고 주장하는 사람도 봤다.
<br>
이런 코드를 봤을 때, 내가 원하는 함수가 어디있는건지 도통 찾기가 어려웠다.
<br><br>
메서드를 나눴을 때의 장점을 이해는 하지만 위치를 어디에 둬야 하는지, 메서드가 매우 많아지면 오히려 가독성이 떨어지는게 아닌지 의문이 든다.
<br>
이는 MVC 패턴을 따르는 웹 애플리케이션을 제작하기 때문에 더 의문이 드는 것 같기도 하다.

<img src="/assets/img/241215/bad.png" style="border-radius:5px" alt="bad" width="300">

이게 과연 좋은 코드일까..?

<img src="/assets/img/241215/wtf.png" style="border-radius:5px" alt="wtf" width="600">

## 마무리

어떻게 하면 더 좋은코드를 쓸 수 있을지 고민하던 와중에 좋은 도움이 되었던 책이었다.
<br>
그러나 메서드는 하나의 기능만 수행해야 하며, 최대한 쪼개는 것을 권장하는 부분에서는 **오히려 가독성이 떨어진다**고 느꼈었다.
<br>
그저 좋다고 막쓰지 말고 내 주관으로 판별할 수 있는 개발자가 되자

{% include embed/youtube.html id='th7n1rmlO4I' %}

<br>

## 각주

[^hash]: [해시 충돌](https://namu.wiki/w/%ED%95%B4%EC%8B%9C)에서 비슷한 내용이 있다.
[^server-code]: 위에서 특정 메서드를 사용하는 코드를 클라이언트 코드라고 언급했으므로, 참조당하는 코드를 서버 코드라 하겠다.