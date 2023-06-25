# Number Theory Cheatsheet

정수론과 관련된 사실들을 정리해보자.

## Euclidean Algorithm

정수 $a, b$가 주어질 때, $\operatorname{gcd}(a, b)$를 계산해 보자.
$$
\begin{align*}
a &= q_1 \cdot b + r_1 \\
b &= q_2 \cdot r_1 + r_2 \\
r_1 &= q_3 \cdot r_2 + r_3 \\
\vdots &= \vdots \\
r_{n - 3} &= q_{n - 1} \cdot r_{n - 2} + r_{n - 1} \\
r_{n - 2} &= q_n \cdot r_{n - 1} + r_n \\
r_{n - 1} &= q_{n + 1} \cdot r_n + 0
\end{align*}
$$
여기서 $r_0 = b, r_{-1} = a$라고 놓으면 $r_{i - 1} = q_{i + 1} \cdot r_i + r_{i + 1}$이라고 쓸 수 있다. 코드로 구현한다면 다음과 같이 쓸 수도 있다.

```python
def gcd_rec(a, b):
    if b == 0:
        return a
    return gcd(b, a % b)
```

재귀를 푼 버전은 다음과 같다.

```python
def gcd_iter(a, b):
    r_old = a # r_{i - 1}
    r_new = b # r_i
    while r_new:
        r_old, r_new = r_new, r_old % r_new
    return r_old
```

### Extended Euclid's Algorithm

$a, b \in \mathbb{Z}^+$가 주어질 때, $ax + by = \operatorname{gcd}(a, b)$를 풀어보자. 앞서 소개한 euclidean algorithm의 동작 과정을 생각하면
$$
\begin{align*}
a &= q_1 \cdot b + r_1 \\
b &= q_2 \cdot r_1 + r_2 \\
r_1 &= q_3 \cdot r_2 + r_3 \\
\vdots &= \vdots \\
r_{n - 3} &= q_{n - 1} \cdot r_{n - 2} + r_{n - 1} \\
r_{n - 2} &= q_n \cdot r_{n - 1} + r_n \\
r_{n - 1} &= q_{n + 1} \cdot r_n + 0
\end{align*}
$$
새로 계산되는 $r_1, r_2, \dots$ 값들은 결국 $a, b$에 대한 어떤 integer combination이다.
$$
\begin{align*}
r_1 &= a - q_1 \cdot b \\
r_2 &= b - q_2 \cdot r_1 = b - q_2(a - q_1 \cdot b) \\
r_3 &= r_1 - q_3 \cdot r_2 \\
\vdots &= \vdots \\
r_{n - 1} &= r_{n - 3} - q_{n - 1} \cdot r_{n - 2} \\
r_n &= r_{n - 2} - q_n \cdot r_{n - 1} \\
0 &= r_{n - 1} - q_{n + 1} \cdot r_n
\end{align*}
$$
그렇다면, $r_i$ 값들이 어떤 integer combination인지를 추적하면 방정식을 풀 수도 있을 것이다. $r_i = a \cdot s_i + b \cdot t_i$가 되도록 $s_i, t_i$를 정의하면, $r_i$와 정확히 같은 점화식을 $s_i, t_i$에 대해 사용할 때 방정식을 풀 수 있을 것이다. 이 아이디어를 코드로 구현하면 다음과 같다.

```python
def xgcd(a, b):
    r_old, r_new = a, b
    s_old, s_new = 1, 0
    t_old, t_new = 0, 1
    while r_new:
        q = r_old // r_new
        r_old, r_new = r_new, r_old % r_new
        s_old, s_new = s_new, s_old - q * s_new
        t_old, t_new = t_new, t_old - q * t_new
    return (r_old, s_old, t_old)
```

`r_old`는 앞서와 같이 $\operatorname{gcd}(a, b)$를, `s_old`, `t_old`는 $ax + by = \operatorname{gcd}(a, b)$의 해를 나타낸다.

이 방정식은 Bézout's identity라고도 부르고, modulo inverse를 구하는 데에 사용할 수 있다.

$\operatorname{gcd}(a, b) = 1$이고, $\text{mod}\; b$에서 $a$의 modulo inverse가 $x$라 할 때 어떤 수 $y$에 대해 $ax + by = 1$이기 때문이다.

## Fermat's Little Theorem

소수 $p$와 정수 $a$에 대해, 다음이 성립한다.
$$
a^p \equiv a \quad (\text{mod}\; p)
$$

## Euler's $\varphi$

Euler's totient function, $\varphi(n)$은 $1 \leq k \leq n$인 $k$ 중에서 $n$과 coprime인 것의 개수를 세는 함수이다.

$\varphi(n)$은 multiplicative function이고, coprime $n, m$에 대해 다음이 성립한다.
$$
\varphi(nm) = \varphi(n) \varphi(m)
$$
조금 더 일반화하면 다음과 같이 쓸 수도 있다.
$$
\varphi(n)
= n \prod_{p | n} \left( 1 - \frac{1}{p} \right)
= p_1^{k_1 - 1} (p_1 - 1) p_2^{k_2 - 1} (p_2 - 1) \dots p_r^{k_r - 1} (p_r - 1)
$$
또한 Fermat's little theorem과 같이, Euler's formula를 다음과 같이 쓸 수 있다. $a$와 $n$이 coprime일 때,
$$
a^{\varphi(n)} \equiv 1 \quad (\text{mod}\; n)
$$
Textbook RSA에서 이 사실이 사용된다.

## Chinese Remainder Theorem

## Quadratic Residue