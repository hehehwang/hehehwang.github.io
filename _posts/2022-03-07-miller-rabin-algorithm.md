---
title: Miller-Rabin primality test 밀러-라빈 소수 판정법
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
밀러-라빈 소수 판정법은 빠르게($$O(k\log^3 n)$$) 소수를 판별할 수 있는 알고리즘입니다.


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


# 설명

짝수가 아닌 어떤 수 $$x$$가 소수인지 궁금하다고 하면, $$x-1$$ 은 짝수이므로 2의 지수와 홀수의 곱으로 나타낼 수 있습니다.

$$
x-1 = 2^s \times d
$$

이를 페르마의 소정리에 대입해봅시다.

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

밀러-라빈 탐색은 각 항들에 대해서 0을 만족하는지 일일히 검사해서 하나라도 만족하는 경우 소수로 판정합니다.

다시 말해서 $$a^d \equiv 1 \mod x $$ 이 성립하거나, 또는 $$0\leq i<s $$에 대한 $$a^{2^id} \equiv x-1 \mod x $$가 성립한다면, $$x$$는 소수입니다.

다만, 밀러-라빈 판별법은 확률적 소수 판별법이기에 여러 $$a$$에 대해서 검사를 시행해야 하며, 숫자가 커질수록 검사를 해야하는 횟수($$a$$의 종류)도 많아지는데,\
`unsigned int`범위 내라면 `2, 3, 5, 7`,\
`unsigned long long`범위 내라면 `2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37`\
에 대해서 검사를 시행해야 한다고 알려져 있습니다.\
(하단의 [base의 개수에 대해서](#base의-개수에-대해서) 항목 참조)

## 알고리즘
0. 검사할 소수들 `base`를 준비합니다.
1. 만약 검사할 대상 `x`가 `base`에 들어있다면, `x`는 소수입니다.
2. 만약 `x`가 2로 나눠떨어진다면, `x`는 소수가 아닙니다.
3. `x-1`을 구성하는 2의 개수 `s`와 홀수 `d`를 구합니다.
4. `base`의 각 소수 `a`에 대해서
    0. 만약 `(a^d)%x == 1`이라면 다음 `base`로 넘깁니다.
    1. `0`부터 `s-1`까지의 숫자 `r`에 대해서
        1. 만약 `(a^(d*2^r))%x == x-1`이라면 다음 `base`로 넘어갑니다.
    2. 만약 여기에 도달했다면, `x`는 소수가 아닙니다.
5. 모든 검사를 통과했다면, `x`는 소수입니다.


## base의 개수에 대해서

* [Wikipedia](https://en.wikipedia.org/wiki/Miller–Rabin_primality_test#Testing_against_small_sets_of_bases)
* Carl Pomerance; John L. Selfridge; Samuel S. Wagstaff, Jr. (July 1980). "The pseudoprimes to 25·109" (PDF). Mathematics of Computation. 35 (151): 1003–1026. [doi:10.1090/S0025-5718-1980-0572872-7.](https://doi.org/10.1090%2FS0025-5718-1980-0572872-7)
* Jaeschke, Gerhard (1993), "On strong pseudoprimes to several bases", Mathematics of Computation, 61 (204): 915–926, [doi:10.2307/2153262, JSTOR 2153262](https://www.ams.org/journals/mcom/1993-61-204/S0025-5718-1993-1192971-8/)

if n < 2,047, it is enough to test a = 2;\
if n < 1,373,653, it is enough to test a = 2 and 3;\
if n < 9,080,191, it is enough to test a = 31 and 73;\
if n < 25,326,001, it is enough to test a = 2, 3, and 5;\
if n < 3,215,031,751, it is enough to test a = 2, 3, 5, and 7;\
if n < 4,759,123,141, it is enough to test a = 2, 7, and 61;\
if n < 1,122,004,669,633, it is enough to test a = 2, 13, 23, and 1662803;\
if n < 2,152,302,898,747, it is enough to test a = 2, 3, 5, 7, and 11;\
if n < 3,474,749,660,383, it is enough to test a = 2, 3, 5, 7, 11, and 13;\
if n < 341,550,071,728,321, it is enough to test a = 2, 3, 5, 7, 11, 13, and 17.
if n < 3,825,123,056,546,413,051, it is enough to test a = 2, 3, 5, 7, 11, 13, 17, 19, and 23.\
if n < 18,446,744,073,709,551,616 = 264, it is enough to test a = 2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, and 37.\
if n < 318,665,857,834,031,151,167,461, it is enough to test a = 2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, and 37.\
if n < 3,317,044,064,679,887,385,961,981, it is enough to test a = 2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, and 41.

| n | bases |
|---|---|
| `1 byte = 16` |
| 2,047 | 2 |
| `2 byte = 65536` |
| 1,373,653 | 2, 3 |
| 9,080,191 | 31, 73 |
| 25,326,001 | 2, 3, 5 |
| `4 byte (unsigned int) = 4,294,967,296` |
| 3,215,031,751 | 2, 3, 5, 7|
| 4,759,123,141 | 2, 7, 61|
| 1,122,004,669,633 | 2, 13, 1662803|
| 2,152,302,898,747 | 2, 3, 5, 7, 11 |
| 3,474,749,660,383 | 2, 3, 5, 7, 11, 13 |
| 341,550,071,728,321 | 2, 3, 5, 7, 11, 13, 17 |
| 3,825,123,056,546,413,051 | 2, 3, 5, 7, 11, 13, 17, 19, 23 |
| `8 byte (unsigned long long) = 18,446,744,073,709,551,616`|
| 318,665,857,834,031,151,167,461 | 2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37 |
| 3,317,044,064,679,887,385,961,981 | 2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41 |
| `16 byte (__int128_t) = 340,282,366,920,938,463,463,374,607,431,768,211,456` |


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
            return False
    else:
        return True
```

# 관련 문제
* BOJ
    * [5615 아파트 임대](https://www.acmicpc.net/problem/5615)

# 참고문서
* [ruz](https://aruz.tistory.com/142)
* [Wikipedia](https://en.wikipedia.org/wiki/Miller%E2%80%93Rabin_primality_test)