---
title: private 생성자나 열거 타입으로 싱글턴임을 보증하라
date: 2025-02-21 00:00:00
categories: [Effective Java]
tags: [Effective Java, Java]
---

## **싱글턴(Singleton)이란**

**싱글턴(Singleton)**이란 인스턴스를 오직 하나만 생성할 수 있는 클래스를 말한다.
싱글턴이라는 개념에 생소하더라도 우리는 이미 싱글턴을 매우 자주 사용하고 있다.
싱글턴의 대표적인 예는 스프링의 **빈(Bean)**이다.
<u>빈은 스프링 IoC 컨테이너에 등록되어 관리되는 객체</u>이다.
`@RestController`, `@Service`, `@Repository`, `@Configuration`, `@Component`, `@Bean` 등의 애너테이션을 사용하여 객체를 빈으로 등록한다.

스프링 빈은 싱글턴으로 관리됨으로써 아래와 같은 장점을 지닌다:

- 하나의 인스턴스를 공유하므로 메모리 절약
- 객체 생성 및 초기화 비용 절감
- 의존성 주입이 한 번만 일어나므로 상태 관리 용이

물론 `@Scope` 애너테이션으로 빈의 생명주기와 사용 범위를 관리할 수 있다.

예를 들어:

```java
@Scope("prototype")
@Component
public class PrototypeBean {
    // 요청할 때마다 새로운 인스턴스 생성

    ...
}
```

위 코드로 생성된 빈은 요청할 때마다 새로운 인스턴스를 생성하므로 애플리케이션 내에서 싱글턴이 아니게 된다.
하지만 대부분의 경우는 빈을 싱글턴으로 관리하므로 빈의 기본 설정은 싱글턴이다.

## **싱글턴 구현 방법**

다음으로는 싱글턴을 만들고 사용하는 방법에 대해 알아보겠다.
이펙티브 자바에서 제안하는 싱글턴 구현 방식은 세가지이지만,
싱글턴 구현 방식은 보통 둘 중 하나이다.
두 방식의 공통점으로는 `private` 접근 제어자를 사용함으로써 외부에서의 생성을 방지한다는 것과 `public static` 멤버를 보유한다는 것이다.
세 번째 방법은 `Enum` 클래스를 활용하는 것이다.

### **`public static final` 필드 방식**

가장 단순한 방식으로, `private` 생성자와 `public static final` 필드를 사용하는 방식이다.

```java
public class Counter {
    
    public static final Counter INSTANCE = new Counter();
    private int count = 0;
    
    private Counter() {}
    
    public void increment() { count++; }
    
    public int getCount() { return count; }
}
```

생성자는 `private`으로 선언되어 외부로부터의 호출을 방지한다.
`private` 생성자는 `public static final` 필드인 `Counter.INSTANCE`를 초기화할 때 단 한 번만 호출된다.
그러므로 `Counter` 인스턴스는 전체 시스템에서 하나뿐임을, 즉 싱글턴임을 보장한다.

이 방식의 특징은:

- 필드가 `public`이므로 외부에서 직접 접근 가능
- `static`이므로 클래스 로딩 시점에 초기화
- `final`이므로 한번 초기화된 인스턴스 변경 불가

클라이언트 코드에서는 아래 방식으로 해당 싱글턴을 사용한다:

```java
Counter counter = Counter.INSTANCE;
counter.increment();
System.out.println("count: " + counter.getCount()); // count: 1

Counter anotherCounter = Counter.INSTANCE;
System.out.println(counter == anotherCounter);      // true
```

객체 간 `==` 동일성 비교를 통해 두 객체가 동일한 주소를 가지고 있다는 것과 이것을 통해 싱글턴임을 알 수 있다.

### **정적 팩터리 메서드 방식**

두 번째 방식은 정적 팩터리 메서드를 `public static` 멤버로 제공하는 방식이다.

```java
public class Counter {

    private static final Counter INSTANCE = new Counter();
    private int count = 0;

    private Counter() {}

    public static Counter getInstance() { return INSTANCE; }

    ...
}
```

`private` 생성자로 외부에서의 생성을 방지한다는 점은 동일하다.
하지만 첫 번째 방식과의 차이점은 필드의 접근자가 `public`에서 `private`으로 바뀌었다는 점이다.
따라서 클래스 외부에서는 `INSTANCE` 필드로 직접 접근하는 것이 불가능해졌다.
대신 정적 팩터리 메서드를 정의함으로써 싱글턴 인스턴스에 접근한다.

```java
Counter counter = Counter.getInstance();
counter.increment();
System.out.println("count: " + counter.getCount()); // count: 1
```

이 방식은 클라이언트가 인스턴스를 어떻게 얻는지에 대한 세부 구현을 숨기고, 단지 getInstance() 메서드를 통해서만 접근하도록 강제한다.
이러한 캡슐화 덕분에 인스턴스의 생성과 관리 방식을 나중에 변경하더라도 클라이언트 코드에는 영향을 주지 않을 수 있다.
첫 번째 방식과 비교했을 때, 메서드를 통한 접근 방식은 더 유연하다는 장점이 있다.
또한 다음과 같은 구체적인 이점들도 제공한다.

#### **API 변경 없이 싱글턴이 아니게 변경 가능**

이 방식의 가장 큰 장점은 API를 변경하지 않고도 싱글턴이 아니게 변경할 수 있다는 점이다.

예를 들어, 애플리케이션의 요구사항이 변경되어 각 호출마다 새로운 인스턴스를 반환해야 하는 경우:

```java
public class Counter {

    private int count = 0;

    private Counter() {}

    // API 그대로 유지
    public static Counter getInstance() {
        return new Counter(); // 새로운 인스턴스 생성
    }
}
```

또는 스레드마다 고유한 인스턴스가 필요한 경우:

```java
public class Counter {
    
    private static final ThreadLocal<Counter> threadLocalInstance = ThreadLocal.withInitial(() -> new Counter());
    private int count = 0;

    private Counter() {}

    // API 그대로 유지
    public static Counter getInstance() {
        return threadLocalInstance.get(); // 현재 스레드의 인스턴스 반환
    }
}
```

이처럼 정적 팩터리 메서드를 사용하면 클라이언트 코드를 변경하지 않고도 싱글턴 인스턴스의 생성과 관리 정책을 유연하게 변경할 수 있다.

#### **제네릭 싱글턴 팩터리 패턴 구현 가능**

정적 팩터리 메서드를 사용하면 제네릭 타입의 싱글턴을 구현할 수 있다.
이는 타입 안전성을 보장하면서도 여러 타입에 동일한 싱글턴 객체를 재사용할 수 있는 방법을 제공한다.

```java
public class GenericSingleton<T> {

    private static final GenericSingleton<?> INSTANCE = new GenericSingleton<>();

    private GenericSingleton() {}

    // 제네릭 타입 T를 받아 해당 타입의 싱글턴을 반환하는 정적 팩터리 메서드
    @SuppressWarnings("unchecked")
    public static <T> GenericSingleton<T> getInstance() {
        return (GenericSingleton<T>) INSTANCE; // 기존 인스턴스를 T 타입으로 캐스팅하여 반환
    }

    public void printData(T data) {
        System.out.println("data: " + data);
    }    
}
```

이 패턴을 사용하면 실제로는 하나의 인스턴스만 생성하면서도 다양한 타입으로 안전하게 사용할 수 있다.

```java
GenericSingleton<String> stringSingleton = GenericSingleton.getInstance();
stringSingleton.printData("Hello, world!"); // data: Hello, world!

GenericSingleton<Integer> intSingleton = GenericSingleton.getInstance();
intSingleton.printData(42); // data: 42

System.out.println(stringSingleton.hashCode() == intSingleton.hashCode()); // true
```

두 객체 `stringSingleton`과 `intSingleton`은 타입 파라미터는 다르지만 실제로는 메모리상 동일한 인스턴스를 참조하는 싱글턴이다.
이 패턴은 Java의 **타입 소거(type erasure)** 덕분에 가능한데, 런타임에는 제네릭 타입 정보가 제거되어 동일한 클래스의 인스턴스로 취급된다.

이러한 제네릭 싱글턴 팩터리 패턴은 상태를 갖지 않거나 불변 객체에 특히 유용하다.
다양한 타입의 데이터를 처리하는 유틸리티 클래스나 처리기에 적합하며, 메모리 사용량을 최소화하면서도 타입 안전한 인터페이스를 제공할 수 있다.

#### **메서드 참조를 공급자로 사용 가능**

Java8부터 도임된 함수형 프로그래밍 기능을 활용할 때 정적 팩터리 메서드는 특히 유용하다.
정적 팩터리 메서드는 **메서드 참조(Method Reference)**를 통해 공급자 `Supplier<T>` 같은 함수형 인터페이스의 구현체로 사용할 수 있다.

> **메서드 참조(Method Reference)**:
> 메서드 참조는 이미 정의된 메서드를 함수형 인터페이스의 구현체로 간결하게 전달하는 방법이다.
> `::` 연산자를 사용하여 표현한다.
> 예를 들어, `Counter::getInstance`는 `Counter` 클래스의 `getInstance` 메서드를 참조한다.
>
> **공급자**:
> `Supplier<T>`는 매개변수 없이 결과를 제공하는 함수형 인터페이스다.
> 단일 추상 메서드 `T get()`을 가지고 있으며, 호출될 때 `T` 타입의 결과를 반환한다.
> 주로 지연 계산이나 값 생성 등에 사용된다.
{: .prompt-info }

```java
// 메서드 참조를 Supplier로 사용
Supplier<Counter> counterSupplier = Counter::getInstance;

Counter counter = counterSupplier.get();
counter.increment();
```

이러한 방식은 다양한 함수형 프로그래밍 패턴에서 유용하게 활용된다:

```java
// Optional과 함께 사용
Optional.empty()
        .or(() -> Optional.of(counterSupplier.get()))
        .ifPresent(Counter::increment);

// 스트림 API와 함께 사용
Stream.generate(Counter::getInstance)
      .limit(3)
      .forEach(counter -> {
          counter.increment();
          System.out.println("count: " + counter.getCount());
      });
```

이러한 메서드 참조 방식은 코드를 더 간결하고 선언적으로 만들어주며, 의존성 주입 프레임워크나 빌더 패턴 등과 함께 사용할 때 특히 유용하다.
또한 함수형 인터페이스를 받는 API에 싱글턴 인스턴스를 쉽게 제공할 수 있게 해준다.

### **열거 타입 방식**

마지막 세 번째 방식은 원소가 하나인 열거 타입(enum)을 선언하는 방식이다.
이 방식은 <u>이펙티브 자바에서 가장 추천하는 방식</u>이기도 하다.

```java
public enum Counter {
    INSTANCE;

    private int count = 0;

    public void increment() { count++; }

    public int getCount() { return count; }
}
```

열거 타입 싱글턴의 클라이언트 코드에서 사용하는 방식은 매우 간단하다:

```java
Counter counter = Counter.INSTANCE;
counter.increment();
System.out.println("count: " + counter.getCount()); // count: 1
```

클라이언트 코드에서 사용하는 방식은 `public static final` 방식과 정확히 동일하다.
열거 타입 방식을 가장 추천하는 이유는 구현하고 사용하는 데 간편한 것에 그치지 않는다.

#### **직렬화 자동 처리**

**직렬화(Serialization)**란 자바 객체를 바이트 스트림으로 변환하여 파일로 저장하거나 네트워크로 전송할 수 있게 하는 메커니즘이다.
반대로 **역직렬화(Deserialization)**는 바이트 스트림을 다시 자바 객체로 복원하는 과정이다.

싱글턴에서 직렬화는 매우 중요한 문제인데, 기본적으로 <u>역직렬화는 항상 새로운 인스턴스를 생성</u>하기 때문이다.
그래서 싱글턴 패턴이 깨질 위험이 있다.
열거 타입은 다른 싱글턴 구현 방식과 달리 <u>직렬화가 자동으로 처리 되고 역직렬화 시에 기존에 생성된 인스턴스를 반환</u>한다.
다른 방식은 `Serializable`을 구현하고 `readResolve` 메서드를 추가해야 하는 별도의 작업이 필요하다.

열거 타입 이외의 방법으로 싱글턴을 구현하고 직렬화를 처리하려면:

```java
public class Counter implements Serializable {
    
    private static final long serialVersionUID = 1L;
    public static final INSTANCE = new Counter();

    ...

    // 역직렬화 시 호출되는 메서드, 새로운 인스턴스 생성 없이 기존 인스턴스 반환
    private Object readResolve() { return INSTANCE; }
}
```

`Serializable`을 구현할 때 `readResolve` 메서드가 없다면, 역직렬화 시에 새로운 인스턴스가 생성되어 싱글턴 특성이 깨진다.

열거 타입으로 싱글턴을 구현하면 위의 복잡한 추가 작업이 없이 직렬화를 처리하고 역직렬화 시에도 싱글턴 패턴을 보증한다.

#### **리플렉션 공격으로부터 안전**

**리플렉션(Reflection)** 은 자바 프로그램이 <u>런타임에 자신의 구조를 검사하고 수정할 수 있게 해주는 API</u>이다.
리플렉션을 이용한다면 프로그램은 컴파일 시점에 알 수 없었던 클래스와 객체의 정보에 접근할 수 있다.
이를 통해 <u>클래스의 <code>private</code> 멤버에도 접근할 수 있어 보안상의 위험을 초래</u>할 수 있다.

아래는 리플렉션으로 클래스의 `private` 멤버에 접근한다:

```java
public class Counter {

    public static final Counter INSTANCE = new Counter();

    // 인스턴스 생성 확인을 위해 출력문 추가
    private Counter() {
        System.out.println("New instance is created");
    }

    ...
}
```

```java
try {
    Constructor<Counter> constructor = Counter.class.getDeclaredConstructor(); // Counter 클래스의 private 생성자 객체 획득
    constructor.setAccessible(true); // 생성자의 접근성 변경 (접근 허용)

    Counter counter = Counter.getInstance(); // 기존 인스턴스
    Counter reflectionCounter = constructor.newInstance(); // 새로운 인스턴스 생성

    System.out.println(counter == reflectionCounter); // 인스턴스 비교
} catch (Exception e) {
    e.printStackTrace();
}
```

실행 결과는 아래와 같다:

```terminal
New instance is created
New instance is created
false
```

인스턴스가 두 번 생성되었다는 것은 생성자가 두 번 호출되었다는 것을 의미한다.
이는 클라이언트 코드에서 리플렉션으로 `private` 생성자를 호출 가능하도록 변경했기 때문이다.
당연히 `==` 연산자로 객체 동일성 비교를 하면 두 객체는 같지 않다.

리플렉션은 런타임에 클래스 구조에 접근하고 조작하는 것을 허용함으로써 유연함을 제공할 수도 있지만,
실제로 사용하기에는 <u>접근 제한을 무시하여 캡슐화, 정보 은닉 등의 객체지향 원칙을 위반</u>할 가능성이 매우 크다.
때문에 테스트 도구 같은 특별한 경우를 제외하고는 사용을 신중히 고려해야 한다.

<u>열거 타입은 Java 언어 명세에 의해 리플렉션을 통한 인스턴스 생성이 금지</u>되어 있다.
열거 타입으로 싱글턴을 구현했을 때, 리플렉션으로 새로운 인스턴스를 생성하면 예외가 발생한다:

```terminal
java.lang.IllegalArgumentException: Cannot reflectively create enum objects
...
```

## **결론**

세 가지 방식 모두 싱글턴을 구현할 수 있지만, 특별한 이유가 없다면 열거 타입 방식을 사용하는 것이 가장 좋다.
코드가 간결하고, 직렬화와 리플렉션 공격에 대한 방어가 자동으로 처리되기 때문이다.
다만, 열거 타입은 다른 클래스를 상속할 수 없으므로 상속이 필요한 경우 또는 정적 팩터리 메서드의 추가 기능이 필요할 경우에는 선택할 수 없다.
상황에 따라 적절한 싱글턴 구현 전략을 선택해야 하는 것은 개발자 개인의 몫이다.
