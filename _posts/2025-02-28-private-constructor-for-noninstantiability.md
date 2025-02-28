---
title: 인스턴스화를 막으려거든 private 생성자를 사용하라
date: 2025-02-27 00:00:00
categories: [Effective Java]
tags: [Effective Java, Java]
---

## **유틸리티 클래스 (Utility Class)**

<u>유틸리티 클래스는 정적 메서드와 정적 필드만을 담고 있는 클래스</u>를 의미한다.
예를 들어, JDK(Java Development Kit)에 포함된 `java.lang.Math`, `java.util.Arrays`, `java.util.Collections` 같은 클래스가 유틸리티 클래스이다.

유틸리티 클래스의 모든 멤버는 정적으로 선언되기 때문에 클라이언트 코드에서는 인스턴스 메서드가 아닌 정적 메서드를 호출한다.
예를 들어, `Circle` 클래스가 아래와 같을 때,

```java
public class Circle {

    private final double radius;

    public Circle(double radius) {
        this.radius = radius;
    }

    public double getArea() {
        return Math.PI * Math.pow(radius, 2);
    }
}
```
`Math.PI`와 `Math.pow()` 메서드가 `Math` 유틸리티 클래스의 정적 멤버이다.
클라이언트 코드에서는 다음과 같이 사용하다:

```java
Circle circle = new Circle(1);
System.out.println("Area: " + circle.getArea()); // Area: 3.141592653589793
```

만약 `getAreat()` 메서드의 `Math` 클래스를 아래처럼 인스턴스화 하여 사용하려고 하면 에러가 발생한다:

```java
public double getArea() {
    Math math = new Math();
    return math.PI * math.pow(radius, 2);
}
```

```terminal
java: Math() has private access in java.lang.Math
```

유틸리티 클래스는 상태를 저장하지 않는다.
즉, 인스턴스 변수를 가지지 않고 모든 메서드는 입력 매개변수만으로 결과를 반환한다.
그러므로 인스턴스화 자체가 불필요한 것이다.
따라서 Java Standard Library의 유틸리티 클래스는 이미 인스턴스화를 방지했기 때문에 오류가 발생했다.

## **기본 생성자의 함정**

따라서 개발자가 커스텀으로 유틸리티 클래스를 만들 때는 인스턴스화를 방지하는 것이 매우 중요하다.
예를 들어 아래 유틸리티 클래스를 정의하면:

```java
public class UtilityClass {

    ...
}
```

위 코드는 생성자를 명시하지 않았다.
생성자를 명시하지 않았음에도 불구하고 아래 코드처럼 생성자를 호출하여 인스턴스를 만들 수 있다.

```java
UtilitieClass utilityClass = new UtilitiyClass();
```

생성자가 없는데도 인스턴스를 만들 수 있는 이유는 컴파일러가 자동으로 **기본 생성자(매개변수가 없는 public 생성자)**를 만들어 주기 때문이다.
이로 인해 의도치 않게 인스턴스화가 가능하다.
즉, 유틸리티 클래스의 핵심은 어떻게 인스턴스 생성을 방지하느냐이다.

### **추상 클래스?**

직관적으로 떠오르는 아이디어는

> **추상 클래스(Abstract Class)**는 인스턴스를 생성할 수 없다.

이다.

기존 유틸리티 클래스를 `abstract` 키워드를 사용하여 추상 클래스로 정의해보자:

```java
public abstract class UtilityClass {

    ...
}
```

아래와 같은 코드로 위 유틸리티 클래스를 인스턴스화 하는 것은 불가능하다:

```java
UtilityClass utilityClass = new UtilityClass();
```

```terminal
java: UtilityClass is abstract; cannot be instantiated
```

컴파일러는 친절하게 에러 메시지를 제공하여 <u>추상클래스는 인스턴스화 할 수 없음</u>을 알려준다.
그러나 이 방법은 효과적이지 않다.
이유는 추상 클래스의 목적 그 자체에 있고, 그것은 추상 클래스가 인스턴스화 할 수 없음과 밀접한 관련이 있다.

추상 클래스의 본래 목적은 **상속(Inheritance)**이다.
따라서 추상 클래스로 정의된 유틸리티 클래스는 상속하여 사용될 수 있다.

```java
public class ExtendedUtilityClass extends UtilityClass {
    
    ...
}
```

```java
ExtendedUtilityClass extendedUtilityClass = new ExtendedUtilityClass(); // 인스터스 생성 가능
```

컴파일러는 위 코드를 전혀 문제가 없다고 인식한다.
그렇다고 해서 그것이 문제가 없는 것인가?
예를 들어 `ExtendedUtilityClass`에서 새로운 메서드를 추가한다거나 상위 클래스(`UtilityClass`)의 메서드를 오버라이딩 한다고 생각해보자.
그 추가 작업을 `UtilityClass`에 하지 않을 이유가 전혀 없다.
유틸리티 클래스는 애플리케이션 전체에서 공통적으로 활용되는 메서드를 정의한 클래스이기 때문에 상속과는 거리가 멀다.

애초에 추상 클래스는 상속할 클래스들의 공통 멤버를 가지고 있고, 그것들이 반드시 정적으로 선언된다는 보장도 없다.
추상 클래스로 구현한다는 것은 개발자들에게 "이 클래스를 확장해서 사용하라는 의미인가?"라는 오해를 불러일으킨다.
즉, 추상 클래스가 인스턴스화를 방지하는 것은 "이 클래스를 상속하여 사용하세요"라는 의미이다.

### **`abstract` + `final` 키워드?**

> 그렇다면 `final` 한정자를 사용하여 상속을 방지하는 것은 어떨까?

클래스를 `final`로 선언하면 다른 클래스가 상속할 수 없게 된다.

- `abstract` 키워드는 클래스의 인스턴스화를 방지함
- `final` 키워드는 클래스 상속을 방지함

인스턴스화도 방지하고 상속도 방지할 것 같다.
자연스럽게 도출되는 나름 합리적인 방법이지만, 이 아이디어는 `abstract`에 대한 이해를 완벽하게 하지 못 한 것이다.
다시 설명하면,

> 추상 클래스의 본래 목적은 **상속(Inheritance)**이다.

여기에 `final` 한정자를 사용하여 상속을 방지한다는 것은 언어의 설계와 맞지 않는 모순되는 행위이다.

> 당연히 Java에서는 `abstract`와 `final`의 혼용이 문법적으로 금지된다.
>
> ```java
> public abstract final class UtilityClass {
>
>    ...
> }
> ```
>
> ```terminal
> java: illegal combination of modifiers: abstract > and final
> ```
{: .prompt-danger }

`abstract`나 `final` 같이 Java가 제공하는 키워드로 간편하게 해결하는 방법은 애초에 없다.

## **private 생성자**

이 문제를 해결하는 가장 효과적인 방법은 `private` 생성자를 추가하는 것이다.
생성자를 `private`으로 선언하면 <u>클래스 외부에서는 접근할 수 없으므로 인스턴스화를 방지</u>할 수 있다.
또한, 호출될 시 <u>예외를 발생시킴으로써 클래스 내부에서의 인스턴스화도 방지</u>한다.

```java
public class UtilityClass {

    private UtilityClass() {
        throw new AssertionError("UtilityClass cannot be instantiated");
    }

    ...
}
```

외부에서 위 클래스의 인스턴스를 생성하려고 하면 오류가 발생한다:

```java
UtilityClass utilityClass = new UtilityClass();
```

```terminal
java: UtilityClass() has private access in UtilityClass
```

## **결론: 인스턴스화 방지를 위한 private 생성자 패턴의 효과**

`private` 생성자를 가진 클래스는 상속이 불가능하다.
모든 생성자는 명시적이든 암묵적이든 상위 클래스의 생성자를 호출해야 하는데, 상위 클래스의 생성자가 `private`이면 하위 클래스에서 접근할 수 없기 때문이다.
따라서 이 패턴은 상속을 통한 확장이 필요하지 않은 유틸리티 클래스에 적합하다.

유틸리티 클래스는 `final`로 선언해도 되고, 안 해도 된다.
`private` 생성자는 그 자체로 상속을 방지하고 있기 때문이다.
다만, 명시적으로 개발자에게 상속이 불가능함을 알리고 싶다면 `final` 클래스로 선언하는 것도 방법이다.

실제로 JDK에 내장된 유틸리티 클래스는 `final`로 선언되었다:

```java
public final class Math {

    private Math() {}

    public static final double E = 2.718281828459045;

    public static final double PI = 3.141592653589793;

    ...
}
```

`private` 생성자 패턴은 간단하지만 효과적인 방법으로, 인스턴스화가 필요 없는 클래스를 설계할 때 반드시 기억해두어야 할 패턴이다.
