---
title: 파이썬 ctypes 활용하기 Python with ctypes
categories:
  - develop
  - devnote
tags:
  - Python
---

# 개요
파이썬의 속도를 개선할 수 있는 방법중 하나인 ctypes에 대해 알아보자.

ctypes는 C로 작성한 `.so`(mac, linux)나 `.dll`(windows)를 이용할 수 있도록 해주는 라이브러리이다.

그리고 이렇게 C로 작성해서 코드를 돌리면 당연히 파이썬보다 빠르니까 지금 당장 속도가 아쉬운 상황이라면 부하가 많이 걸리는 함수를 C로 빼내서 작성하는 식으로 최적화를 할 수 있다.

상세한 설명은 정보의 바다에 많으니까 거기서 찾도록 하고, 이 글을 읽는 당신과 쓰는 나 모두 시간이 없으므로 최대한 빨리 사용법에 대해서만 알아보자.

# 유의점
갈 땐 가더라도 알 건 알고 가자

* 인자가 간단하다면 파이썬에서 바로 C로 넘길수 있지만, 자료가 배열이 되면 그냥 넘기기가 좀 힘들어진다. 그래서 여기서는 numpy를 이용해 간편하게 전달하는 방법에 대해서만 알아보도록 하자. 
* C랑 파이썬 사이에 왔다갔다하는 부분에서 병목이 발생할 수 있다. 그러니까 꼭 빨라지지 않을수도 있다는 이야기. numpy등 다른 최적화 방법을 먼저 사용해보고, 그래도 안되면 ctypes를 고려해보자.
* ctypes를 쓰기 전에 한번 더 고민해봐야 할 부분은, C로 작성된 부분은 디버깅하기가 까다롭다.
* 리눅스랑 윈도우에서 코드 구성이 약간 다르다. 이 부분이 좀 까다롭고 까먹기 쉬운데, 전처리자를 통해서 한방에 해결할 수 있도록 예제를 짰다.
* `.so`나 `.dll`파일 내부가 어떻게 되어있는지 파이썬 입장에서는 알 수 없으므로, 래퍼를 이용해 추상화를 한 후에 사용하면 편하다.
* 당연하지만 msvc건 gcc건 컴파일러가 있어야 한다. 나는 gcc를 이용해 예제를 테스트했다.


# TLDR;
좌표값들의 배열을 받아온 후 일정 거리 내의 좌표값들만 추려서 돌려주는 예제이다.


## C (Library)
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

long long* worker(long long* works, int work_length, int criteria) {
  long long* ret = (long long*)malloc(sizeof(long long) * work_length * 2);
  memset(ret, -1, sizeof(long long) * work_length * 2);
  long long ri = 0;

  for (int i = 0; i < work_length * 2; i += 2) {
    long long x = works[i];
    long long y = works[i + 1];
    // printf("%lld, %lld\n", x, y);
    if (x * x + y * y <= 1ll* criteria * criteria) {
      ret[ri] = x;
      ret[ri + 1] = y;
      ri += 2;
    }
  }

  return ret;
}

#ifdef _WIN32
__declspec(dllexport)long long* worker(long long* works, int work_length, int criteria);
#endif
```
## Python (Wrapper)
```python
import ctypes
import os

import numpy as np

lib = ctypes.CDLL("./clib.so" if os.name == "posix" else "./clib.dll")
f = lib.worker
f.restype = ctypes.POINTER(ctypes.c_longlong)


def _clib(nparr: np.ndarray):
    length = len(nparr)
    nparr = nparr.astype("int64")
    ret_raw = f(np.ctypeslib.as_ctypes(nparr), length, 5)
    ret = np.ctypeslib.as_array(ret_raw, shape=[length, 2])
    return ret[ret[:, 0] != -1]
```
## 설명

* 컴파일: 
  * LINUX/MAC : `gcc clib.c -o clib.so -shared`
  * WINDOWS : `gcc clib.c -o clib.dll -shared`
* #6: C -> Python에서 리턴 형식을 지정해주어야 하는데, 이 때 어떤 자료형의 포인터인지 지정해주어야 한다. 예제에선 `longlong`형식의 포인터라고 지정해주었다. (`ctypes.c_longlong`)
* #13: Python -> C으로 넘겨주는 넘파이 배열의 데이터타입(`dtype`)을 제대로 맞춰주어야 한다. 예제에선 `int64`로 맞춰주었다.
  * 당연하지만 굳이굳이 래퍼에서 다시 데이터 형식을 변환하면 시간을 까먹으니, 만들때부터 `dtype`를 잘 맞춰서 `np.array`를 생성하자. 
* #15: 리턴 받은 배열의 shape를 반드시 지정해주어야 한다.
* 배열만 까다롭지, 기본적인 정수는 아주 스무스하게 잘 주고 받아진다.


# 참고자료
* [Python ctypes Reference](https://docs.python.org/3/library/ctypes.html)
* [Numpy C-Types Foreign Function Interface](https://numpy.org/doc/stable/reference/routines.ctypeslib.html)

* [DEVOCEAN - [Python] C-확장 모듈을 사용해서 Performance 향상시키기](https://devocean.sk.com/experts/techBoardDetail.do?ID=163718)
* [DEVOCEAN - [Python] C Library 이용해서 성능 높이기(Ctypes, 1편)](https://devocean.sk.com/experts/techBoardDetail.do?ID=163835)
* [DEVOCEAN - [Python] C Library 이용해서 성능 높이기(Ctypes+Numpy, 2편)](https://devocean.sk.com/experts/techBoardDetail.do?ID=163848)