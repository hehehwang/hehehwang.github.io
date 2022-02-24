---
title: BFS와 DFS
categories:
  - algorithm
  - algonote
tags:
  - BFS
  - DFS


description: 상세설명
---

그래프 탐색의 기본인 BFS와 DFS에 대해서 알아봅시다.

# 개요

DFS와 BFS는 알고리즘 풀이의 시작이라고도 할 수 있다.


[[실전 알고리즘] 0x09강 - BFS](https://blog.encrypted.gg/941?category=773649)

[[실전 알고리즘] 0x0A강 - DFS](https://blog.encrypted.gg/942?category=773649)

# 특징

- BFS의 경우에는 큐, DFS는 스택(혹은 재귀)가 필요하다.
- 방문했는지 여부를 판별할 수 있는 배열이 있어야 한다.
- 진짜 그래프가 주어지기 보다는 2차원 배열 형태의 그래프도 많이 주어지는데, 이 경우에는 dy, dx 배열을 통해 순차적으로 방문한다.

# 구현

## BFS

### C++

```cpp
queue<pair<int, int>> Q;
while (!Q.empty()) { // Q.size()로 쓰는 사람도 있다.
      int vr, vc;
      tie(vr, vc) = Q.front();
      Q.pop();
      FOR(k, 0, 4) {
         int nr = vr + "1012"[k] - '1'; // dr 배열을 선언한 뒤, dr[k]를 더하는게 일반적.
         int nc = vc + "2101"[k] - '1';
         if (nr < 0 || nc < 0 || R <= nr || C <= nc) // out of bound 검사
            continue;
         if (vis[nr][nc]) continue; // 방문 검사
         // 방문시 해야할 일
         Q.push({ nr, nc });
      }
   }
```

- 점을 기준으로 반 시계 방향으로 돌면서 탐색한다.

### python

```python
while Q:
    v = Q.popleft()
    for i in range(4):
        nr, nc, brk = v[0]+dr[i], v[1]+dc[i], v[2]
        if oob(nr, nc): continue
        if brk:
            if board[nr][nc]: continue
            if vis[1][nr][nc]: continue
            vis[1][nr][nc] = vis[1][v[0]][v[1]] + 1
            Q.append((nr, nc, 1))
        else:
            if board[nr][nc]: brk=1
            if vis[brk][nr][nc]: continue
            vis[brk][nr][nc] = vis[0][v[0]][v[1]] + 1
            Q.append((nr, nc, brk))
```

- 큐에 위치 정보 외에 brk라는 새로운 정보를 담은 형태의 BFS이다.

## DFS

DFS는 재귀로도 돌릴 수 있고, stack을 사용해서 돌릴수도 있다.

단, stack를 통해 돌릴 때와 재귀로 돌릴 때와는 방문 순서가 약간 달라진다.