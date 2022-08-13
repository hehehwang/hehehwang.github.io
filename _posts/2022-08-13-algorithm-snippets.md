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
## Tarjan
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