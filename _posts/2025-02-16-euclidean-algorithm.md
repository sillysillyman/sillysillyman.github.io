---
title: 유클리드 호제법
date: 2025-02-16 00:00
categories: [Algorithm]
tags: [Algorithm, Math, Number Theory]
math: true
---

## **유클리드 호제법이란**

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

## **구현**

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

혹은 간단히, 표준 라이브러리에서 `std::gcd` 함수를 제공하므로 직접 구현하지 않고도 `<numeric>` 헤더를 추가하여 사용할 수 있다.

```cpp
#include <iostream>
#include <numeric>

using namespace std;

int main() {
  cout << gcd(12, 42); // 6
}
```

## **증명**

$a = bq + r (0 \leq r < b)$일 때, $gcd(a, b) = G, gcd(b, r) = G'$라 하자.

이때 $G = G'$임을 보이면 유클리드 호제법이 참으로 증명된다.

$a = GA, b = GB$로 나타낼 수 있고, $G$는 $a, b$의 최대공약수이므로 $A, B$는 서로소이다.
이를 $a = bq + r$에 대입하면, $GA = GBq + r$이고, $r = G(A - Bq)$이다.

즉, $G$는 $r$의 약수임을 알 수 있다.
$G$는 $b, r$의 약수이고, $gcd(b, r) = G'$이므로, $G$는 $G'$의 약수이다.

$G' = mG$라고 할 때, 서로소인 두 정수 $k, l$에 대해 $GB = G'k = Gmk, G(A - Bq) = G'l = Gml$이 성립한다.
각 변을 $G$로 나누면 $B = mk, A - Bq = ml$이고, $A = ml + Bq = ml + mkq = m(l + kq)$이다.

즉,

- $A = m(l + kq)$
- $B = mk$

이므로 $m$은 $A, B$의 공약수이다.
이때 $A, B$는 서로소이므로 $m = 1$일수밖에 없다.

따라서 $G' = G$이므로 유클리드 호제법은 참으로 증명된다.
