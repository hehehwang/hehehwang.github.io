---
title: fast exponential 빠른 거듭제곱
categories:
  - algorithm
  - algonote
tags:
  - math
  - divide_and_conquer
  - C
  - Python
  - writing
---
# 개요
빠른 거듭제곱법은 분할정복을 이용해 빠르게 거듭제곱을 계산하는 방법입니다.
주로 수학이나 암호와 관련된 알고리즘들, 예를 들어 이항 계수나 소인수분해를 할 때에 사용됩니다.

# 본론
(대충 일반적으로 쓰이는 거듭제곱 대한 설명)\
(대충 pow와 실수에 대한 설명)

그러나, 분할정복을 이용하면 거듭제곱을 `logN`만에 계산할 수 있습니다.\
(대충 재귀를 이용한 분할정복)

하지만 재귀로 돌리면 call stack이 쌓이는 문제도 있고\
특히 `Python`과 같은 언어는 재귀랑 별로 안 친한것도 있으니\
`while`문을 이용한 빠른 거듭제곱이 주로 많이 쓰이니 알아둡시다.

# 코드
# Python
```python
def modpow(base: int, pwr: int, mod: int):
    r, base = 1, base%mod
    while pwr:
        if pwr&1:
            r = (r * base) % mod
        pwr >>= 1
        base = (base * base) % mod
    return r
```

# C++
```c++
long long modpow(long long base, long long pwr, long long mod) {
    __int128_t r = 1;
    while (pwr > 0) {
        if (pwr & 1)
            r = (r * base) % mod;
        pwr >>= 1;
        base = (base * base) % mod;
    }
    return r;
}
```

# 참고자료
* [Wikipedia](https://en.wikipedia.org/wiki/Exponentiation_by_squaring)
* [남남서의 블로그](https://namnamseo.tistory.com/entry/%EB%B9%A0%EB%A5%B8-%EA%B1%B0%EB%93%AD%EC%A0%9C%EA%B3%B1)