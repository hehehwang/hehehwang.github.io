---
title: 파이썬 타이핑 Python typing
categories:
  - develop
  - devnote
tags:
  - Python
---
# 개요
Python은 동적언어이다.

동적언어가 무엇을 의미하는가. 프로그램을 실행시키기 전까지는 변수가 어떤 타입인지 전혀 알 수 없고, 알고 싶지도 않다는 뜻이다.

덕분에 우리는 의식의 흐름대로 코딩을 할 수 있는 능력을 갖춘 대신, 라이브러리 개발자, 옆자리 동료, 혹은 과거의 나 자신으로부터 뒤통수를 맞을 위험을 노출하게 되었다. 


```python
```

1. 동적언어의 특징
1. 동적언어의 단점의 예시
1. 이를 해결하기 위한 typing의 필요성
1. 사용법에 대한 대략적인 설명
1. 예시
1. 구체적인 설명과 각 예시


typing 모듈의 발전 과정

🔹 Python 3.5 (typing 모듈 최초 도입)
	•	기본적인 타입 힌트 (List, Dict, Tuple, Optional 등) 추가

🔹 Python 3.6
	•	TypeVar(제네릭) 및 Callable 지원 강화
	•	NewType 추가 (새로운 타입 정의 가능)

🔹 Python 3.8
	•	TypedDict 도입 (딕셔너리에 특정 타입의 값을 요구할 수 있도록 개선)

🔹 Python 3.9
	•	List[int] 대신 list[int] 사용 가능 (타입 힌트 문법이 간결해짐)

🔹 Python 3.10
	•	| 연산자로 Union 타입 정의 가능 (Union[int, str] 대신 int | str)