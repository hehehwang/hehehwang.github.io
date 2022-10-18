---
title: 2023 카카오 하반기 코딩 테스트 후기
categories:
  - algorithm
  - problem_solving
tags:
  - algorithm
  - coding_test
---

# 개요
2022년 9월 24일 토요일에서 카카오에서 진행한 블라인드 코딩 테스트 후기.

* 일시: 2022년 9월 24일 토요일 14:00 ~ 17:00, 5시간
* 문제 수: 7+1문제

난이도는 작년보다 많이 어려웠다.

# 문제 풀이
## 1번 문제
문자열 파싱 + 약간의 계산문제

한 달이 28일이라는 점을 유의하고, 날짜 더하고 뺄 때 0이 되거나 하지 않도록 주의하자.

### 답
```python
def parsedate(s):
    return map(int, s.split("."))


def solution(today, terms, privacies):
    termdata = {}
    for t in terms:
        sig, month = t.split()
        termdata[sig] = int(month)

    ty, tm, td = parsedate(today)

    answer = []
    for i, p in enumerate(privacies):
        pd, psig = p.split()
        py, pm, pd = parsedate(pd)
        pm += termdata[psig]
        pd -= 1
        if pd < 1:
            pm -= 1
            pd += 28

        while 12 < pm:
            pm -= 12
            py += 1
        
        if pm < 1:
            pm += 12
            py -= 1

        if (py, pm, pd) < (ty, tm, td):
            answer.append(i + 1)

    return answer


r = solution(
    "2020.01.01",
    ["A 12", "B 6"],
    ["2019.01.01 A", "2019.01.02 A", "2018.12.31 A"],
)
print(r)
```

## 2번 문제
그리디인데, 예시가 낚시성이라서 어려웠을 수 있다.

자꾸 틀려서 잘못 짠 줄 알았는데, 나중에 보니 min, max를 잘못 쓴 거였다.

아이디어는 이렇다. 

1. 일단 어떤 상황이든 간에 가장 멀리 있는 집부터 방문해야 한다. (가장 멀리 있는 집 * 2)
2. 그리고 일단 배달할 물건을 가득 실어서 출발하는데, 이래도 되는 이유는 가면서 마주치는 집들에다가 물건을 놓고 오면 되기 때문이다.
3. 그리고 올 때 물건을 수거해오면 된다.

즉, 갈 때 무슨 3개씩만 챙겨서 간다느니 하는 건 고려할 필요가 없다. 

### 답
```python
def solution(cap, n, deliveries, pickups):
    deliver_order = []
    pickup_order = []
    for i, d in enumerate(deliveries):
        if d:
            deliver_order.append((i + 1, d))
    for i, d in enumerate(pickups):
        if d:
            pickup_order.append((i + 1, d))

    total_deliver, total_pickups = sum(deliveries), sum(pickups)

    answer = 0
    while total_deliver + total_pickups:
        dm = deliver_order[-1][0] if deliver_order else 0
        pm = pickup_order[-1][0] if pickup_order else 0
        answer += max(dm, pm) * 2

        carrier = min(cap, total_deliver)
        while deliver_order:
            addr, order = deliver_order.pop()
            to_deliver = min(carrier, order)
            carrier -= to_deliver
            total_deliver -= to_deliver
            if order - to_deliver:
                deliver_order.append((addr, order - to_deliver))
            if not total_deliver or not carrier:
                break
        if carrier:
            raise NotImplementedError
        while pickup_order:
            addr, order = pickup_order.pop()
            to_pickup = min(cap - carrier, order)
            carrier += to_pickup
            total_pickups -= to_pickup
            if order - to_pickup:
                pickup_order.append((addr, order - to_pickup))
            if not total_pickups or carrier == cap:
                break
    return answer
  ```

## 3번 문제
완전 탐색. 더 이상의 자세한 설명은 생략한다.

itertools 사용을 강력하게 권장한다.

### 답
```python
from itertools import product


def solution(users, emoticons):
    answer = (0, 0)

    m = len(emoticons)
    discounts = [10, 20, 30, 40]
    for disc_combi in product(discounts, repeat=m):
        plus_enroll, sales = 0, 0
        for discount_threshold, wallet_threashold in users:
            total_buy = 0
            for discount, price in zip(disc_combi, emoticons):
                if discount_threshold <= discount:
                    total_buy += int(price * (100 - discount) / 100)
            if wallet_threashold <= total_buy:
                plus_enroll += 1
            else:
                sales += total_buy
        if answer < (plus_enroll, sales):
            answer = (plus_enroll, sales)

    return list(answer)


r = solution(
    [
        [40, 2900],
        [23, 10000],
        [11, 5200],
        [5, 5900],
        [40, 3100],
        [27, 9200],
        [32, 6900],
    ],
    [1300, 1500, 1600, 4900],
)
print(r)
```

## 4번 문제
더미인 부모 노드 밑에 진짜 노드인 자식이 있진 않는지를 검사하면 된다.

여기서 부모 노드와 자식 노드간에 왔다갔다 하기가 좀 어려운데, 나는 그냥 탑다운으로 매 깊이마다 살펴볼 너비를 반으로 줄여가면서 살펴보았다.

### 답
```python
from math import log2, ceil


def solution(numbers):
    answer = []
    for n in numbers:
        b = []
        while n:
            b.append(n & 1)
            n >>= 1
        full = 2 ** ceil(log2(len(b)))
        while len(b) != full:
            b.append(0)
        b = b[::-1]

        curr = len(b) // 2
        seek_width = curr // 2
        stk = [(curr, seek_width)]
        while stk:
            c, sw = stk.pop()
            if not sw:
                continue
            if not b[c]:
                if b[c - sw] or b[c + sw]:
                    answer.append(0)
                    break
            stk.append((c - sw, sw // 2))
            stk.append((c + sw, sw // 2))

        else:
            answer.append(1)

    return answer


r = solution([63, 111, 95])
print(r)
```

## 5번 문제
빡구현인데, Merge에 유니온 파인드를 적용시키려고 보니, Unmerge가 문제로다...

시간이 부족해서 못 풀었다. 약간 변명을 하자면, 작년에는 시간이 남았어서 올해도 그럴 줄 알고 점심을 좀 느긋하게 먹었다 ㅎㅎ


## 6번 문제
dddd lrlrlr uuuu 형식에 맞추면 되는데, 왼쪽 아래 모퉁이에 도착하지 못하는 경우는 dp로 돌리면 된다 카더라

난 못풀었다.

## 7번 문제
춘식이 인성에 문제가 좀 있는것 같다.

