---
title: 유클리드 호제법
date: 2025-02-16 00:00
categories: [Algorithm]
tags: [Algorithm, Math, Number Theory]
math: true
---

## 유클리드 호제법이란?

> In mathematics, the **Euclidean algorithm**, or **Euclid's algorithm**,
> is an efficient method for computing the greatest common divisor (GCD) of two integers,
> the largest number that divides them both without a remainder.
>
> **[번역]** 수학에서, **유클리드 호제법** 또는 **유클리드 알고리즘**은 두 정수의 최대공약수, 즉 나머지 없이 두 정수를 나누는 가장 큰 수를 계산하는 효율적인 방법이다.

유클리드 호제법은 최대공약수를 계산하는 매우 간단한 알고리즘이다.

> 두 양의 정수 $a, b (a > b)$에 대하여 $a = bq + r (0 \leq r < b)$이라 하면, $a, b$의 최대공약수는 $b, r$의 최대공약수와 같다. 즉,
>
> $gcd(a, b) = gcd(b, r)$
>
> $r = 0$이라면, $a, b$의 최대공약수는 $b$가 된다.

구현 코드는 아래와 같다:
```cpp
int gcd(int a, int b) {
    return b ? gcd(b, a % b) : a;
}
```

위 코드는 유클리드 호제법의 동작 방식을 재귀적으로 구현한 것이다.

- "$r = 0$이라면, $a, b$의 최대공약수는 $b$가 된다." → `b ? ... : a`
- "$a, b$의 최대공약수는 $b, r$의 최대공약수와 같다." → `gcd(b, a % b)`

`gcd` 함수가 재귀적으로 호출되면서 두번째 파라미터인 `b`가 `0`이 될 때의 `a`값이 원래 두 수의 최대공약수이다.

혹은 간단히, 표준 라이브러리에서 `std::gcd` 함수를 제공하므로 직접 구현하지 않고도 `<numeric>` 헤더를 추가하여 사용할 수도 있다.
