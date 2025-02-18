---
title: 생성자에 매개변수가 많다면 빌더를 고려하라
date: 2025-02-05 00:00:00
categories: [Effective Java]
tags: [Effective Java, Java]
---

지난 글에서는 생성자와 정적 팩터리 메서드 통한 객체 생성 방법을 알아보았다.
하지만, 생성자와 정적 팩터리 메서드에는 똑같은 제약이 하나 있다.
바로 선택적 매개변수가 많을 때 적절히 대응하기 어렵다는 점이다.

## **점층적 생성자 패턴 (Telescoping Constructor Pattern)**

아래 예시는 영양 성분에 대한 정보를 담고 있는 클래스이다.

```java
public class NutritionFacts {

    private final int servingSize;  // (ml, 1회 제공량)   필수
    private final int servings;     // (회, 총 n회 제공량) 필수
    private final int calories;     // (kcal/1회 제공량)  선택
    private final int fat;          // (g/1회 제공량)     선택
    private final int sodium;       // (mg/1회 제공량)    선택
    private final int carbohydrate; // (g/1회 제공량)     선택

    public NutritionFacts(int servingSize, int servings) {
        this(servingSize, servings, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories) {
        this(servingSize, servings, calories, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat) {
        this(servingSize, servings, calories, fat, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium) {
        this(servingSize, servings, calories, fat, sodium, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium, int carbohydrate) {
        this.servingSize = servingSize;
        this.servings = servings;
        this.calories = calories;
        this.fat = fat;
        this.sodium = sodium;
        this.carbohydrate = carbohydrate;
    }
}
```

필수 매개변수만 받는 생성자, 필수 매개변수와 선택 매개변수 1개를 받는 생성자, ... <u>모든 매개변수를 받는 생성자까지 늘려가는 방식</u>이다.
위 방식을 **점층적 생성자 패턴**이라고 한다. 일단 명백한 점은 <u>객체 생성의 챔임이 클래스에 있고 캡슐화되어 있어서 안정적</u>이라는 점이다.

하지만 한눈에 봐도 뭔가 잘못됐다.

### **점층적 생성자 패턴의 단점**

내가 파악한 문제점은 아래와 같다:

1. 코드 자체에서 생성자가 차지하는 비율이 과도하게 커서 **클래스 설계의 초점이 분산됨**
2. 새로운 매개변수를 추가할 때마다 **새로운 생성자를 추가해야 함**
3. **생성자 간의 중첩 호출**로 인해 수정할 때 연쇄적으로 다른 생성자들도 수정해야 할 가능성이 있음

즉, 가독성과 유지보수성 그리고 설계의 유연성을 저하시키는 결과를 초래한다.

위 설계대로 객체를 생성한다면 아래와 같은 방식으로 코드를 작성한다.

```java
NutritionFacts nutritionFacts = new NutritionFacts(240, 8, 100, 20);
```

이렇게 객체를 생성한다면 개발자가 당장은 설정하고 싶지 않은 필드까지 어쩔 수 없이 값을 할당해줘야 한다.
위 코드에서는 점층적 생성자 패턴으로 인해 `sodium`, `carbohydrate`에 각각 `0`이 할당되었다.
이때 개발자가 실수로 `sodium`, `carbohydrate`에 적절한 실제 값을 할당해주는 것을 잊고
해당 변수를 사용하게 된다면, 할당된 값인 `0`이 실제로 그러한 값인지 아니면 아직 할당되지 않은 것인지 혼동하여
예기치 않은 오류를 발생시킬 가능성이 있다.

또한 매개변수가 많아질수록 클라이언트 코드를 작성하거나 읽기 어렵다는 단점도 있다.
코드를 읽을 때 각 값의 의미가 무엇인지 헷갈릴 것이고 특히나 타입이 같은 매개변수가 연속적으로 주어진다면
IDE가 제공하는 정적 코드 분석 기능으로도 오류를 발견하기 힘들 것이다.

---

## **자바빈즈 패턴 (JavaBeans Pattern)**

선택 매개변수가 많을 때 발생하는 단점의 대안으로 **자바빈즈 패턴**이 있다.
자바빈즈 패턴은 매개변수가 없는 생성자, 즉 <u>기본 생성자로 객체를 생성한 뒤 세터(setter)로 값을 할당하는 방식</u>이다.

```java
public class NutritionFacts {

    private int servingSize = -1; // 필수; 기본값 없음
    private int servings = -1;    // 필수; 기본값 없음
    private int calories = 0;
    private int fat = 0;
    private int sodium = 0;
    private int carbohydrate = 0;

    public NutritionFacts() { }

    public void setServingSize(int servingSize) { this.servingSize = servingSize; }
    public void setServings(int servings) { this.servingSize = servings; }
    public void setCalories(int calories) { this.calories = calories; }
    public void setFat(int fat) { this.fat = fat; }
    public void setSodium(int sodium) { this.sodium = sodium; }
    public void setCarbohydrate(int carbohydrate) { this.carbohydrate = carbohydrate; }
}
```

점층적 생성자 패턴의 단점들이 자바빈즈 패턴에서는 사라졌다.
달라진 점으로는 과도한 생성자의 나열이 사라졌고 불필요한 값의 할당도 없다.
값의 할당은 세터의 책임으로 각 세터의 역할이 분명하다.
참고로 세터로 값을 할당해주어야 하기 때문에 `final` 키워드는 사라졌다.

위 방법으로 객체를 생성한다면 아래처럼 코드를 작성한다.

```java
NutritionFacts nutritionFacts = new NutritionFacts();
nutritionFacts.setServingSize(240);
nutritionFacts.setServings(8);
nutritionFacts.setCalories(100);
nutritionFacts.setFat(20);
```

점층적 생성자 패턴으로 객체를 생성했을 때보다 코드는 길어졌지만 어느 필드에 어떤 값이 할당되는지 훨씬 알기 쉽다.
다만 위 방법에는 치명적인 단점이 있다.
객체 생성의 책임이 클라이언트 코드에 있다는 것과 객체가 완전히 생성되기 전까지는 일관성(consistency)이 깨져 있다는 것이다.

### **자바빈즈 패턴의 단점 (1) - 객체 생성 책임 회피**

먼저, 객체 생성의 책임이 클라이언트 코드에 있다는 것은 아래와 같은 문제를 야기하기 쉽다:

- **코드 중복**: 동일한 객체를 생성하는 코드가 여러 곳에서 반복될 수 있음
- **생성 규칙 관리의 어려움**: 필드 간 의존관계나 제약사항, **<sup>*</sup>불변식(invariant)**이 있을 때, 모든 클라이언트 코드에서 올바르게 구현해야 함
- **캡슐화 약화**: 객체의 내부 구현이 클라이언트 코드에 노출됨

> <sup>*</sup>불변식(invariant): 프로그램이 실행되는 동안, 혹은 정해진 기간 동안 반드시 만족해야 하는 조건

### **자바빈즈 패턴의 단점 (2) - 일관성의 파편화**

일관성이 깨져 있다는 것은 객체가 유효하지 않은 중간 상태에 놓일 수 있다는 것이다.

예를 들어:

```java
NutritionFacts nutritionFacts = new NutritionFacts();
nutritionFacts.setServingSize(240);
// 이 시점에서 nutritionFacts 객체를 사용하려고 한다면?
nutritionFacts.setServings(8);
nutritionFacts.setCalories(100);
```

위 코드에서 주석으로 표시된 시점은 `servingSize`만 설정된 불완전한 객체이다.
`nutritionFacts` 객체가 완전히 초기화되지 않았을 때, 즉 유효하지 않은 중간 상태일 때 해당 객체를 사용할 수 있을까?
물론 사용할 수는 있다. 하지만 사용해도 될까? 해당 시점에는 객체의 필드가 완전히 설정되지 않은 상태이다.
이때 객체를 사용한다면 시맨틱 에러나 비즈니스 로직 오류로 이어질 가능성이 매우 크다.

이 단점의 핵심적인 것은 <u>일관성이 깨져 있는 상태 중에 다른 코드의 간섭을 방지하는 언어적 기능이 거의 없다</u>는 것이다.

---

## **빌더 패턴 (Builder Pattern)**

다행히도 점층적 생성자 패턴의 안정성과 자바빈즈 패턴의 가독성을 겸비하는 객체 생성 패턴, 빌더 패턴이 있다.
클라이언트는 필요한 객체를 직접 만드는 대신, 필수 매개변수만으로 생성자(혹은 정적 패터리 메서드)를 호출해 **빌더 객체**를 얻는다.
그 후 빌더 객체의 일종의 세터 메서드로 필드 값을 설정한다. 마지막으로 `build` 메서드를 호출하여 빌더 객체를 우리에게 필요한 인스턴스로 변환한다.

이제 빌더가 동작하는 방식을 코드로 살펴보자.

```java
public class NutritionFacts {

    private final int servingSize;
    ...
    private final int carbohydrate;

    public static class Builder {

        // 필수 매개변수
        private final int servingSize;
        private final int servings;

        // 선택 매개변수; 기본값으로 초기화
        private int calories = 0;
        ...
        private int carbohydrate = 0;

        // 필수 매개변수 초기화 생성자
        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }

        public Builder calories(int calories) {
            this.calories = calories;
            return this;
        }

        ...

        public Builder carbohydrate(int carbohydrate) { 
            this.carbohydrate = carbohydrate;
            return this;
        }

        public NutritionFacts build() { return new NutritionFacts(this); }
    }

    private NutritionFacts(Builder builder) {
        this.servingSize = builder.servingSize;
        ...
        this.carbohydrate = builder.carbohydrate;
    }
}
```

빌더 패턴은 점층적 생성자 패턴과 자바빈즈 패턴의 장점을 취합한 패턴이다.

### 빌더 패턴의 특징

빌더 패턴의 특징으로는 아래와 같다:

- 클래스의 <u>모든 필드는 불변</u>
- 빌더의 세터 메서드는 빌더를 반환하여 <u>메서드 체이닝 (method chaining)</u> 가능

따라서, 불변 클래스이고 한번에 객체 생성을 한다는 점에서 안정적이고, 적절한 이름의 세터를 통해 필드가 초기화 된다는 점에서 가독성이 좋다.

또한, `build` 메서드에서 클래스의 불변식을 검사할 수 있다는 점에서 예외 처리를 통해 객체를 안전하게 생성할 수 있다.

예를 들어, 객체의 모든 필드가 0을 포함한 양수이어야 한다는 조건을 만족해야 한다고 가정해보자:

```java
public class NutritionFacts {

    ...

    public static class Builder {

        ...

        public NutritionFacts build() {
            if (servingSize < 0 || ... || carbohydrate < 0) {
                throw new IllegalArgumentException("All parameters must be greater than or equal to 0");
            }
            return new NutritionFacts(this);
        }
    }

    ...
}
```

```java
NutritionFacts nutritionFacts = NutritionFacts.Builder(240, 8)
    .calories(100)
    .fat(20)
    .build();
```

불변식 검증을 객체 생성에서 담당하므로 클라이언트 코드는 오로지 객체 생성에만 집중할 수 있고,
무엇보다도 위 코드는 매우 읽기 쉽다.
빌더 패턴의 사소한 단점으로는 <u>빌더를 구현하는 것이</u> 매개변수의 개수에 비해 <u>지나치게 장황해질 수 있다</u>는 것이다.
이펙티브 자바에서는 매개변수가 적어도 4개 이상은 되어야 빌더 패턴을 구현하는 것이 제 값을 한다고 한다.
그럼에도 불구하고 <u>클래스는 확장 가능성을 열어두어야 하기 때문</u>에 애초부터 빌더 패턴으로 구현하는 것이 좋다고 생각한다.
더군다나 <u>Lombok의 <code>@Builder</code> 애너테이션이 빌더 패턴을 지원</u>하기 때문에 더욱 고민할 필요가 없을 것 같다.
