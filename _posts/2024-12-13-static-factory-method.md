---
title: 생성자 대신 정적 팩터리 메서드를 사용하라
date: 2024-12-12 00:00:00
categories:
- Effective Java
tags:
- Effective Java
---

OOP에서 객체를 생성하는 방법은 매우 중요하다. 가장 기본적인 객체 생성 방식은 생성자를 호출하는 것이다.
Java에서는 전통적으로 생성자(Constructor)를 통해 객체를 생성해왔다. 이펙티브 자바에서는 생성자 대신에 정적 팩터리 메서드를 사용하라고 제안한다.
이 글에서는 정적 팩터리 메서드가 무엇이고 어떤 특징이 있는지 알아보겠다.

## 생성자란 무엇인가

먼저 생성자에 대해 알아보자. 생성자는 객체가 생성될 때 초기화를 담당하는 특별한 메서드이다.
특징으로는 아래와 같다:

- 클래스명과 메서드명이 일치
- 리턴 타입이 없음
- `new` 키워드를 통해 호출됨
- 객체 초기화 담당

예를 들어 상품 결제를 위해 `Payment` 클래스를 구현한다고 하자. `Payment` 클래스는 결제 ID, 금액, 결제 방식와 결제 시각으로 이루어져 있다.
`Money` 클래스는 `Payment` 클래스에서 쓰이고 화폐에 따른 금액과 화폐를 필드로 가지고 있다.

```java
public class Payment {
    private final String paymentId;
    private final Money money;
    private final PaymentMethod method;
    private final LocalDateTime createdAt;

    private PaymentStatus status;

    public Payment(
        String paymentId,
        Money money, 
        PaymentStatus status,
        PaymentMethod method
    ) {
        this.paymentId = paymentId;
        this.money = money;
        this.status = status;
        this.method = method;
        this.createdAt = LocalDateTime.now();
    }

    public enum PaymentStatus {
        PENDING,              // 결제 대기
        WAITING_FOR_DEPOSIT,  // 입금 대기
        COMPLETED,            // 결제 완료
        FAILED                // 결제 실패
    }

    public enum PaymentMethod {
        CREDIT_CARD,    // 신용카드
        BANK_TRANSFER   // 무통장 입금
    }
}

public class Money {
    private final BigDecimal amount;
    private final Currency currency;

    public Money(BigDecimal amount, Currency currency) {
        this.amount = amount;
        this.currency = currency;
    }

    public enum Currency {
        KRW,
        USD
    }
}
```

생성자를 통하여 이 `Payment` 클래스의 인스터스를 생성할 때는 아래의 과정이 이루어진다.

```java
// 신용 카드로 50000₩ 결제
Payment payment1 = new Payment(
    UUID.randomUUID().toString(),
    new Money(new BigDecimal("50000"), Currency.KRW),
    PaymentStatus.PENDING,
    PaymentMethod.CREDIT_CARD
);

// 무통장 입금으로 50$ 결제
Payment payment2 = new Payment(
    UUID.randomUUID().toString(),
    new Money(new BigDecimal("50"), Currency.USD),
    PaymentStatus.WAITING_FOR_DEPOSIT,
    PaymentMethod.BANK_TRANSFER
);
```

생성자를 통해 인스턴스를 생성하는 방법을 알아보았다. 생성자는 객체 생성을 위한 가장 기본적인 방법이지만, 몇 가지 단점을 가지고 있다.
생성자를 사용함으로써 객체를 생성할 때 발생할 수 있는 문제점들에 대해 살펴보도록 하자.

## 생성자의 문제점

### 객체 생성 방식이 외부로 노출

- 객체의 생성 과정이 외부에 노출되면서 내부 구현 세부사항이 드러남
- 생성 로직 변경 시 이 객체를 사용하는 모든 클라이언트 코드를 수정해야 함
- 클라이언트가 자신의 책임 외에도 객체 생성에 대한 책임을 가짐

객체 생성 방식이 외부로 노출된다는 것은 결국 책임 문제이다.
생성자를 통해 객체를 생성하는 방식은 **캡슐화(Encapsulation)**, **TDA(Tell, Don't ask)**, **SRP**, **OCP** 등을 위반하기 쉽다.

### 가독성이 떨어짐

생성자는 의미 있는 이름을 가질 수 없어서 객체 생성의 의도가 명확히 드러나지 않는다.

```java
Payment payment1 = new Payment(
    UUID.randomUUID().toString(),
    new Money(new BigDecimal("50000"), Currency.KRW),
    PaymentMethod.CREDIT_CARD
);

Payment payment2 = Payment.createCardPayment(money, Money.wons(50000));
```

첫 번째 코드는 생성자를 통한 객체 생성이고, 두 번째 코드는 정적 팩터리 메서드를 통한 객체 생성이다.
두 코드는 '신용 카드로 50000₩ 결제'라는 똑같은 의미를 가지고 있다. 첫 번째 코드보다 두 번째 코드가 가독성이 좋다는 것은 명백하다.
이는 정적 팩터리 메서드가 이름을 가질 수 있다는 것과 관련있다. 또한, 생성자는 많은 매개변수를 순서대로 나열해야 하기 때문에 실수하기도 쉽다.

### 유연성 부족

생성자는 항상 새로운 객체를 반환해야 하며, 객체 생성 방식을 변경하기 어렵다

- 동일한 정보를 가진 객체에 대해 항상 새로운 객체 생성
  - 싱글톤으로 관리될 수 있는 객체임에도 불구하고 항상 `new` 키워드로 생성해야 함
- 객체 생성 방식이 변경될 경우 모든 클라이언트 코드를 수정해야 함
  - `paymentId`에 대한 스펙이 달라질 경우 모든 클라이언트 코드를 일일이 수정해야 함(ID를 커스터마이징 하는 경우)
  - **OCP(Open-CLosed Principle)** 위반

---

## 정적 팩터리 메서드

정적 팩터리 메서드는 객체 생성을 담당하는 static 메서드이다. 객체를 생성하고 초기화하는 역할을 클래스 내부에서 수행하여 캡슐화하고, 생성 의도를 메서드 이름으로 표현할 수 있다.
앞서 살펴본 Payment 클래스의 예시를 정적 팩터리 메서드로 개선해보자:

```java
public class Payment {
    private final String paymentId;
    private final Money money;
    private final PaymentMethod method;
    private final LocalDateTime createdAt;

    private PaymentStatus status;

    // private 생성자를 사용함으로써 외부에서 사용하지 못 하도록 함
    private Payment(Money money, PaymentMethod method) {
        this.paymentId = UUID.randomUUID().toString();
        this.money = money;
        this.method = method;
        this.createdAt = LocalDateTime.now();
    }

    // 신용카드 결제 생성
    public static Payment createCardPayment(Money money) {
        Payment payment = new Payment(money, PaymentMethod.CREDIT_CARD);
        payment.status = PaymentStatus.PENDING;
        return payment;
    }

    // 무통장 입금 결제 생성
    public static Payment createBankTransferPayment(Money money) {
        Payment payment = new Payment(money, PaymentMethod.BANK_TRANSFER);
        payment.status = PaymentStatus.WAITING_FOR_DEPOSIT;
        return payment;
    }
}

public class Money {
    private final BigDecimal amount;
    private final Currency currency;

    // private 생성자를 사용함으로써 외부에서 사용하지 못 하도록 함
    private Money(BigDecimal amount, Currency currency) {
        this.amount = amount;
        this.currency = currency;
    }

    // 원 생성
    public static Money wons(String amount) {
        return new Money(amount, Currency.KRW);
    }

    // 달러 생성
    public static Money dollars(String amount) {
        return new Money(amount, Currency.USD);
    }

    public enum Currency {
        KRW,
        USD
    }
}
```

이제 클라이언트는 아래와 같이 객체를 생성할 수 있다:
```java
// 신용카드로 50000원 결제
Payment cardPayment = Payment.createCardPayment(
    Money.wons(new BigDecimal("50000"))
);

// 무통장 입금으로 50달러 결제
Payment bankPayment = Payment.createBankTransferPayment(
    Money.dollars(new BigDecimal("50"))
);
```

정적 팩터리 메서드를 사용하면 다음과 같은 이점이 있다:

생성자는 클래스와 이름이 같아야 하지만, 정적 팩터리 메서드는 의도를 드러내는 이름을 가질 수 있다
객체 생성을 캡슐화하여 생성 과정에서의 유효성 검사나 초기화 로직을 숨길 수 있다
결제 수단에 따른 적절한 초기 상태 설정이 보장된다

위 예시에서 createCardPayment와 createBankTransferPayment라는 이름을 통해 객체 생성의 의도가 명확히 드러나며,
각 결제 수단에 맞는 적절한 상태(PENDING, WAITING_FOR_DEPOSIT)가 자동으로 설정된다.

<!-- ## 정적 팩터리 메서드의 장점

### 이름을 가질 수 있다
### 호출될 때마다 인스턴스를 새로 생성하지는 않아도 된다
### 반환 타입의 하위 타입 객체를 반환할 수 있다
### 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다
### 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다

## 정적 팩터리 메서드의 단점

### 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다
### 정적 팩터리 메서드는 프로그래머가 찾기 어렵다 -->

## 명명 컨벤션

- `from`: 매개변수를 하나 받아서 해당 타입의 인스턴스를 반환하는 형변환 메서드
  - e.g. `Date d = Date.from(instant);`
- `of`: 여러 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 집계 메서드
  - e.g. `Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING);`
- `valueOf`: `from`과 `of`의 더 자세한 버전
  - e.g. `BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE);`
- `instance` 혹은 `getInstance`: (매개변수를 받는다면) 매개변수로 명시한 인스턴스를 반환하지만, 같은 인스턴스임을 보장하지는 않는다.
  - e.g. `StackWalker luke = StackWalker.getInstance(options);`
- `create` 혹은 `newInstance`: `instance` 혹은 `getInstance`와 같지만, 매번 새로운 인스턴스를 생성해 반환함을 보장한다.
  - e.g. `Object newArray = Array.newInstance(classObject, arrayLen);`
- `getType`: `getInstance`와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 쓴다. *"Type"*은 팩터니 메서드가 반환할 객체의 타입이다.
  - e.g. `FileStore fs = Files.getFileStore(path);`
- `newType`: `newInstance`와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 쓴다. *"Type"*은 팩터리 메서드가 반환할 객체의 타입이다.
  - e.g. `BufferedReader br = Files.newBufferedReader(path);`
- `type`: `getType`과 `newType`의 간결한 버전
  - e.g. `List<Complaint> litany = Collections.list(legacyLitany);`
