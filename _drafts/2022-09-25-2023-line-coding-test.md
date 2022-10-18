---
title: 2023 라인 하반기 코딩 테스트 후기
categories:
  - algorithm
  - problem_solving
tags:
  - algorithm
  - coding_test
---

# 개요
2022년 9월 24일 토요일에서 라인에서 진행한 블라인드 코딩 테스트 후기.

* 일시: 2022년 9월 24일 토요일 10:00 ~ 12:30, 2시간 30분
* 문제 수: 5문제
* 언어: C, C++, Java, Python3

작년과 동일하게 오전에는 라인, 오후에는 카카오가 보는 식으로 메신저 기업 채용의 날이 구성되었다. (두 회사간의 협의가 있는걸까?) 오전에 시험을 보는 만큼 전날 잘 자는것도 중요한듯.

문제에 대한 저작권은 라인과 프로그래머스에 있다.

# 문제 풀이
## 1번 문제
C++의 벡터와 비슷한 자료구조들에서 [Capacity를 정할 때 쓰는 방법](https://boycoding.tistory.com/236)에 대한 문제이다.

요는 size의 log2의 ceil를 구하는 것이다. 예를 들어 size가 5인 배열은 log2를 씌우면 2.xx이므로 2의 3승인 8개까지 담을 수 있다.

난이도는 브론즈 상위, 관련 태그는 수학, 구현이다.

### 답
```python
from typing import List
from math import log2, ceil


def get_capacity(v):
    return 2 ** ceil(log2(v))


def solution(queries: List[List[int]]) -> int:
    answer = 0
    arr_size = [0] * 1010
    for a, v in queries:
        next_size = arr_size[a] + v
        if arr_size[a] and get_capacity(arr_size[a]) < next_size:
            answer += arr_size[a]
        arr_size[a] = next_size

    return answer

print(solution([[2, 10], [7, 1], [2, 5], [2, 9], [7, 32]]))
```

## 2번 문제
문자열간의 비교를 다루는 문제였다.

메신저 회사들은 이런 식으로 문자열을 다루는 문제는 꼭 내는 것 같고, 작년 라인 문제에서도 비슷한 문제가 출제되었던 걸로 기억한다.

문제의 핵심은 와일드카드에 해당하는 `.`문자열을 처리하는 방법이었는데, 이 `.`은 정규표현식에서 `[a-z]{1, k}`에 해당하는 것으로 생각하면 된다.

중요한 점은 반드시 1회 이상 아무 문자열이 와야 한다는 점이다. slang에 해당하는 문자열이 없이 건너뛸 수 없다.

여러 풀이가 있을 수 있겠으나, 나는 재귀로 풀이를 작성하였다. 이외에는 직접 정규표현식을 작성해서 풀면 더 간결하고 세련되게 작성할 수 있을 것 같다.

난이도는 실버 2~4, 관련 태그는 문자열, 정규표현식, 재귀이다.

### 답
```python
from typing import List


def is_slang_check(word, slang, k):
    if word == slang:
        return True
    if "." not in word:
        return False
    return __check(word, 0, slang, 0, k)


def __check(word, wi, slang, si, k):
    if si == len(slang) and wi == len(word):
        return True
    if si == len(slang) or wi >= len(word):
        return False

    if word[wi] == slang[si]:
        return __check(word, wi + 1, slang, si + 1, k)
    if word[wi] == ".":
        c = [__check(word, wi + 1, slang, si + i, k) for i in range(1, k + 1)]
        return any(c)

    return False


def solution(k: int, dic: List[str], chat: str) -> str:
    answer = ""
    results = []
    for word in chat.split():
        isslang = False
        for d in dic:
            if is_slang_check(word, d, k):
                isslang = True
                break
        results.append("#" * len(word) if isslang else word)

    answer = " ".join(results)
    return answer

r = solution(2, ["slang", "badword"], "badword ab.cd bad.ord .word sl.. bad.word")
print(r)
```

## 3번 문제
이 문제는 잘못 접근하기 쉬운, 혹은 그러한 접근이 유도된 문제이다.

예시를 보면 마을 전체에 대해서 매 분마다 온도를 m분까지 갱신시켜 나가는데, 제한사항이 `1<=n<=30`, `1<=m<=1,000,000,000`이라서 최악의 경우 `900,000,000,000`번의 연산을 해야 한다.

따라서 예시처럼 풀면 안되고, 격자의 각 점에 대해서 불이랑 만나는 시간, 얼음이랑 만나는 시간을 계산한 후 최종 온도가 얼마가 될지 계산해야한다.

예를 들어 마을의 한 위치에서 3분에 불의 영향권 내에 들어오고 8분 시점에 온도를 계산해야 한다면, 이 지점의 온도는 5도라고 계산할 수 있다.

마찬가지로 3분에 불, 5분에 얼음, 7분에 불, 10분에 계산한다고 하면 2+3=5도가 된다.

m에 해당하는 숫자가 커서 Python 외의 언어로 푼다면 자료형에 신경을 써야 할 것 같다.

참고로 본 문제에서 얼음 토템의 영향권을 계산하는 거리 공식은 맨해탄 거리(Manhattan distance), 혹은 [택시캡 거리(Taxicab distance)](https://en.wikipedia.org/wiki/Taxicab_geometry)라고 한다.

난이도는 실버 상위, 관련 태그는 시뮬레이션이다.

### 답
```python
from typing import List


def firedist(d1, d2):
    if d1 == d2:
        return 1
    return max(abs(d1[0] - d2[0]), abs(d1[1] - d2[1]))


def icedist(d1, d2):
    if d1 == d2:
        return 1
    return abs(d1[0] - d2[0]) + abs(d1[1] - d2[1])


def solution(
    n: int, m: int, fires: List[List[int]], ices: List[List[int]]
) -> List[List[int]]:
    answer = [[0] * n for _ in range(n)]
    for r in range(n):
        for c in range(n):
            for fr, fc in fires:
                fd = firedist((fr - 1, fc - 1), (r, c))
                if fd <= m:
                    td = m - fd + 1
                    answer[r][c] += td
            for ir, ic in ices:
                icd = icedist((ir - 1, ic - 1), (r, c))
                if icd <= m:
                    td = m - icd + 1
                    answer[r][c] -= td

    return answer
```

## 4번 문제
은근히 난이도가 있었던 문제로, 잘은 모르지만 1차 테스트의 컷은 여기서 이뤄지지 않았을까? 싶다.

알고리즘적으로 어렵다기 보다는 시간의 압박을 받으면서 깔끔하게 풀어내기가 어려운 문제로, 문제 난이도에 비해서 받는 압박은 컸을 것 같다.

문제 자체는 전형적인 탐색 문제이지만, 탐색에 걸리는 제약조건들이 답답하게 되어있다.

중간에 실수를 조금 해서 더럽지만, 풀이에서 주목해서 봐야할 부분이 있는 문제는 아니었기에 그냥 올린다.

난이도는 골드 하위, 관련 태그는 탐색, 구현이다.

### 답
```python
from collections import deque
from typing import List


def solution(wall: List[str]) -> List[List[int]]:
    wr, wc = len(wall), len(wall[0])
    ans = [[-1] * wc for _ in range(wr)]
    for r in range(wr):
        for c in range(wc):
            if wall[r][c] != "H":
                ans[r][c] = 0

    bfsq = deque([(wr - 1, 0, 1)])
    ans[wr - 1][0] = 1

    def oobc(v):
        return v < 0 or wc <= v

    def oobr(v):
        return v < 0 or wr <= v

    while bfsq:
        cr, cc, cm = bfsq.popleft()
        nm = cm + 1
        # basic movement
        for k in range(4):
            rd, cd = [0, 1, -1, 0][k], [1, 0, 0, -1][k]
            nr, nc = cr + rd, cc + cd
            if oobc(nc):
                continue
            if oobr(nr):
                continue
            if wall[nr][nc] != "H":
                continue
            if ans[nr][nc] != -1:
                continue
            ans[nr][nc] = nm
            bfsq.append((nr, nc, nm))

        # horizontal movement
        if 1 <= cr:
            for cd in [-3, -2, 2, 3]:
                nr, nc = cr, cc + cd
                if oobc(nc):
                    continue
                if wall[nr][nc] != "H":
                    continue
                if ans[nr][nc] != -1:
                    continue
                l, r = min(cc, nc), max(cc, nc)
                obstacle = False
                for c in range(l + 1, r):
                    if wall[cr][c] != ".":
                        obstacle = True
                        break
                for c in range(l, r + 1):
                    if wall[cr - 1][c] != ".":
                        obstacle = True
                        break

                if obstacle:
                    continue

                ans[nr][nc] = nm
                bfsq.append((nr, nc, nm))

        # vertical
        nr, nc = cr - 2, cc
        if not oobr(nr) and wall[nr][nc] == "H" and wall[nr + 1][nc] == ".":
            if ans[nr][nc] == -1:
                ans[nr][nc] = nm
                bfsq.append((nr, nc, nm))

        # diag
        nr, nc = cr - 1, cc + 1
        if (
            not oobr(nr)
            and not oobc(nc)
            and wall[nr][nc] == "H"
            and wall[nr][cc] == "."
            and wall[cr][nc] == "."
        ):
            if ans[nr][nc] == -1:
                ans[nr][nc] = nm
                bfsq.append((nr, nc, nm))

    return ans


r = solution(["....HH", "H..H.H"])
print(r)
```

## 5번 문제
테스트 시간동안에는 못 풀었는데, 아마 최대공약수로 어떻게 잘 후보군을 추려내는 식으로 풀 수 있을 것 같은데 확실하지는 않다.
