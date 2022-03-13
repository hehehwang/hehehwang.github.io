---
title: Miller-Rabin 밀러-라빈 소수 판정법
categories:
  - algorithm
  - algonote
tags:
  - math
  - miller-rabin
  - Python
  - writting
mathjax: true
---

# 개요
밀러-라빈 소수 판정법은 빠르게 소수를 판별할 수 있는 알고리즘입니다.


# 배경지식

## 빠른 거듭제곱

* [Wikipedia](https://en.wikipedia.org/wiki/Exponentiation_by_squaring)
* [남남서의 블로그](https://namnamseo.tistory.com/entry/%EB%B9%A0%EB%A5%B8-%EA%B1%B0%EB%93%AD%EC%A0%9C%EA%B3%B1)


## 페르마의 소정리

어떤 소수 $$p$$와 서로소인 $$a$$에 대해서,

$$
a^{p-1} \equiv 1 \; mod \; p
$$

단, 그 역은 성립하지 않는다.<br>
(페르마의 소정리가 성립한다고 $$p$$가 소수임을 보장하지 않음.)


# 본론

짝수가 아닌 어떤 수 $$x$$가 소수인지 궁금하다고 하면, $$x-1$$ 은 짝수이므로 2의 지수와 홀수의 곱으로 나타낼 수 있음.

$$
x-1 = 2^s \times d
$$

이를 페르마의 소정리에 대입한다.

$$
\begin{aligned}
a^{x-1} &\equiv 1 \mod x \\
a^{x-1}-1 &\equiv 0 \mod x  \\
a^{2^nd}-1 &\equiv 0 \mod x \\
(a^{2^{s-1}d}+ 1)\cdot(a^{2^{s-1}d}-1) &\equiv 0 \mod x \\
\cdots \\
(a^{2^{s-1}d}+1)\cdot(a^{2^{s-2}d}+1)\cdots(a^{2d}+1)\cdot(a^d+1)\cdot(a^d-1) &\equiv 0 \mod x
\end{aligned}
$$

밀러-라빈 탐색은 각 항들에 대해서 0을 만족하는지 일일히 검사해서 하나라도 만족하는 경우 소수로 판정한다.

다시 말해, $$a^d \equiv 0 \mod x $$ 또는 $$0\leq i<s $$에 대해서 $$a^{2^id} \equiv x-1 \mod x $$가 성립하는지 확인한다.


## [base의 개수에 대해서](https://en.wikipedia.org/wiki/Miller–Rabin_primality_test#Testing_against_small_sets_of_bases)

* [`Carl Pomerance; John L. Selfridge; Samuel S. Wagstaff, Jr. (July 1980). "The pseudoprimes to 25·109" (PDF). Mathematics of Computation. 35 (151): 1003–1026. doi:10.1090/S0025-5718-1980-0572872-7.`](https://doi.org/10.1090%2FS0025-5718-1980-0572872-7)
* [`Jaeschke, Gerhard (1993), "On strong pseudoprimes to several bases", Mathematics of Computation, 61 (204): 915–926, doi:10.2307/2153262, JSTOR 2153262`](https://www.ams.org/journals/mcom/1993-61-204/S0025-5718-1993-1192971-8/)

if n < 2,047, it is enough to test a = 2;
if n < 1,373,653, it is enough to test a = 2 and 3;
if n < 9,080,191, it is enough to test a = 31 and 73;
if n < 25,326,001, it is enough to test a = 2, 3, and 5;
if n < 3,215,031,751, it is enough to test a = 2, 3, 5, and 7;
if n < 4,759,123,141, it is enough to test a = 2, 7, and 61;
if n < 1,122,004,669,633, it is enough to test a = 2, 13, 23, and 1662803;
if n < 2,152,302,898,747, it is enough to test a = 2, 3, 5, 7, and 11;
if n < 3,474,749,660,383, it is enough to test a = 2, 3, 5, 7, 11, and 13;
if n < 341,550,071,728,321, it is enough to test a = 2, 3, 5, 7, 11, 13, and 17.

# 코드
## Python
```python
def isprime_mr(x: int):
    def modpow(base, pwr, mod):
        r = 1
        while pwr:
            if pwr % 2 == 1:
                r = (r * base) % mod
            pwr >>= 1
            base = (base * base) % mod
        return r

    bases = [2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41]
    if x in bases:
        return True
    if x % 2 == 0:
        return False
    s, d = 0, x - 1
    while d % 2 == 0:
        s += 1
        d >>= 1

    for a in bases:
        if modpow(a, d, x) == 1:
            continue
        for r in range(s):
            if modpow(a, d << r, x) == x - 1:
                break
        else:
            break
    else:
        return True
    return False
```



# 참고문서
* [ruz](https://aruz.tistory.com/142)
* [Wikipedia](https://en.wikipedia.org/wiki/Miller%E2%80%93Rabin_primality_test)