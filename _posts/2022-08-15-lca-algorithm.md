---
title: LCA; Lowest Common Ancestor
categories:
  - algorithm
  - algonote
tags:
  - study
---

# 개요
LCA는 트리상의 두 노드간의 최소 공통 조상 노드를 찾는 알고리즘이다.

대표문제: [LCA 2](https://www.acmicpc.net/problem/11438)

LCA가 왜 필요하냐면, 트리의 경로를 효율적을 탐색할 수 있기 떄문이라고 한다.

가령 두 노드간의 최단거리를 계산하려고 한다고 하면, 각 노드의 LCA까지의 거리를 재면 되는 식이다.

알고리즘 자체는 간단하다. 예를 들어 노드 u, v간의 LCA를 구한다고 하면,\
0) 각 노드들의 깊이를 구한다.
1) u, v의 깊이를 동일하게 맞추고,
2) 깊이가 같아진 u, v를 위로 올려가면서 조상을 찾는다.

여기서 1) 2)과정을 빠르게 탐색하는 것이 LCA의 핵심이라 하겠는데,\
그 과정에서 희소 배열(Sparce table)이라는 개념이 들어간다.

이 희소배열에 대해서도 간단히 짚고 넘어가보자.

# 희소 배열 Sparce table
희소 배열은 말하자면 미리 저장해둔 분할정복, 혹은 이분탐색이라고 할 수 있다.\
진짜로 그렇다는게 아니라 느낌적으로 그렇다는 것이다.

이분탐색에서 어떤 특정한 값을 찾거나, 조건을 만족시키기 위해서\
변수를 하나씩 움직이는게 아니라 남은 공간의 반씩 움직였던 것을 기억해보자.

희소 배열도 비슷하다. 나중에 어떤 값이나 노드에 도달하기 위해 간선을 한 개씩 타는 대신,\
2의 지수단위로 움직일 수 있도록 미리 값을 저장해두는 것이다.

예를 들어 노드 u가 한 번 움직인 결과가 v1, 두 번 움직인 결과가 v2이라면, \
```c++
sparce_table[u][0] = v1;
sparce_table[u][1] = v2;
```
이런 식이 되겠다. 그리고 `sparce_table[u][2]`에는 u에서 4번 움직인 결과가 들어가는 식.

희소배열을 이용하는 문제는
1) 희소배열을 만든다.
2) 희소배열을 써먹어서 움직인다.

이 두 단계를 효율적으로 해내야 한다.\
하나씩 해결해보자.

## 희소배열 만들기
먼저 희소 배열을 만드는 부분에 대해서 생각해보면,\
간선 하나씩 움직여가면서 만들기엔 딱 봐도 비효율적이라는 생각이 들 것이다.

희소 배열을 효율적으로 만들 수 있는 방법은 지금 만들어지고 있는 희소 배열을 활용하는 것이다.

예를 들어 노드 u에서 8번 움직인 결과를 `sparce_table[u][3]`에 넣어야 한다고 생각해보자.\
그럼 u에서 진짜로 8번 움직이는게 아니라, 4번 움직인 결과에 해당하는 노드에서 \
4번 움직인 결과를 찾아서 써주면 된다.

즉, `sparce_table[u][k] = sparce_table[sparce_table[u][k-1]][k-1]`이 된다.

이게 성립되기 위해선 희소배열을 작성할 때 BFS식으로, 즉 한 깊이에 대해서 모든 노드에 대한\
작성이 이루어진 후 다음 깊이로 넘어가야 한다.

여기까지를 코드르 나타내면,
```c++
for (int p = 0; p < 20; p++)
    for (int x = 1; x <= m; x++)
        st[x][p + 1] = st[st[x][p]][p];
```
이다.

## 희소배열 써먹기
다음으로 희소배열을 잘 써먹는 과정은 숫자를 2진법으로 바꾸는 과정과 비슷하다.

예를 들어 u에서 13번 움직인 결과를 찾는다고 생각해보면,\
13번은 8+4+1이니까 희소배열에서 그에 맞게 움직이면 된다.

만약 x를 n번 움직인다고 하면, n을 끝에서부터 긁어 없애나가면서\
x를 희소배열에 따라 움직이면 된다.

말로 설명하면 뭐지 싶지만 코드로 보면 이해가 쉬울 것이다.
```c++
for (int pwr = 0; n; pwr++) {
    if (n & 1) x = st[x][pwr];
    n >>= 1;
}
```
여기까지가 희소배열에 대한 설명이다.

## 관련문제
* [합성함수와 쿼리](https://www.acmicpc.net/problem/17435)

# 설명
다시 돌아가서 LCA의 절차를 살펴보자.

0. 각 노드들의 깊이를 구한다.
1. u, v의 깊이를 동일하게 맞추고,
2. 깊이가 같아진 u, v를 위로 올려가면서 조상을 찾는다.

깊이를 맞춰서 올라가는 것이 핵심인데, 그러려면 우선 깊이를 알아야 한다. 깊이를 알려면? DFS가 딱이다.\
DFS를 돌리면서 조상 희소배열을 채우고, 채워진 희소배열을 바탕으로 1.과 2.를 수행하면 된다.

## 준비물
* `int adj[u][v]`: 인접행렬
* `int depth[u]`: 노드 u의 깊이
* `int ancestor[node][log_2(depth)]`: 노드 u의 조상 희소배열


## 절차
0. DFS로 각 노드들의 깊이를 구하면서 노드의 2^0번째 조상을 기록한다. 
1. 희소배열의 특징을 이용해 조상 희소배열을 완성한다.
2. (각 쿼리에 대해서) u, v의 깊이를 희소배열을 이용해 동일하게 맞춘다.
3. 깊이를 맞추었는데 u, v가 서로 다르다면, 깊이를 맞춰 올라가면서 LCA를 찾는다.
    1. 일단 가장 멀리 뛰어보고, 조상이 같거나 루트의 조상에 닿으면 슬금슬금 내려온다.\
    여기서 조상이 같다는 의미는 뛰어넘은 구간 안에 LCA가 존재한다는 뜻이다.
    2. 내려오다가 조상이 다르게 나올 때(LCA로부터 갈라진 이후) u, v를 그 지점으로 변경하고 계속한다.
    3. 참고로 당연하지만 이 시점에서 LCA는 2. 지점의 보폭보다 가까운 곳에 위치할 것이므로 다시 가장 멀리 뛸 필요는 없다.
    4. 조건을 u, v의 조상을 기준으로 잡았기 때문에 u, v의 부모가 LCA이다.

## 소스코드
* [LCA 2](https://www.acmicpc.net/problem/11438)의 풀이
```C++
#include <bits/stdc++.h>
// 11438 LCA 2
// https://www.acmicpc.net/problem/11438
using namespace std;
const int ROOT = 1, ROOT_PAR = -1;

int N, M, u, v;
int main(void) {
  ios::sync_with_stdio(0);
  cin.tie(0);

  cin >> N;
  vector<int> depth(N + 1);
  vector<vector<int>> adj(N + 1), ancestors(N + 1, vector<int>(30, 0));
  for (int i = 0; i < N - 1; i++) {
    cin >> u >> v;
    adj[u].push_back(v);
    adj[v].push_back(u);
  }
  stack<int> dfs_stk;
  dfs_stk.push(ROOT);
  depth[ROOT] = 1;
  ancestors[ROOT][0] = ROOT_PAR;

  while (!dfs_stk.empty()) {
    u = dfs_stk.top();
    dfs_stk.pop();
    for (int v : adj[u]) {
      if (depth[v]) continue;
      dfs_stk.push(v);
      ancestors[v][0] = u;
      depth[v] = depth[u] + 1;
    }
  }
  for (int p = 0; p < 20; p++)
    for (u = 1; u <= N; u++)
      // CAUTION! loop should end when reached to ROOT
      if (ancestors[u][p] != ROOT_PAR)
        ancestors[u][p + 1] = ancestors[ancestors[u][p]][p];

  cin >> M;
  while (M--) {
    cin >> u >> v;
    if (depth[u] < depth[v]) swap(u, v);
    int depth_diff = depth[u] - depth[v];
    for (int p = 0; depth_diff; p++) {
      if (depth_diff & 1)
        u = ancestors[u][p];
      depth_diff >>= 1;
    }
    if (u != v) {
      for (int p = 20; - 1 < p; p--) {
        if (ancestors[u][p] != ROOT_PAR &&
            (ancestors[u][p] != ancestors[v][p])) {
          u = ancestors[u][p];
          v = ancestors[v][p];
        }
      }
      // CAUTION!
      u = ancestors[u][0];
    }
    cout << u << '\n';
  }
}
```

# 생각해볼 점
* LCA의 시간복잡도는 크기가 N인 트리에서 각 쿼리당 `O(logN)`이다.
* 트리상의 두 노드간의 거리를 어떻게 구할 수 있을까?
  * 만약 각 쿼리를 BFS등을 이용해 나이브하게 처리한다면 `O(QN)`이 걸린다.
  * 하지만 LCA를 활용하면 `O(N) + O(QlogN)`만에 구할 수 있다.
    * 각 노드의 루트간의 거리를 구해놓고, (u-ROOT) + (v-ROOT) - 2*(LCA-ROOT)를 구하면 u-v간의 거리이다.
    

# 관련문제
* [백준 단계별로 풀기 - 최소 공통 조상](https://www.acmicpc.net/step/40)

# 참고자료
* Ries 마법의 슈퍼마리오
  * [LCA](https://m.blog.naver.com/kks227/220820773477)
  * [Sparce table](https://m.blog.naver.com/kks227/220793361738)