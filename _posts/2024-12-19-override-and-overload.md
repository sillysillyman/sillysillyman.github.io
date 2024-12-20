---
title: 오버라이딩과 오버로딩
date: 2024-12-19 00:00:00
categories:
- CS
tags: [CS, Java]
---

오버라이딩과 오버로딩은 **다형성(Polymorphism)**을 구현하는 방법이다. 먼저 다형성의 정의는 아래와 같다.

> In programming language theory and type theory,
> polymorphism is the use of a single symbol to represent multiple different types.
> 
> _번역: 프로그래밍 언어 이론과 타입 이론에서, 다형성이란 다른 여러 타입을 나타내기 위해 하나의 기호를 사용하는 것이다._

쉽게 말하자면, 하나의 이름이 여러가지 형태로 사용될 수 있다는 것이다. 정의만 보고는 이해하기 어려울 수 있으나 확실한 것은 우리는 이미 이 개념을 잘 사용하고 있다는 것이다.

```cpp
#include <iostream>

using namespace std;

int main() {
  string hello = "Hello";
  string world = "World!";

  cout << 1 + 1 << '\n';                // 2
  cout << hello + ' ' + world << '\n';  // Hello World!
}
```

위 C++ 코드는 간단한 정수 덧셈과 문자열 출력 예제이다. 공통점은 항이 `+` 기호로 연산된다는 것이다.
똑같은 기호 `+`를 사용해서 정수의 덧셈을 계산할 수도 있고 문자열을 이을 수 있다. 이것은 대표적인 오버로딩인 연산자 오버로딩의 예시이다.

## Overload

위에서 다형성의 예시로 오버로딩을 언급했으니 오버로드부터 설명하겠다.

### Method Overlaod

메서드 오버로딩이란 여러 메서드를 같은 이름으로 정의하되 각각 다르게 구현하는 것을 뜻한다. 여기서 다르게 구현한다는 것은 매개변수 목록(개수, 타입, 순서)을 뜻한다.
아래는 메서드 오버로딩에 대한 간단한 예시 Java 코드이다.

```java
class MathUtils {
    int add(int a, int b) {
        return a + b;
    }

    double add(double a, double b) {
        return a + b;
    }
}

public class Main {
    public static void main(String[] args) {
        MathUtils mathUtils = new MathUtils();

        System.out.println(mathUtils.add(2, 3));        // 5
        System.out.println(mathUtils.add(2.5, 3.5));    // 6.0
    }
}
```

여기서 두 `add` 메서드는 동일한 이름으로 각각 `int` 타입 덧셈과 `double` 타입 덧셈 기능을 한다. 만약 오버로딩을 사용하지 않으면 아래와 같이 메서드 시그니처를 달리 해야 한다.

```java
int addInt(int a, int b);
double addDouble(double a, double b);
```

위 두 메서드는 동일한 논리의 덧셈 연산을 불필요하게 이름을 다르게 하는 것이기 때문에 코드를 작성하기에 매우 비효율적이다.

여담으로 예시 중 `println` 메서드도 오버로드되었다고 할 수 있다. 인자로 받는 것이 각각 `int` 타입과 `double` 타입으로 다르기 때문이다.

### Operator Overload

연산자 오버로드는 연산자 (예: +, -, *, / 등)의 기능을 확장하여, 사용자 정의 타입에 대해 연산자를 사용할 수 있도록 하는 방식이다.
연산자 오버로딩은 사실 메서드 오버로딩에 포함된다. 연산자 또한 일종의 함수, 즉 메서드이기 때문이다. 아래는 연산자 오버로딩에 대한 간단한 C++ 코드이다.

```cpp
#include <iostream>

using namespace std;

class Vector2 {
 private:
  int x_;
  int y_;

 public:
  Vector2(int x = 0, int y = 0) : x_(x), y_(y) {}

  int operator*(const Vector2& other) const {
    return x_ * other.x_ + y_ * other.y_;
  }

  friend ostream& operator<<(ostream& os, const Vector2& vec) {
    os << "(" << vec.x_ << ", " << vec.y_ << ")";
    return os;
  }
};

int main() {
  Vector2 v1(1, 2);
  Vector2 v2(3, 4);

  cout << "Dot Product: " << v1 * v2 << '\n';  // Dot Product: 11
  cout << "v1: " << v1 << '\n';                // v1: (1, 2)
  cout << "v2: " << v2 << '\n';                // v2: (3, 4)

  return 0;
}
```

위 코드는 두 2차원 벡터의 내적과 각각의 벡터를 출력하는 코드이다. 여기서 주목할 부분은 `operator*`와 `operator<<` 함수이다.
이 함수들은 각각 `*`와 `<<` 연산자를 오버로딩하여, `Vector2` 객체에 대해 사용자 정의 동작을 구현한다.

연산자 오버로딩은 코드를 더 직관적이고 가독성 있게 만들어 준다. 특히, 복잡한 객체나 사용자 정의 타입에 대해 연산을 수행할 때 유용하다.
위 예시처럼 벡터 간의 내적을 계산할 때 `v1 * v2`와 같은 표현은 수학적으로 직관적이며, 코드를 읽는 사람이 바로 그 의미를 이해할 수 있게 한다.
만약 연산자 오버로딩이 없다면, 내적 계산은 `v1.dot(v2)` 같은 메서드 형태로 호출해야 할 것이다.

## Override

오버로딩에 이어 다형성을 구현하는 또 다른 중요한 방법인 오버라이딩에 대해 설명하겠다.
오버라이딩은 객체지향 프로그래밍에서 매우 중요한 개념으로, 상속을 통해 파생된 클래스가 부모 클래스의 메서드를 재정의할 수 있도록 한다.
즉, 부모 클래스에서 정의된 메서드를 자식 클래스에서 동일한 이름과 시그니처로 다시 정의하는 것을 의미한다.
이 과정을 통해 자식 클래스는 부모 클래스의 메서드를 자신에게 맞게 변경할 수 있으며, 객체의 행동을 커스터마이징할 수 있다. 아래는 오버라이딩에 대한 간단한 Java 코드이다.

```java
class Animal {
    void makeSound() {
        System.out.println("Some generic animal sound");
    }
}

class Dog extends Animal {
    @Override
    void makeSound() {
        System.out.println("Woof Woof");
    }
}

class Cat extends Animal {
    @Override
    void makeSound() {
        System.out.println("Meow");
    }
}

public class Main {
    public static void main(String[] args) {
        Animal animal = new Animal();
        Animal dog = new Dog();
        Animal cat = new Cat();

        animal.makeSound(); // Some generic animal sound
        dog.makeSound();    // Woof Woof
        cat.makeSound();    // Meow
    }
}
```

위 코드에서 `Dog` 클래스와 `Cat` 클래스는 `Animal` 클래스를 상속받아 `makeSound` 메서드를 오버라이딩한다.
결과적으로, `dog.makeSound()` 호출 시 부모 클래스인 `Animal`의 `makeSound`가 아닌 자식 클래스인 `Dog`에서 정의된 `makeSound` 메서드가 호출되어 `Woof Woof`가 출력된다.
이와 마찬가지로 `cat.makeSound()`는 `Meow`를 출력한다.

### Override의 필요성

오버라이딩은 객체지향 프로그래밍의 핵심 개념인 다형성을 구현하는 데 필수적이다. 이를 통해 부모 클래스의 일반적인 동작을 자식 클래스에서 구체화하거나 확장할 수 있다.
예를 들어, 다양한 동물 클래스를 상속받는 각각의 동물 클래스가 고유의 `makeSound` 메서드를 가지도록 함으로써, `Animal` 타입의 참조 변수를 통해 각각 다른 동작을 실행할 수 있게 된다.

### `@Override`의 역할

`@Override` 애너테이션은 컴파일러에게 해당 메서드가 상위 클래스의 메서드를 오버라이딩하고 있음을 명시적으로 알리는 역할을 한다.
이 애너테이션을 사용하면, 컴파일러는 해당 메서드가 상위 클래스나 인터페이스의 메서드를 정확하게 오버라이딩하고 있는지 확인한다.
만약 상위 클래스에 동일한 시그니처의 메서드가 존재하지 않으면, 컴파일 오류가 발생한다.

#### `@Override`가 없는 경우

`@Override` 애너테이션이 없는 경우, 컴파일러는 해당 메서드가 상위 클래스의 메서드를 오버라이딩한다고 명시적으로 가정하지 않는다.
단순히 메서드가 상위 클래스에 정의된 것과 동일한지 확인한다. 그러나 만약 메서드의 이름이나 시그니처가 다르다면, 그 메서드는 새로운 메서드로 간주되고 오버라이딩되지 않는다.
이로 인해 개발자의 실수로 오버라이딩에 실패하여 의도하지 않은 동작을 할 가능성이 있다.

## 오버로딩과 오버라이딩의 결정 시점

오버로딩과 오버라이딩은 각각 컴파일 시점과 런타임 시점에서 결정되는 방식이 다르다. 이 두 가지 다형성의 결정 시점을 비교하겠다.

### 오버로딩: 컴파일 타임에 결정

동일한 이름의 메서드가 여러 개 정의될 때, 컴파일러는 전달된 인자의 개수와 타입에 따라 적절한 메서드를 선택한다.
이는 주로 같은 클래스 내에서 발생하며, 객체의 다양한 초기화 방법을 제공하거나, 서로 다른 타입의 데이터를 처리할 수 있는 유연성을 제공한다.

### 오버라이딩: 런타임에 결정

자식 클래스가 부모 클래스에서 상속받은 메서드를 재정의함으로써, 다형성을 통해 런타임에 객체의 실제 타입에 맞는 메서드를 호출할 수 있게 한다.
이는 주로 클래스 계층 구조에서 사용되며, 상속을 통한 코드 재사용과 객체 행동의 커스터마이징을 가능케 한다.
컴파일러는 메서드 호출이 가능한지, 즉 상위 클래스에서 선언된 메서드가 하위 클래스에서 오버라이딩된 메서드인지 여부를 검증한다.
그러나 실제로 어떤 메서드가 실행될지는 객체가 가리키는 실제 타입에 의해 런타임에 결정된다.
