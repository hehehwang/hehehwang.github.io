---
title: 파이썬 코드 조각모음 01 - Pathlib, File operation
categories:
  - develop
  - devnote
tags:
  - Python
  - snippets
---

# 01. Pathlib
[Official document]()

## 이름따기
* `p.stem`
* `p.name`

## 경로따기
* 상대 경로: `p1.relative_to(p2)`

# 02. File operation
## read/write
* `f.write`
* `f.writelines`
* `f.read`
* `f.readline`

## pickle
```Python
if Path('picle.pkl').exists():
    obj = pkl.load(open("pickle.pkl", "rb"))
    
with open("pickle.pkl", "Wb") as p:
    pkl.dump(obj, p)
```

## Json
```Python

```
