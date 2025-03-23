---
title: (작성중) 파이썬 타이핑 Python typing
categories:
    - develop
    - devnote
tags:
    - Python
---

# 개요

Python은 동적언어이다.

동적언어가 무엇을 의미하는가? 프로그램을 실행시키기 전까지는 변수가 어떤 타입인지 전혀 알 수 없고, 알고 싶지도 않다는 뜻이다.

덕분에 우리는 타입이라는 거지같은 제약으로부터 벗어나 의식의 흐름대로 코딩을 할 수 있는 능력을 갖춘 대신, 라이브러리 개발자, 옆자리 동료, 혹은 과거의 나 자신으로부터 뒤통수를 맞을 위험을 노출하게 되었다.

예를 들면 `get_data_from_json` 라는 함수를 생각해보자.

이 함수는 어떤 정보를 받아서 `data`를 출력할까?

1. JSON 파일의 경로?
1. 아니면 파일 객체?
1. 혹은 JSON 형식의 문자열?
1. 그도 아니라라면 JSON 형식을 갖춘 딕셔너리?

죽음의 N지선다를 피하기 위해 당신은 어떻게 해야할까?

아마도 당신은 이 문제를 당신의 IDE가 이 문제를 멋지게 해결해 해주기를 바라며 함수에 가만히 커서를 올려다 둘 지도 모르겠다.

그 결과, 당신의 충직한 IDE는 당신의 믿음을 져버리지 않기 위해, 최선을 다해서 아래와 같은 정보를 가져왔다.

```python
(function) def get_data_from_json(src: Any) -> dict
```

이렇게 유용할수가! 당신은 이제 이 함수가 `src`라는 인자를 받는다는 사실을 알게 되었다.

그리고 그 결과, 위 4지선다 중 아무런 선지도 제외하지 못했음을 깨닫는다.

이 쯤에서 현명한 Python 개발자라면, 위 선택지 중 아무거나 넣어보고 실행해보면서 확인할 것이다. 그리고 그 반대라면 별 도움도 안 될 API 문서를 살펴보고 있을겠지만, 어느쪽이건 시간을 낭비하게 될 뿐만 아니라, 원래 하던 작업으로부터 방해받는 것은 마찬가지이다.

어떻게 하면 이 난관을 슬기롭게 해결할 수 있을까?

답은 IDE를 똑똑하게 만드는 것이다.

예를 들면, 만약 아까 IDE가 가져온 정보가 아래와 같았다면 어떨까?

```python
(function) def get_data_from_json(src: Union[str, Path]) -> dict
```

아니면

```python
(function) def get_data_from_json(src: TextIO) -> dict
```

혹은

```python
(function) def get_data_from_json(src: dict) -> dict
```

아름답지 않은가? 이렇게 아름다운 도움말을 만들어내는 것이 바로 typing이다.

# typing 사용법 (기초편)

# (아마도 몰라도 될) typing 모듈의 역사

-   Python 3.5 (typing 모듈 최초 도입)

    기본적인 타입 힌트 (List, Dict, Tuple, Optional 등) 추가

-   Python 3.6

    `TypeVar`(제네릭) 및 `Callable` 지원 강화
    `NewType` 추가 (새로운 타입 정의 가능)

-   Python 3.8

    `TypedDict` 도입 (딕셔너리에 특정 타입의 값을 요구할 수 있도록 개선)

-   Python 3.9

    `List[int]` 대신 `list[int]` 사용 가능 (타입 힌트 문법이 간결해짐)

-   Python 3.10

    `|` 연산자로 Union 타입 정의 가능 (`Union[int, str]` 대신 `int | str`)
