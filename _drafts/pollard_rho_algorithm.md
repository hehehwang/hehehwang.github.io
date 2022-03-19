---
title: pollard's rho integer factorization 폴라드 로 인수분해
categories:
  - algorithm
  - algonote
tags:
  - math
  - pollard_rho
  - Python
mathjax: true
---

# 개요
폴라드 로 알고리즘은 1975년에 John Pollard가 발명한 알고리즘입니다.\
작은 공간만을 사용하며, 피분해 숫자의 가장 작은 소수의 루트에 비례한 시간이 걸리는 것으로 알려져 있습니다.

# 배경지식
## Miler-Rabin 밀러-라빈
반드시 밀러-라빈을 이용해야 하는것은 아니지만, 보통 폴라드 로는 큰 숫자를 소인수분해하기 위해 사용되는 알고리즘이므로\
보통 밀러-라빈을 이용하여 소수 판정을 합니다.
* [밀러-라빈 소수판정법](https://hehehwang.github.io/algorithm/algonote/miller-rabin-algorithm/)

## Floyd cycle detection 플로이드 사이클 탐지
흔히 토끼-거북이법이라고 알려져있는 사이클 탐지법입니다.
* [Wikipedia](https://en.wikipedia.org/wiki/Cycle_detection#Floyd's_tortoise_and_hare)


# 설명
예를 들어 $$


# 참고자료
* [Wikipedia](https://en.wikipedia.org/wiki/Pollard%27s_rho_algorithm)
* [ruz](https://aruz.tistory.com/140)
