---
title: 파이썬 객체 복사의 기술 Art of python copy
categories:
  - develop
  - devnote
tags:
  - Python
  - snippets
---

# 개요
파이썬을 다루다보면 처음 충격을 받고 골치를 썩히는 부분이 바로 '객체의 복사'이다.

이에 대한 세부 설명은 'mutable vs immutable'이나 'shallow copy vs deep copy'등으로 찾아보면 관련 자료가 빽뺵할테니 대략적으로 이야기 해보자면, 이렇다.

```Python
lst1 = [1, 2, 3]
lst2 = lst1
lst2[2] = 4
print(lst1)
print(lst2)
'''
[1, 2, 4]
[1, 2, 4]
'''
```
위 출력 결과는 짐작하다시피, 연속된 `[1, 2, 4]`이다.\
그리고 이 결과에 대해서 참조복사가 일어나는 mutable 객체의 특성을 모르는 입문자들은 (암시적인 참조복사에 의해)'mutable reality shock'에 빠지는 것이다.

특히 코드가 복잡해지면 이렇게 은근슬쩍 일어나는 참조복사에 엿을 먹기가 쉬우니, 항상 조심해야 한다. ([이런거](https://stackoverflow.com/questions/43255481/python-default-list-in-function) 라던지?)

그렇다면 mutable 객체를 복사하는 바른 방법은 무엇일까?

좋은 접근 방법중 하나는 객체에서 제공하는 `copy()` 메소드를 이용하는 것이다.

위 코드를 예로 들자면, 이렇게 된다.
```Python
lst1 = [1, 2, 3]
lst2 = lst1.copy()
lst2[2] = 4
print(lst1)
print(lst2)
'''
[1, 2, 3]
[1, 2, 4]
'''
```

근데 사실 mutable 객체를 복사하는 방법은 한 가지가 아니다. 예를 들면, 널리 알려진 방법중에선 이런 방법도 있다.\
`cpy = lst[:]`

이러면 list의 shallow copy를 수행한다.

그럼 이 쯤에서 드는 의문으로는, '과연 어떤 복사방법이 가장 빠를까?' 이다.

물론 그렇게 대동소이하게 차이나진 않을 것 같지만, 또 안 그래도 느린 언어에서 조금이라도 빠른 방법을 찾으려고 하는건 자연스러운 이치 아닐까?

거기에 파이썬은 참으로 신비로운 언어라서 global에서 실행시키냐 아니냐도 실행속도에 차이를 보이기 때문에 돌다리를 뽀개지 않으면 못참게 되는 것이다.

# 실험
가장 대표적인 mutable 객체인 list를 복사하는 여러가지 방법을 실험한다.

실험 환경:
- Macbook M1
- Python 3.9.6

셋업:
```Python
from copy import deepcopy, copy;
arr = list(range(int(10e3)))
```

1. reference copy `cpy = arr`
2. shallow copy `cpy = arr[:]`
3. shallow copy `cpy = arr.copy()`
4. shallow copy `cpy = copy(arr)`
5. deep copy `cpy = deepcopy(arr)`
6. shallow copy `cpy  = [arr[i] for i in range(len(arr))]`
7. shallow copy `cpy = []; for i in arr:\    cpy.append(i)`
8. shallow copy `cpy = [0]*len(arr)\for i in range(len(arr)):\    cpy[i] = arr[i]`

실험 결과:
```
statement 1 : 6.545899999999948e-05
statement 2 : 0.19917220800000002
statement 3 : 0.20795312500000002
statement 4 : 0.206773041
statement 5 : 25.428860292
statement 6 : 2.6137173750000002
statement 7 : 2.8815743749999996
statement 8 : 3.2975386670000013
```

## 결론
reference copy < shallow copy < shallow copy(대입) << deep copy 순으로 오래걸린다.

생각했던 것보다 별로 흥미로운 결과가 나오진 않았지만, deep copy가 생각보다 연산량이 많으므로 shallow copy로 충분한 작업은 반드시 shallow copy를 이용하도록 해야겠다.
