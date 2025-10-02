+++
title = '算法竞赛Cheatsheet'
date = '2025-09-27T12:47:52+08:00'
description = ""
categories = ["经验"]
tags = ["算法竞赛"]
draft = true
isCJKLanguage = true
+++

## 基础

```cpp
#include <algorithm>
#include <cstddef>
#include <cstdlib>
#include <iomanip>
#include <iostream>
#include <queue>
#include <set>
#include <utility>
#include <vector>
#include <string>

#define Heap priority_queue

using namespace std;

using ll = long long;
using ull = unsigned long long;
using pii = pair<int, int>;
using pll = pair<ll, ll>;

const int k_max = 0x3f3f3f3f;
```

## STL

### 算法

![标准库算法](https://hackingcpp.com/cpp/std/algorithms.png)

### 容器

#### 线性容器

![vector](https://hackingcpp.com/cpp/std/vector.png)

![deque](https://hackingcpp.com/cpp/std/deque.png)

![string](https://hackingcpp.com/cpp/std/string.png)

#### 树状容器

![set](https://hackingcpp.com/cpp/std/set.png)

![map](https://hackingcpp.com/cpp/std/map.png)

## IO

```cpp
template <typename Type>
inline Type read() {
  Type x = 0, sign = 1;
  char ch = getchar();
  for (; ch >= '0' && ch <= '9'; ch = getchar())
    if (ch == '-') sign = -1;
  for (; '0' <= ch && ch <= '9'; ch = getchar())
    x = (x << 1) + (x << 3) + (ch ^ 48);
  return sign * x;
}
```

```cpp
std::ios::sync_with_stdio(false);
std::cin.tie(nullptr);
```

|                  算子 | 功能                                                    |
| --------------------: | ------------------------------------------------------- |
|                 `dec` | 以十进制形式输出整数                                    |
|                 `hex` | 以十六进制形式输出整数                                  |
|                 `oct` | 以八进制形式输出整数                                    |
|               `fixed` | 以普通小数形式输出浮点数                                |
|          `scientific` | 以科学计数法形式输出浮点数                              |
|                `left` | 左对齐，即在宽度不足时将填充字符添加到右边              |
|               `right` | 右对齐，即在宽度不足时将填充字符添加到左边              |
|          `setbase(b)` | 设置输出整数时的进制，`b` = 8、10 或 16                 |
|             `setw(w)` | 指定输出宽度为 `w` 个字符                               |
|          `setfill(c)` | 在指定输出宽度的情况下，输出的宽度不足时用字符 `c` 填充 |
|     `setprecision(n)` | 设置输出浮点数的精度为 `n`                              |
|   `setiosflags(flag)` | 将某个输出格式标志置为 `1`                              |
| `resetiosflags(flag)` | 将某个输出格式标志置为 `0`                              |

## 数据结构

### 并查集

```cpp
class UFSet {
 private:
  vector<size_t> parent;

 public:
  UFSet(size_t n) : parent(n) {
    for (size_t i = 0; i < n; i++) parent[i] = i;
  }

  size_t find(size_t x) {
    if (parent[x] != x) parent[x] = find(parent[x]);
    return parent[x];
  }

  void unite(size_t x, size_t y) { parent[find(x)] = find(y); }
};
```

### 树状数组

```cpp
inline size_t lowbit(size_t i) { return i & -i; }
template <typename Type>
class BITree {
 private:
  vector<Type> tree;

 public:
  BITree(size_t n) : tree(n + 1, 0) {}

  void update(size_t i, Type x) {
    for (; i < tree.size(); i += lowbit(i)) tree[i] += x;
  }

  Type query(size_t i) {
    Type sum = 0;
    for (; i > 0; i -= lowbit(i);) sum += tree[i];

    return sum;
  }
};
```

### 珂朵莉树

```cpp
template <typename Type>
struct CTNode {
  size_t left, right;
  mutable Type value;
  CTNode(size_t l, size_t r = 0, Type v = 0) : left(l), right(r), value(v) {}
  bool operator<(const CTNode& other) const { return left < other.left; }
};

template <typename Type>
class CTree : public set<CTNode<Type>> {
 public:
  using set<CTNode<Type>>::set;
  auto split(size_t pos) {
    auto it = this->lower_bound(CTNode<Type>(pos));
    if (it != this->end() && it->left == pos) return it;
    --it;
    if (it->right < pos) return this->end();
    auto node = *it;
    this->erase(it);
    this->insert(CTNode<Type>(node.left, pos - 1, node.value));
    return this->insert(CTNode<Type>(pos, node.right, node.value)).first;
  }
  void assign(size_t l, size_t r, Type v) {
    auto itr = split(r + 1), itl = split(l);
    this->erase(itl, itr);
    this->insert(CTNode<Type>(l, r, v));
  }
  Type query(size_t pos) {
    auto it = this->upper_bound(CTNode<Type>(pos));
    if (it == this->begin()) return Type();
    --it;
    return it->value;
  }
};
```

### 图

```cpp
template <typename Type>
struct Edge {
  size_t from, to;
  Type weight;

  bool operator<(const Edge& other) const { return weight < other.weight; }
};

template <typename Type>
struct Graph {
  size_t num_v, num_e;
  vector<set<Edge<Type>>> edges;

  Graph(size_t v, size_t e) : num_v(v), num_e(e), edges(v) {}
};
```

## 图论

### 最短路

```cpp
template <typename Type>
vector<Type> dijkstra(const Graph<Type>& graph, size_t start) {
  vector<Type> dist(graph.num_v, k_max);
  dist[start] = 0;
  Heap<pii, vector<pii>, greater<pii>> heap;
  heap.push({0, start});

  while (!heap.empty()) {
    auto [d, p] = heap.top();
    heap.pop();
    if (d > dist[p]) continue;
    for (auto e : graph.edges[p]) {
      if (dist[e.to] > dist[p] + e.weight) {
        dist[e.to] = dist[p] + e.weight;
        heap.push({dist[e.to], e.to});
      }
    }
  }
  return dist;
}
```

### 最小生成树

```cpp
template <typename Type>
Type kruskal(const Graph<Type>& graph) {
  set<Edge<Type>> all_edges;
  all_edges.reserve(graph.num_e);
  for (int i = 0; i < graph.num_v; i++)
    for (const auto& e : graph.edges[i]) all_edges.insert(e);
  UFSet uf(graph.num_v);
  Type min_w = 0;
  for (const auto& e : all_edges) {
    if (uf.find(e.from) != uf.find(e.to)) {
      uf.unite(e.from, e.to);
      min_w += e.weight;
    }
  }
  return min_w;
}
```

## 动态规划

### 01 背包

```cpp
for (int i = 0; i < n; i++) {
  for (int j = capacity; j >= weights[i]; j--) {
    dp[j] = max(dp[j], dp[j - weights[i]] + values[i]);
  }
}
```

### 最长上升子序列

```cpp
for (int i = 1; i <= m; i++) 
  for (int j = 1; j <= n; j++) 
    if (text1[i - 1] == text2[j - 1]) 
      dp[i][j] = dp[i - 1][j - 1] + 1;
    else 
      dp[i][j] = max(dp[i - 1][j], dp[i][j - 1]);
```

## 数学

### 快速幂

```cpp
template <typename Type>
Type fast_pow(Type base, Type exp, Type mod) {
  Type result = 1;
  base %= mod;
  while (exp > 0) {
    if (exp & 1) result = (result * base) % mod;
    base = (base * base) % mod;
    exp >>= 1;
  }
  return result;
}
```

### 埃氏筛

```cpp
template <typename Type>
vector<Type> prime_filter(Type n) {
  vector<bool> is_prime(n + 1, true);
  vector<Type> primes;
  is_prime[0] = is_prime[1] = false;

  for (Type i = 2; i <= n; i++) {
    if (is_prime[i]) {
      primes.push_back(i);
      for (ll j = (ll)i * i; j <= n; j += i) is_prime[j] = false;
    }
  }
  return primes;
}
```

## 其它

![编译](https://hackingcpp.com/cpp/lang/separate_compilation.png)
