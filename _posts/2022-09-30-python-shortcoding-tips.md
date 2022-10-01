---
title: 내가 보려고 모은 파이썬 숏코딩 팁들 
categories:
  - algorithm
  - algonote
tags:
  - Python
---
# 개요
왜 백준에 보면 숏코딩 그런거


# 팁
## `open(0)`
open(0)을 하면 stdin을 받아올 수 있다.

`*open(0)`으로 unpackking도 가능
`print(list(map(int, open(0))))` 이런것도 잘 된다

## `exec("script;" * int(input()))`
코드를 여러번 반복해야 할 때 쓰면 좋다.

`exec` 내부에서 `input()`을 받아오는 것도 가능하다.

예시:
https://www.acmicpc.net/problem/3059
```python
for _ in range(int(input())):
    print(2015 - sum(map(ord, set(input()))))
```
를
```python
exec("print(2015 - sum(map(ord, set(input()))));" * int(input()))
```
로 줄일 수 있다.

## 문자열 병합
`v`의 값이 0이면 'even', 1이면 'odd'를 출력해야 하는 문제가 있으면

`"eovdedn"[n%2::2]` 하는 식으로 줄일 수 있다.

이를 잘 응용하면 k의 배수에 따라서 길이 n의 문자열이 나오도록 합칠 수 있다.

## 타입 캐스팅

* bool -> int:
    * `int({bool_value})` (x)
    * `+({bool_value})` (o)