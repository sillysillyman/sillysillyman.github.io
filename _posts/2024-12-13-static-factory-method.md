---
title: 생성자 대신 정적 팩터리 메서드를 사용하라
date: 2024-12-12 00:00:00
categories: [Effective Java]
tags: [Effective Java, Java]
---

OOP에서 객체를 생성하는 방법은 매우 중요하다. 가장 기본적인 객체 생성 방식은 생성자를 호출하는 것이다.
Java에서는 전통적으로 생성자(Constructor)를 통해 객체를 생성해왔다. 이펙티브 자바에서는 생성자 대신에 정적 팩터리 메서드를 사용하라고 제안한다.
이 글에서는 정적 팩터리 메서드가 무엇이고 어떤 특징이 있는지 알아보겠다.

## **생성자란 무엇인가**

먼저 생성자에 대해 알아보자. 생성자는 <u>객체가 생성될 때 초기화를 담당하는 특별한 메서드</u>이다.
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

## **생성자의 단점**

### **객체 생성 방식이 외부로 노출**

- 객체의 생성 과정이 외부에 노출되면서 내부 구현 세부사항이 드러남
- 생성 로직 변경 시 이 객체를 사용하는 모든 클라이언트 코드를 수정해야 함
- 클라이언트가 자신의 책임 외에도 객체 생성에 대한 책임을 가짐

객체 생성 방식이 외부로 노출된다는 것은 결국 책임 문제이다.
생성자를 통해 객체를 생성하는 방식은 **캡슐화(Encapsulation)**, **TDA(Tell, Don't ask)**, **SRP**, **OCP** 등을 위반하기 쉽다.

### **가독성이 떨어짐**

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

### **유연성 부족**

생성자는 항상 새로운 객체를 반환해야 하며, 객체 생성 방식을 변경하기 어렵다

- 동일한 정보를 가진 객체에 대해 항상 새로운 객체 생성
  - 싱글톤으로 관리될 수 있는 객체임에도 불구하고 항상 `new` 키워드로 생성해야 함
- 객체 생성 방식이 변경될 경우 모든 클라이언트 코드를 수정해야 함
  - `paymentId`에 대한 스펙이 달라질 경우 모든 클라이언트 코드를 일일이 수정해야 함(ID를 커스터마이징 하는 경우)
  - **OCP(Open-CLosed Principle)** 위반

---

## **정적 팩터리 메서드**

정적 팩터리 메서드는 객체 생성을 담당하는 static 메서드이다. <u>객체를 생성하고 초기화하는 역할을 클래스 내부에서 수행하여 캡슐화</u>하고, <u>생성 의도를 메서드 이름으로 표현</u>할 수 있다.
앞서 살펴본 `Payment` 클래스의 예시를 정적 팩터리 메서드로 개선해보자:

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

이제 클라이언트에서는 아래와 같이 객체를 생성할 수 있다:

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

위 예시에서 `createCardPayment`와 `createBankTransferPayment`라는 정적 팩터리 메서드를 사용했다.
이는 이름을 통해 객체 생성의 의도가 명확히 드러나며, 각 결제 수단에 맞는 적절한 상태(`PENDING`, `WAITING_FOR_DEPOSIT`)가 자동으로 설정된다.

다음으로는 이펙티브 자바에서 말하는 정적 팩터리 메서드의 장점을 예시와 함께 설명하겠다.

## **정적 팩터리 메서드의 장점**

### **이름을 가질 수 있다**

생성자는 클래스와 동일한 이름을 사용해야 하지만, 정적 팩터리 메서드는 의도를 명확히 드러내는 이름을 가질 수 있다.
예를 들어 `Payment` 클래스의 `createCardPayment`와 `createBankTransferPayment`는 각각 카드 결제와 무통장 입금 결제를 생성한다는 의도를 명확히 전달한다.

### **호출될 때마다 인스턴스를 새로 생성하지는 않아도 된다**

불변 객체를 미리 만들어 놓거나 새로 생성한 객체를 캐싱하여 재활용하는 식으로 <u>불필요한 객체 생성을 피할 수 있다.</u>
예를 들어 결제 설정값과 같이 애플리케이션 전반에서 동일하게 사용되는 객체는 <u>싱글톤으로 관리</u>할 수 있다.

```java
public class PaymentConfig {
    private static final PaymentConfig INSTANCE = new PaymentConfig();
    
    private PaymentConfig() {}
    
    public static PaymentConfig getInstance() {
        return INSTANCE;
    }
}
```

개발자는 불필요하게 매번 `PaymentConfig` 객체를 생성할 필요 없이
정적 팩터리 메서드로 구현된 `PaymentConfig.getInstance()`을 호출함으로써 싱글톤으로 관리되는 `INSTANCE`를 사용할 수 있다.

### **반환 타입의 하위 타입 객체를 반환할 수 있다**

반환할 객체의 실제 타입을 숨길 수 있어 **API의 유연성**이 높아진다.
예를 들어 결제 처리기를 생성할 때, 결제 수단에 따라 다른 구현체를 반환할 수 있다.

```java
public interface PaymentProcessor {
    void process();

    static PaymentProcessor createCreditCardProcessor() {
        return new CreditCardProcessor();
    }

    static PaymentProcessor createBankTransferProcessor() {
        return new BankTransferProcessor();
    }
}
```

Java 8 이후로는 인터페이스가 정적 메서드를 가질 수 있기 때문에 위처럼 인터페이스에 구현할 수 있다.
이 장점으로 실제 구현부를 외부로부터 감추고 클라이언트에서는 인터페이스만 알면 된다.

### **입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다**

두 번째 장점에서 설명한 코드를 매개변수를 받도록 바꿔보자.
`PaymentProcessor`는 결제 방법에 따라 달라지기 때문에 `PaymenyMethod`를 매개변수로 받는 정적 팩터리 메서드를 만들 수 있다.

```java
public class PaymentProcessorFactory {
    public static PaymentProcessor createProcessor(PaymentMethod method) {
        return switch (method) {
            case CREDIT_CARD -> new CreditCardProcessor();
            case BANK_TRANSFER -> new BankTransferProcessor();
        };
    }
}
```

이 장점 또한 비즈니스 규칙에 따른 <u>객체 생성을 캡슐화하여 클라이언트에게 내부 구현 로직을 노출시키지 않는다</u>는 장점이 있다.
또한, 구현체 선택 로직을 한 곳에서 관리하여 만약 다른 결제 방법이 추가된다면 이 정적 메서드만을 수정하면 되기에 OCP를 준수한다고 할 수 있다.

### **정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다**

이 장점은 특히 서비스 제공자 프레임워크(Service Provider Framework) 패턴을 구현할 때 핵심이 된다.
서비스 제공자 프레임워크란 서비스의 구현체를 등록하고 접근하는 체계를 말한다.

온라인 쇼핑몰에서 다양한 PG사(Payment Gateway)의 결제 시스템을 유연하게 연동해야 하는 상황을 가정해보자.
우리는 아직 어떤 PG사와 연동할지 모르지만, 향후 어떤 PG사가 추가되더라도 유연하게 대응할 수 있는 구조가 필요하다.

```java
// 결제 게이트웨이 인터페이스
public interface PaymentGateway {
    void processPayment(Payment payment);
}

// 결제 게이트웨이를 관리하는 팩터리
public class PaymentGatewayFactory {
    private static final Map<String, PaymentGateway> gateways = new HashMap<>();
    
    // 게이트웨이 등록
    public static void registerGateway(String name, PaymentGateway gateway) {
        gateways.put(name, gateway);
    }
    
    // 게이트웨이 조회
    public static PaymentGateway getGateway(String name) {
        return gateways.get(name);
    }
}
```

이렇게 기본 구조를 작성해두면, 나중에 각 PG사들은 자신들만의 결제 게이트웨이 구현체를 제공할 수 있다:

```java
// PG사별 게이트웨이 구현체
public class TossPaymentGateway implements PaymentGateway {
    @Override
    public void processPayment(Payment payment) {
        // 토스 결제 처리
    }
}

public class KakaoPaymentGateway implements PaymentGateway {
    @Override
    public void processPayment(Payment payment) {
        // 카카오페이 결제 처리
    }
}
```

이제 각 PG사의 구현체를 등록하고 사용할 수 있다:

```java
// 구현체들을 등록
PaymentGatewayFactory.registerGateway("TOSS", new TossPaymentGateway());
PaymentGatewayFactory.registerGateway("KAKAO", new KakaoPaymentGateway());

// 사용
Payment payment = new Payment(Money.wons("50000"));
PaymentGateway gateway = PaymentGatewayFactory.getGateway("TOSS");
gateway.processPayment(payment);
```

이러한 설계의 장점은 새로운 PG사가 추가되어도 기존 코드를 수정할 필요가 없다는 것이다.
새로운 구현체를 만들고 등록하기만 하면 된다.
이처렁 정적 팩터리 메서드를 통해 <u>아직 존재하지 않는 구현체들이 나중에 추가될 수 있는 유연한 확장 구조를 만들 수 있다.</u>

## **정적 팩터리 메서드의 단점**

정적 팩터리 메서드가 많은 장점을 가지고 있지만, 두 가지 주요한 단점도 있다.
각각의 단점을 자세히 살펴보자.

### **정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다**

상속을 하기 위해서는 `public`이나 `protected` 생성자가 필요하다.
하지만 정적 팩터리 메서드만 제공하는 클래스는 생성자가 `private`이기 때문에 **상속이 불가능**하다.

```java
public class Payment {
    private final Money money;
    private final PaymentMethod method;

    // private 생성자 - 상속 불가능
    private Payment(Money money, PaymentMethod method) {
        this.money = money;
        this.method = method;
    }

    // 정적 팩터리 메서드
    public static Payment createCardPayment(Money money) {
        return new Payment(money, PaymentMethod.CREDIT_CARD);
    }
}

// 컴파일 에러 - Payment의 생성자가 private이라 상속 불가능
public class SubscriptionPayment extends Payment {
    private final Period period;
    
    public SubscriptionPayment(Money money, Period period) {
        super(money, PaymentMethod.CREDIT_CARD);  // 불가능
        this.period = period;
    }
}
```

하지만 이 제약은 <u>상속보다 컴포지션을 사용하도록 유도한다는 점에서 특정 경우에서는 오히려 장점</u>이 될 수 있다.
상속은 캡슐화를 깨뜨리고 상위 클래스와 하위 클래스가 강하게 결합되는 단점이 있기 때문이다.

### **정적 팩터리 메서드는 프로그래머가 찾기 어렵다**

생성자는 클래스와 동일한 이름을 가지며 API 문서에서 따로 모아서 보여주기 때문에 찾기 쉽다.
반면 정적 팩터리 메서드는 다른 메서드들과 구분되지 않아 찾기가 어려울 수 있다.

이러한 단점을 완화하기 위해 메서드 이름을 짓는 규약이 있다.

#### **명명 컨벤션**

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

이런 명명 규칙을 따르면 정적 팩터리 메서드를 찾기가 한결 수월해진다.

그러나 이러한 단점들은 정적 팩터리 메서드가 주는 장점들에 비하면 크게 문제되지 않는다.
특히 상속을 제한하는 것은 때로는 바람직할 수 있으며, 메서드를 찾기 어렵다는 문제는 널리 알려진 명명 규칙을 따름으로써 완화할 수 있기 때문이다.
