---
title: 내가 보려고 모은 PS용 code snippets
categories:
  - algorithm
  - algonote
tags:
  - study
---

# 최단거리 알고리즘
## Dijkstra
### Python
```python
from heapq import heappop, heappush

adj = [[] for _ in range(N)]
for _ in range(M):
    s, e, c = map(int, input().split())
    adj[s].append((c, e))

INF = int(1e9)+1
dijk = [(0, st)]
dist = [INF]*N
dist[st] = 0
while dijk:
    c, v = heappop(dijk)
    if dist[v] != c: continue
    for nc, nv in adj[v]:
        nc += c
        if dist[nv] <= nc: continue
        heappush(dijk, (nc, nv))
        dist[nv] = nc
print(dist[en])
```

### C++
```c++
typedef pair<int, int> PII;

const int INF = 1e9 + 1;
void dijk(vector<PII>* adj, int* dist, int S) {
    priority_queue<PII, vector<PII>, greater<>> pQ;

    pQ.push({0, S});
    dist[S] = 0;
    while (pQ.size()) {
        int c, v;
        tie(c, v) = pQ.top();
        pQ.pop();
        if (dist[v] != c) continue;
        for (PII p : adj[v]) {
            int nc, nv;
            tie(nc, nv) = p;
            nc += c;
            if (dist[nv] <= nc) continue;
            pQ.push({nc, nv});
            dist[nv] = nc;
        }
    }
}
```

## Bellman-Ford
### C++
```c++
  const int INF = 1e9 + 1;
  fill(dist, dist + 502, INF);
  dist[1] = 0;
  for (int i = 0; i < N; i++) {
      for (int st = 1; st <= N; st++) {
          for (auto &p : adj[st]) {
              int c, en;
              tie(c, en) = p;
              if (dist[st] == INF || dist[en] <= dist[st] + c) continue;
              if (i == N - 1) minusCycle++;
              dist[en] = dist[st] + c;
          }
      }
  }
```

## Floyd-Warshall
### C++
```c++
const int INF = 1e9 + 2;
for (int i = 0; i < 102; i++)
        fill(floyd[i], floyd[i] + 102, INF);
for (int i = 0; i <= N; i++) floyd[i][i] = 0;

for (int k = 1; k <= N; k++) {
    for (int i = 1; i <= N; i++) {
        for (int j = 1; j <= N; j++) {
            int bypass = floyd[i][k] + floyd[k][j];
            floyd[i][j] = min(floyd[i][j], bypass);
        }
    }
}
```


## SPFA
(TODO)

# Segment Tree
(TODO)

# Strongly Connected Component
## Kosaraju
### C++
```c++
#include <bits/stdc++.h>
// 2-SAT-4
// https://www.acmicpc.net/problem/11281
using namespace std;

const int MXV = 20'020;
vector<int> adj[MXV], radj[MXV], vis(MXV);
int sccid, z, x, N, M;
stack<int> stk;
// 1 -> 1, -1 -> 2, ...
int neg(int v) {
  return v & 1 ? v + 1 : v - 1;
}
int iabs(int v) {
  return 0 < v ? v * 2 - 1 : -2 * v;
}
void dfs(int v) {
  vis[v] = 1;
  for (auto nv : adj[v]) {
    if (vis[nv]) continue;
    dfs(nv);
  }
  stk.push(v);
}
void rdfs(int v) {
  vis[v] = sccid;
  for (auto nv : radj[v]) {
    if (vis[nv]) continue;
    rdfs(nv);
  }
}
int main(void) {
  ios::sync_with_stdio(0);
  cin.tie(0);
  cin >> N >> M;
  for (int i = 0; i < M; i++) {
    cin >> z >> x;
    z = iabs(z);
    x = iabs(x);
    adj[neg(z)].push_back(x);
    adj[neg(x)].push_back(z);
    radj[z].push_back(neg(x));
    radj[x].push_back(neg(z));
  }
  for (int i = 1; i <= 2 * N; i++)
    if (!vis[i]) dfs(i);
  fill(vis.begin(), vis.end(), 0);
  while (!stk.empty()) {
    x = stk.top();
    stk.pop();
    if (vis[x]) continue;
    sccid++;
    rdfs(x);
  }
  vector<pair<int, int>> scc_node;
  int ans1 = 1;
  for (int i = 1; i <= N; i++)
    if (vis[2 * i] == vis[2 * i - 1]) ans1 = 0;
  cout << ans1 << "\n";
  if (!ans1) return 0;

  vector<int> ans2(N + 1, -1);
  for (int i = 1; i <= 2 * N; i++)
    scc_node.push_back({vis[i], i});
  sort(scc_node.begin(), scc_node.end());

  for (auto [sid, n] : scc_node) {
    x = (n + 1) / 2;
    // positive면 0을, negative면 1을 담는다
    if (ans2[x] == -1)
      ans2[x] = 1 - (n & 1);
  }
  for (int i = 1; i <= N; i++) cout << ans2[i] << " ";
}
```


## Tarjan
### C++
```c++
bool done[MXN];
stack<int> scc_stk;
vector<int> adj[MXN];
int N, M, u, v, dfs_id, dfs_no[MXN], scc_id, scc_no[MXN];
int tarjan(int node) {
  scc_stk.push(node);
  int result = dfs_no[node] = dfs_id++;

  for (int next_node : adj[node])
    if (!done[next_node])
      result = min(result,
                   (dfs_no[next_node]) ? dfs_no[next_node] : tarjan(next_node));

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

### Python
* recursion limit 주의

```python
setrecursionlimit(10 ** 9)
dfs_id, scc_id = 1, 1
adj, done, dfs_no, scc_no, scc_stk = [], [], [], [], []


def tarjan(node):
    global dfs_id, scc_id
    scc_stk.append(node)
    dfs_no[node] = (result := (dfs_id := dfs_id + 1))
    for next_node in adj[node]:
        if not done[next_node]:
            result = min(result, dfs_no[next_node] or tarjan(next_node))

    if result == dfs_no[node]:
        while 1:
            v = scc_stk.pop()
            scc_no[v] = scc_id
            done[v] = True
            if v == node:
                break
        scc_id += 1
    return result
```

# ETC
## Union-find
### C++
```c++
int grpNo[202];
int find(int v) {
	if (grpNo[v] == v) return v;
	grpNo[v] = find(grpNo[v]);
	return grpNo[v];
}
void unite(int a, int b) {
	if (a < b) swap(a, b);
	grpNo[find(a)] = find(b);
}
```