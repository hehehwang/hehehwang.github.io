---
title: KMP algorithm
categories:
  - algorithm
  - algonote
tags:
  - algorithm
  - kmp
  - python
  - C
---

# 개요
KMP알고리즘(Knuth-Morris-Pratt Algorithm)은 전체 문자열에서 부분 문자열을 빠르게(`O(n+m)`) 찾을 수 있는 알고리즘이다.

참고: 
  - [나무위키](https://namu.wiki/w/%EB%AC%B8%EC%9E%90%EC%97%B4%20%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98#s-2.3)
  - [BOJ 1786](https://www.acmicpc.net/problem/1786)

# 설명
KMP는 검색에 실패했을 때, 어디까지 "맞았다 치고" 탐색을 이어나갈 수 있는지 미리 계산해둔 뒤, 검색을 이어나가도록 하는 알고리즘이다.

`A B C D A B C D A B D E`에서 `A B C D A B D`를 찾는 예시를 생각해보면, 아래와 같다.

| 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| A | B | C | D | A | B | C | D | A | B | D |
| A | B | C | D | A | B | D |  |  |  |  |
|  |  |  |  | A | B | C |  |  |  |  |

7번째 `D`에서 실패했을 때, 우리는 처음 `A`에서부터 찾기를 시작하는 대신, `A B`는 이미 "맞았다 치고" 검색을 이어나갈 수 있다.

이 "맞았다 칠 수 있는 수"를 표로 만들어서 미리 관리해둘 수 있는데,
말하자면 "n번쨰 글자에서 틀렸을 때 얼마나 맞았다 치고 탐색을 진행할 수 있는지"를 미리 저장해두는 것이다.

| 1 | 2 | 3 | 4 | 5 | 6 | 7 |
| --- | --- | --- | --- | --- | --- | --- |
| A | B | C | D | A | B | D |
| 0 | 0 | 0 | 0 | 1 | 2 | 0 |

즉, 탐색하다가 `D`에서 틀렸다면 2개를 "맞았다 치고" 3번째 문자인 `C`를 `D`와 비교하는 것이다.

이러한 형태의 배열을 보통 매칭이 실패했을 때 돌아가는 자리라고 해서 **‘실패함수’** 라고 한다.

이 실패함수를 잘 작성하는 것이 KMP의 첫 걸음이다.

## 실패 함수

실패함수를 잘 짜기는 생각보다 어렵다. 예를 들어, 아래와 같은 파이썬 코드를 생각해보자.
```python
ln = len(N)
failfunc = [0] * ln

matched = 0
for cursor in range(1, ln):
    matched = matched + 1 if N[matched] == N[cursor] else int(N[0] == N[cursor])
    failfunc[cursor] = matched
```
실패할 때마다 matched수를 비우는 식으로 구현되었는데, 이러면 아래와 같은 반례를 제대로 처리하지 못한다.
| 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| A | B | A | B | C | A | B | A | B | A | B |
| 0 | 0 | 1 | 2 | 0 | 1 | 2 | 3 | 4 | 3 | 4 |

즉, 틀렸을 때 무작정 첫 글자로 돌아가면 안되고, 기존에 작성한 실패함수를 활용해야 한다.

만약 ABAB까지 비교했는데 틀렸다면, 두 번째 A까지 "맞았다 치고" 루프를 돌려야 한다. 그 말인즉, 실패함수를 작성하는 중에도 실패함수를 활용해야 한다는 뜻이다.

정리하자면 이렇다.
1. 시작위치에 해당하는 cursor와 매칭된 개수에 해당하는 matched를 준비한다.
2. cursor + matched가 현재 위치를 나타내며, 문자열의 끝에 도달하면 끝난다.
3. cursor는 1부터 시작한다.
    1. 현재위치(cursor+matched)와 matched에 해당하는 문자를 비교한다.
        1. 만약 같다면, 
            1. 실패함수의 현재 위치에 해당하는 값에 matched + 1을 집어넣고,
            2. matched 값을 하나 늘린다.
        2. 만약 다르다면,
            1. matched를 실패함수의 matched - 1에 해당하는 값으로 변경하고,
            2.  cursor 또한 그 값만큼 전진한다.
            3. 근데 만약 matched가 0이라면, 실패함수의 -1번째에 해당하는 값에 접근하는 문제가 발생하므로, matched == 0이면 1로 변경하는 과정을 선행한다. (실패함수의 0번째 값은 0이다.)


## 매칭
실패함수를 작성한 뒤의 매칭은 간단하다. 왜냐하면 앞선 실패함수를 조금만 고치면 되기 때문이다.

1. 아까와 마찬가지로 시작 위치인 cursor와 매칭된 개수은 matched를 준비하자
2. 아까와는 달리 cursor는 0부터 시작한다.
3. 현재 위치(cursor + matched) 가 찾으려는 전체 문자열의 길이에 도달할 때까지 진행한다.
    1. 현재 위치의 문자열을 서로 비교한다.
        1. 만약 같고 matched가 부분 문자열보다 작다면,
            1. matched를 하나 늘린다.
            2. matched와 부분 문자열의 길이를 비교한다.
                1. 만약 matched가 부분 문자열의 전체 길이에 도달하였다면, 문자열을 찾았을 때의 처리를 한다.
        2. else
            1. 아까와 마찬가지로 matched가 0일때의 처리를 한다.
            2. 커서를 옮기고, matched를 실패함수의 matched-1의 값으로 교체한다.


# 코드
## Python
```python
def get_failfunc(needle: Sequence):
    ln = len(needle)
    failfunc = [0] * ln
    cursor, matched = 1, 0
    while cursor + matched < ln:
        if needle[cursor + matched] == needle[matched]:
            failfunc[cursor + matched] = matched + 1
            matched += 1
        else:
            matched = max(1, matched)
            cursor += matched - failfunc[matched - 1]
            matched = failfunc[matched - 1]

    return failfunc


def KMP(haystack: Sequence, needle: Sequence):
    lh, ln = len(haystack), len(needle)
    failfunc = get_failfunc(needle)
    cursor, matched = 0, 0
    while cursor + matched < lh:
        if matched < ln and haystack[cursor + matched] == needle[matched]:
            matched += 1
            if matched == ln:
                # needle found!
                pass

        else:
            matched = max(1, matched)
            cursor += matched - failfunc[matched - 1]
            matched = failfunc[matched - 1]
```
```c++
vector<int> getFailFunc(string &needle) {
  int ln = needle.size();
  vector<int> failfunc(ln);
  int cursor = 1, matched = 0;
  while (cursor + matched < ln) {
    if (needle[cursor + matched] == needle[matched]) {
      failfunc[cursor + matched] = matched + 1;
      matched++;
    } else {
      matched = max(1, matched);
      cursor += matched - failfunc[matched - 1];
      matched = failfunc[matched - 1];
    }
  }
  return failfunc;
}
vector<int> KMP(string &haystack, string &needle) {
  int lh = haystack.size(), ln = needle.size();
  vector<int> failfunc = getFailFunc(needle);

  int cursor = 0, matched = 0;
  while (cursor + matched < lh) {
    if (matched < ln && haystack[cursor + matched] == needle[matched]) {
      matched++;
      if (matched == ln) {
        // needle found!
      }
    } else {
      matched = max(1, matched);
      cursor += matched - failfunc[matched - 1];
      matched = failfunc[matched - 1];
    }
  }
}
```
* whitespace를 포함해서 한 줄을 가져오려면 `getline(cin, {VARIABLE})`
