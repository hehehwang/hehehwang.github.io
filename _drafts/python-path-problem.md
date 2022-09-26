---
title: 파이썬 경로 문제 Python path problem
categories:
  - develop
  - devnote
tags:
  - Python
---
# 개요
파이썬 프로젝트를 진행하다보면, 가장 짜증나면서도 자주 발생하는 문제가 경로 문제이다.

예를 들어, 아래와 같은 프로젝트 경로 구성을 생각해보자.
```
my-project/
├─ main.py/
├─ test/
│  ├─ test.py
├─ common/
│  ├─ math.py
├─ data/
   ├─ sample.jpg  
```
이러한 상황에서, test.py에서 math.py를 불러와서 테스트하려면 어떻게 해야할까?

혹은, math.py에서 sample.py를 `../data/sample.jpg`로 지정하면 나중에 