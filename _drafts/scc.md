---
title: SCC; Strongly Connected Component
categories:
  - algorithm
  - algonote
tags:
  - study
---

# 잡설
퇴근하고 SCC 공부하다 중간에 잠들고 일어나서 까먹기를 어언 한달..

이제 다시는 안까먹으려고 글을 남긴다.

# 개요
SCC는 그래프에서 한 원소가 다른 원소로 갈 수 있는 가장 큰 집단을 말한다.\
즉, 한 SCC 내의 원소 u, v는 DAG 없이 서로 자유롭게 오갈 수 있어야 한다.\
어떻게 보면 좀 복잡한 사이클이라 할 수 있다.

SCC를 구하는 알고리즘으로는 대표적으로 코사라주 알고리즘(Kosaraju's algorithm)과\
타잔 알고리즘(Tarjan's algorithm)이 있다.

SCC를 다루는 글들은 많지만 막상 보면 제대로 설명하고 있는 글들은 많지 않다.

구글 검색에서 상위에 나타나는 약 십수개의 글을 살펴보았는데, 대부분 알고리즘의 절차에 대한 설명 뿐이고,\
왜 이렇게 하면 묶인 원소들이 SCC가 될 수 밖에 없는지에 대해서 다루는 글은 거의 없었다.

그런 와중에, [Ries님의 블로그 글](https://blog.naver.com/kks227/220802519976)이 제일 설명이 세부적으로 되어있어 참고했으며,\
이 글을 읽는 사람도 꼭 한번씩 읽어보길 바란다.

본 글에서는 Ries님의 글을 요약하면서 내가 개인적으로 헷갈렸던 부분, 그리고 중요한 부분에\
대해서만 다루도록 하겠다. (날로 먹겠다는 뜻)

또한, SCC는 SCC집합들 간의 위상정렬이 거의 같이 다니기 때문에,\
이에 대해서도 알아두는 것이 좋다.


# 내용
SCC는 앞서 언급하였듯이, 두 가지 큰 특징, 혹은 조건을 가진다.
1. SCC내의 두 원소 u, v는 u에서 v로, v에서 u로 갈 수 있는 경로가 있다.
2. 위 조건을 만족하는 원소들을 가지는 가장 큰 집합이다.

가장 나이브하게 생각하면, 그래프 내의 모든 원소들에 대해서 탐색을 돌려서\
서로 도달할 수 있는 원소들을 묶어주면 되는데, 이러면 탐색에만 `O(n^2)`이 걸린다.

이걸 좀 스맡뜨하게 구하면 DFS 한 번, 즉 `O(n)`만에 구할 수 있게 되는 것이다.

SCC를 구하는 방법의 핵심은, 자신의 자손들이 자신의 조상으로 갈 수 있는 경우가 하나도 없을 때\
즉, 역방향 간선이 존재하지 않을 때 SCC를 추출한다는 것이다.

여기에 추가적으로 "집합의 크기가 가장 커야 한다는 조건"을 만족하기 위해 위상과 관련된 절차가 더 들어가는 식이다.


## Tarjan's algorithm
### 준비물
* `vector<int> adj[MXN]`: 인접행렬
* `int dfs_id`: dfs tree 방문 id
* `int dfs_no[MXN]`: dfs tree 방문 순서 기록
* `int scc_id`: SCC 집합 id
* `int scc_no[MXN]`: 각 노드의 SCC 집합 번호 기록
* `stack<int> scc_stk`: dfs 방문을 하면서 들어간 stack.
* `bool done[MXN]`: 해당 노드가 이미 SCC로 뽑혔는지?

### 절차
1. DFS를 돌면서 현재 노드의 `dfs_id`를 부여한다.
2. `scc_stack`에 현재 `node`를 `push`
3. 각 자식들의 결과값 중 가장 작고 SCC로 추출되지 않은 `dfs_id`를 받아온다.
4. 만약 결과값이 없을 경우, SCC를 추출한다.
    1. `scc_stack`에서 나 자신이 나올 때 까지 `pop`하면서 각 원소들의 `done`상태를 `true`로 변경하고,\
    `scc_no`에 고유한 `scc_id`를 부여한다.
    
### 설명
* `scc_stk`에 저장되는 원소는 위상정렬이 되어있는 상태이다.\
  이 위상정렬은 SCC의 두 번째 조건, 가장 큰 집합일 것에 중요한 영향을 미친다.
* 3번에서 SCC로 추출되었는지 여부를 확인하는 이유는 교차 간선이 발생할 경우,\
  SCC로 이미 추출된 원소로 가는 간선이 있을 수 있기 때문이다.\
  `done` 배열의 용도


### 코드
```c++
bool done[MXN];
stack<int> scc_stk;
vector<int> adj[MXN];
int N, M, u, v, dfs_id, dfs_no[MXN], scc_id, scc_no[MXN];
int tarjan(int node) {
  scc_stk.push(node);
  int result = dfs_no[node] = dfs_id++;

  for (int next_node : adj[node]) {
    if (!dfs_no[next_node])
      result = min(result, tarjan(next_node));
    else if (!done[next_node])
      result = min(result, dfs_no[next_node]);
  }

  if (result == dfs_no[node]) {
    while (1) {
      v = scc_stk.top();
      scc_stk.pop();
      scc_no[v] = scc_id;
      done[v] = true;
      if (v == node) break;
    }
    scc_id++;
  }

  return result;
}
```

### 생각해볼 점
* `scc_no`가 높을수록 위상도 높을까? (위상 정렬이 된 상태로 뽑히는지)

## Kosaraju's algorithm
### 준비물


# 관련문제
* [백준 강한 연결 요소](https://www.acmicpc.net/step/43)

# 참고자료
* [Ries 마법의 슈퍼마리오](https://blog.naver.com/kks227/220802519976)
