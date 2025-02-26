---
title: 에라토스테네스의 체 시간복잡도
date: 2025-02-18 00:00
categories: [Algorithm]
tags: [Algorithm, Math, Number Theory]
math: true
---

## **$O(\sqrt N)$ 소수 판별법의 한계**

일반적으로 입력받은 수가 소수인지를 판단하는 코드는 $O(\sqrt N)$의 알고리즘을 사용할 수 있다.

```cpp
bool is_prime(int n) {
  if (n < 2) return false;
  for (int i = 2; i * i <= n; i++) {
    if (n % i == 0) return false;
  }
  return true;
}
```

$2$부터 $\sqrt n$까지 나누어떨어지는지에 따라 주어진 수가 소수인지 판별하는 함수이다.
가장 효율적인 알고리즘은 아니지만 구현하기도 쉽고 시간복잡도도 나름 나쁘지 않기 때문이다.

하지만 여러개의 수에 대한 소수 판별을 위 방법대로 한다면 매우 비효율적일 것이다.
$1$부터 $N$까지 위 방식대로 소수의 개수를 센다면 다음과 같은 코드가 구현될 것이다.

```cpp
int cnt = 0;
for (int i = 1; i <= n; i++) {
  if (is_prime(i)) ++cnt;
}
```

$O(\sqrt N)$의 알고리즘을 $N$번 적용하므로 최종적인 시간복잡도는 $O(N \sqrt N)$이 된다.
보통 1억번의 연산을 1초로 가정하므로, $N$이 20만을 넘어가면 이 방법은 사용하기에 매우 느린 알고리즘이 된다.

## **에라토스테네스의 체**

**에라토스테네스의 체(Sieve of Eratosthenes)**는 특정 범위 내의 소수 목록을 찾는 매우 빠르고 간단한 알고리즘이다.
소수를 체로 걸러낸다고 하여 이렇게 이름이 지어졌다.
소수의 배수들을 목록에서 지워나가는 방식이다.
말로 그 과정을 설명하는 것보다 아래 이미지를 통해 이해하는 것이 쉽다.

![seive of eratosthenes](../assets/img/posts/2025-02-18-sieve-of-eratosthenes.gif)

여러 개의 소수를 판별하기 위해서는 **동적 계획법(Dynamic Programming)**을 활용해야 한다.
크기가 $N$인 `dp` 배열에 해당 인덱스의 수가 소수인지의 여부를 저장해 둔다면 특정 범위에 대한 소수 목록을 효율적으로 구할 수 있다.

## **구현**

아래는 에라토스테네스의 체를 구현하는 코드이다:

```cpp
bool dp[N + 1];
std::fill(dp + 2, dp + N + 1, true);
for (int i = 2; i * i <= N; i++) {
  if (!dp[i]) continue;
  for (int j = i * i; j <= N; j += i) dp[j] = false;
}
```

에라토스테네스의 체의 핵심은 소수의 배수를 차례로 지워나간다는 것이다.
`dp` 배열은 해당 인덱스의 수가 소수인지를 담고 있는 배열이다.
예를 들어 `dp[2] == true`이면 2는 소수, `dp[4] == false`이면 4는 합성수이다.

처음에는 배열의 모든 인덱스에 대하여 `true`를 할당한다.

```cpp
bool dp[N + 1];
std::fill(dp + 2, dp + N + 1, true);
```

첫 번째 반복문은 2부터 $\sqrt N$까지의 수에 대해서만 검사한다.

```cpp
for (int i = 2; i * i <= N; i++) {
  ...
}
```

$\sqrt N$보다 큰 수의 배수들은 이미 이전 단계에서 모두 처리되므로 검사할 필요가 없다.

현재 숫자 `i`가 이미 소수가 아니라고 판별되었다면 건너뛴다.

```cpp
if (!dp[i]) continue;
```

이는 `i`가 이전 단계에서 처리된 어떤 소수의 배수였다는 의미이다.

현재 소수 `i`의 배수들을 모두 소수가 아니라고 표시한다.

```cpp
for (int j = i * i; j <= N; j += i) dp[i] = false;
```

`j`가 `i * i`부터 시작하는 이유는 (`i * 2`, `i * 3`, ..., `i * (i - 1)`)은 이미 처리되었기 때문이다.
`i`씩 증가시키면서 $N$까지의 모든 `i`의 배수를 판별한다.

## **시간복잡도: $O(N \log \log N)$**

일단 결론부터 말하면 에라토스테네스의 체의 시간복잡도는 $O(N \log \log N)$이다.
$1, ..., N$까지의 수를 처리한다는 점에서 $O(N)$은 쉽게 납득할 수 있다.
무엇인가 $O(\log \log N)$의 시간이 걸린다는 것이다.

> 아래 증명 과정은 엄밀한 증명이 아니니 개괄적인 흐름만 파악해주시길 바랍니다.
{: .prompt-warning }

에라토스테네스의 체의 알고리즘을 생각해보자.

1. $2$의 배수 제거 → $4, 6, 8, ...$ → $\frac N 2$개
2. $3$의 배수 제거 → $6, 9, 12, ...$ → $\frac N 3$개
3. $5$의 배수 제거 → $10, 15, 20, ...$ → $\frac N 5$개
4. $7$의 배수 제거 → $14, 21, 28, ...$ → $\frac N 7$개
5. $\vdots$

각 소수에 대하여 $N$이하의 배수들을 제거하므로 소수 $p$에 대하여 약 $\frac N p$개의 제거 연산을 한다.
$\sqrt N$ 이하의 소수가 $k$개 있다고 할 때, 총 연산 횟수는 아래와 같다:

$$
N(\frac 1 2 + \frac 1 3 + \frac 1 5 + \cdots + \frac 1 p_k) = N \sum_{p \in \mathbb P}^{p \leq N} p
$$

즉, 소수의 역수의 합인 $\sum_{p \in \mathbb P}^{p \leq N} p$가 $\log \log N$으로 근사됨을 보이는 것이 핵심이다.

**조화급수(Harmonic Series)**를 생각해보자.
조화급수란 각 항의 역수가 등차수열을 이루는 급수이다.

$$
\sum_{n = 1}^{\infty} \frac 1 n = 1 + \frac 1 2 + \frac 1 3 + \frac 1 4 + \cdots
$$

조화급수는 보이는 것과 다르게 소수와 밀접하게 연관되어 있다.
조화급수를 소수에 대한 식으로 표현하면:

$$
\sum_{n = 1}^{\infty} \frac 1 n
= \prod_{p \in \mathbb P} (1 + \frac 1 p + \frac 1 {p^2} + \cdots)
= \prod_{p \in \mathbb P} \frac 1 {1 - p^{-1}}
$$

급수가 갑자기 왜 누승(누적곱)이 되는지 직관적인 이해가 힘들 수 있다.
이해를 돕기 위하여 누승을 전개하면 아래와 같다:

$$
\begin{aligned}
\prod_{p \in \mathbb P} (1 + \frac 1 p + \frac 1 {p^2} + \cdots) =
&(1 + \frac 1 2 + \frac 1 {2^2} + \frac 1 {2^3} + \cdots) \\
&(1 + \frac 1 3 + \frac 1 {3^2} + \frac 1 {3^3} + \cdots) \\
&(1 + \frac 1 5 + \frac 1 {5^2} + \frac 1 {5^3} + \cdots) \\
&(1 + \frac 1 7 + \frac 1 {7^2} + \frac 1 {7^3} + \cdots) \\
&\phantom{(1 + \frac 1 7 + \frac 1 {7^2})} \vdots
\end{aligned}
$$

이때 임의의 수를 떠올려보자.
예를 들어 그 수가 $42$이고, 소인수분해를 하면 $42 = 2 \times 3 \times 7$이다.
조화급수의 항 $\frac 1 {42} = \frac 1 2 \times \frac 1 3 \times \frac 1 7$으로 유일하게 표현되고 그 외의 방법은 없다.
누승에서 각 항의 소수의 거듭제곱에 대한 역수를 적절히 선택하면 모든 자연수를 만들 수 있다.
모든 자연수는 유일한 소인수분해 결과를 가지기 때문이다.

$$
\sum_{n = 1}^{\infty} \frac 1 n = \prod_{p \in \mathbb P} \frac 1 {1 - p^{-1}}
$$

위 식이 성립함을 보였으니, 양 변에 자연로그를 씌우면:

$$
\begin{align}
\ln \sum_{n = 1}^{\infty} \frac 1 n
= \ln \left(\prod_{p \in \mathbb P} \frac 1 {1 - p^{-1}}\right)
= \sum_{p \in \mathbb P} \ln \left(\frac 1 {1 - p^{-1}}\right)
= \sum_{p \in \mathbb P} - \ln \left(1 - p^{-1}\right)
\end{align}
$$

여기서 $\ln (1 - x)$의 테일러 급수를 활용한다.

$$
\ln (1 - x) = -\sum_{n = 1}^{\infty} \frac {x^n} n = -x - \frac 1 2 x^2 - \frac 1 3 x^3
- \frac 1 4 x^4 - \cdots , (|x| < 1)
$$

$\vert\frac 1 p \vert < 1$이므로,

$$
\begin{aligned}
-\ln(1 - p^{-1}) =
&-\left( -\frac 1 p -\frac 1 {2p^2} -\frac 1 {3p^3} - \frac 1 {4p^4} - \cdots \right) \\
&= \frac 1 p + \frac 1 {2p^2} + \frac 1 {3p^3} + \frac 1 {4p^4} + \cdots
\end{aligned}
$$

위 결과를 $(1)$식에 대입하면:

$$
\begin{align}
\ln \sum_{n = 1}^{\infty} \frac 1 n
&= \sum_{p \in \mathbb P} - \ln \left(1 - p^{-1}\right) \notag \\
&= \sum_{p \in \mathbb P} \left(\frac 1 p + \frac 1 {2p^2} + \frac 1 {3p^3} + \frac 1{4p^4} + \cdots \right) \notag \\
&= \sum_{p \in \mathbb P} \frac 1 p + \sum_{p \in \mathbb P} \frac 1 {p^2}
\left(\frac 1 2 + \frac 1 {3p} + \frac 1 {4p^2} + \cdots \right) \notag \\
&< \sum_{p \in \mathbb P} \frac 1 p + \sum_{p \in \mathbb P} \frac 1 {p^2} \left(1 + \frac 1 p + \frac 1 {p^2} + \cdots \right)
= \sum_{p \in \mathbb P} \frac 1 p + \sum_{p \in \mathbb P} \frac 1 {p(p - 1)} \notag \\
&= \sum_{p \in \mathbb P} \frac 1 p + C
\end{align}
$$

여기서 $C$는 $1$보다 작은 상수이다.

$$
\sum_{p \in \mathbb P} \frac 1 {p(p - 1)} < \sum_{n = 2}^{\infty} \frac 1 {n(n - 1)}
= \sum_{n = 2}^{\infty} \left(\frac 1 {n - 1} - \frac 1 n \right)
= 1 - \frac 1 2 + \frac 1 2 - \frac 1 3 + \cdots = 1
$$

$(2)$식으로부터 아래 식이 성립함을 보였다:

$$
\ln \sum_{n = 1}^{\infty} \frac 1 n < \sum_{p \in \mathbb P} \frac 1 p + C
$$

이때,

$$
\int_1^{\infty} \frac 1 x dx
= \lim_{x \to \infty} (\ln(x) - \ln(1))
< \lim_{x \to \infty} \ln(x)
< \sum_{n = 1}^{\infty} \frac 1 n
$$

이므로, 아래 식이 성립한다:

$$
\begin{align}
\lim_{x \to \infty} \ln \ln(x)
< \ln \sum_{n = 1}^{\infty} \frac 1 n  
< \sum_{p \in \mathbb P} \frac 1 p
\end{align}
$$

따라서 소수의 역수의 합이 발산하는 속도가 $\log \log N$에 근접함을 알 수 있다.
이때 에라토스테네스의 체에서는 $1$부터 $N$까지 $N$번 판별하므로 최종적인 시간복잡도는
$O(N \log \log N)$이다.
