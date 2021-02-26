---
title: "每日算法: DLX"
author: Slowy
date: 2021-02-26T19:29:20+08:00
update: 2021-02-26T19:29:20+08:00
#description:
#image: 
categories: algorithm
tags:
  - Dancing Links
  - Luogu
---

每日算法系列第一篇

~~别骂了别骂了，在家实在没动力敲码~~

# Dancing Links & X 算法

X算法是专门解决精确覆盖问题的模板算法，oiwiki上对此有[完整的介绍](https://oi-wiki.org/search/dlx/)

简单来说X算法本身只是简单的回溯：枚举当前列由哪一行覆盖，然后删除冲突的行，递归搜索剩余列。问题就在于怎么优化枚举行和删除行这一过程。一个想法是利用bitset维护，利用lowbit和位运算来枚举和删除，不过这写起来可能有点麻烦。

DLX相对要简单一点，直接维护一个双向十字链表就可以了，也就是所谓的Dancing Links。

oiwiki上关于Dancing Links的图解似乎有点问题，实际上first是不连向右边的。更准确的说，对于一个R行C列,有K个1的矩阵，Dancing Links包含K+C+1个结点，多出的结点其中C个用于指示列，还有一个额外的0号节点，当所有列都被删除(覆盖)后就只剩下0号结点，便于标识X算法是否结束。至于first数组，实际上类似于前向星的head数组那样是用来找出每行第一个1的位置的。

还有额外一点需要补充说明的是insert操作实际上类似于前向星的加边，r行c列的1的位置不必放在r行c列这一位置，刚才所说的first数组是记录第一个1的位置实际上也不准确。实际上只需要保证结点c和first[r]能找到这个1就可以了，因此我们插入结点的写法和前向星差不多，即原文所说的“奇异的方式”。

据oiwiki所说时间复杂度是\\(O(c^n)\\)，其中c接近1的常数，而且1s的话n可以大到几万，这么看来还是十分高效的。

- 例题1: [P4929 【模板】舞蹈链（DLX）](https://www.luogu.com.cn/problem/P4929)

```c++
#include <bits/extc++.h>
#include <bits/stdc++.h>
using namespace std;
using LL = long long;
using Pii = pair<int, int>;
using Pll = pair<LL, LL>;
using VI = vector<int>;
using VP = vector<pair<int, int>>;
#define rep(i, a, b) for (auto i = (a); i < (b); ++i)
#define rev(i, a, b) for (auto i = (b - 1); i >= (a); --i)
#define grep(i, u) for (auto i = gh[u]; i != -1; i = gn[i])
#define mem(x, v) memset(x, v, sizeof(x))
#define cpy(x, y) memcpy(x, y, sizeof(x))
#define SZ(V) static_cast<int>(V.size())
#define pb push_back
#define mp make_pair

constexpr int MAX_NUM = 10100;

int U[MAX_NUM], D[MAX_NUM], L[MAX_NUM], R[MAX_NUM];

int first[MAX_NUM], cnt[MAX_NUM], row[MAX_NUM], col[MAX_NUM], total;

void remove(int c) {
  L[R[c]] = L[c];
  R[L[c]] = R[c];
  for (int i = D[c]; i != c; i = D[i]) {
    for (int j = R[i]; j != i; j = R[j]) {
      U[D[j]] = U[j];
      D[U[j]] = D[j];
      --cnt[col[j]];
    }
  }
}

void recover(int c) {
  for (int i = U[c]; i != c; i = U[i]) {
    for (int j = L[i]; j != i; j = L[j]) {
      U[D[j]] = D[U[j]] = j;
      ++cnt[col[j]];
    }
  }
  R[L[c]] = L[R[c]] = c;
}

void build(int /*r */, int c) {
  rep(i, 0, c + 1) {
    L[i] = i - 1;
    R[i] = i + 1;
    U[i] = D[i] = i;
  }
  L[0] = c;
  R[c] = 0;
  mem(first, 0);
  mem(cnt, 0);
  total = c + 1;
}

void insert(int r, int c) {
  row[total] = r;
  col[total] = c;
  ++cnt[c];
  D[total] = D[c];
  U[D[c]] = total;
  U[total] = c;
  D[c] = total;
  if (first[r]) {
    L[total] = first[r];
    R[total] = R[first[r]];
    R[first[r]] = L[R[total]] = total;
  } else {
    first[r] = L[total] = R[total] = total;
  }
  ++total;
}

constexpr int rk[9][9] = {
    {6, 6, 6, 6, 6, 6, 6, 6, 6}, {6, 7, 7, 7, 7, 7, 7, 7, 6},  {6, 7, 8, 8, 8, 8, 8, 7, 6},
    {6, 7, 8, 9, 9, 9, 8, 7, 6}, {6, 7, 8, 9, 10, 9, 8, 7, 6}, {6, 7, 8, 9, 9, 9, 8, 7, 6},
    {6, 7, 8, 8, 8, 8, 8, 7, 6}, {6, 7, 7, 7, 7, 7, 7, 7, 6},  {6, 6, 6, 6, 6, 6, 6, 6, 6},
};

int ans[MAX_NUM], a[9][9], score;

void choose(int row) {
  int val = row % 9;
  if (!val) val = 9;
  row = (row - val) / 9;
  a[row / 9][row % 9] = val;
}

void dance(int dep) {
  if (!R[0]) {
    rep(i, 0, dep) choose(ans[i]);
    int sum = 0;
    rep(i, 0, 9) rep(j, 0, 9) sum += rk[i][j] * a[i][j];
    if (sum > score) score = sum;
    return;
  }
  int x = R[0];
  for (int i = R[x]; i; i = R[i]) {
    if (cnt[i] < cnt[x]) x = i;
  }
  remove(x);
  for (int i = D[x]; i != x; i = D[i]) {
    ans[dep] = row[i];
    for (int j = R[i]; j != i; j = R[j]) remove(col[j]);
    dance(dep + 1);
    for (int j = L[i]; j != i; j = L[j]) recover(col[j]);
  }
  recover(x);
}

inline int get_block(int x, int y) {
  x /= 3;
  y /= 3;
  return 3 * x + y;
}

int main() {
  rep(i, 0, 9) rep(j, 0, 9) scanf("%d", &a[i][j]);
  build(729, 324);
  rep(i, 0, 9) rep(j, 0, 9) {
    rep(x, 1, 10) {
      if (!a[i][j] || (a[i][j] == x)) {
        insert(81 * i + 9 * j + x, 9 * i + x);
        insert(81 * i + 9 * j + x, 9 * j + 81 + x);
        insert(81 * i + 9 * j + x, 9 * get_block(i, j) + 162 + x);
        insert(81 * i + 9 * j + x, 9 * i + j + 244);
      }
    }
  }
  score = -1;
  dance(0);
  printf("%d\n", score);
}
```

- 例题2: [P1784 数独](https://www.luogu.com.cn/problem/P1784)

```c++
#include <bits/extc++.h>
#include <bits/stdc++.h>
using namespace std;
using LL = long long;
using Pii = pair<int, int>;
using Pll = pair<LL, LL>;
using VI = vector<int>;
using VP = vector<pair<int, int>>;
#define rep(i, a, b) for (auto i = (a); i < (b); ++i)
#define rev(i, a, b) for (auto i = (b - 1); i >= (a); --i)
#define grep(i, u) for (auto i = gh[u]; i != -1; i = gn[i])
#define mem(x, v) memset(x, v, sizeof(x))
#define cpy(x, y) memcpy(x, y, sizeof(x))
#define SZ(V) static_cast<int>(V.size())
#define pb push_back
#define mp make_pair

constexpr int MAX_NUM = 10100;

int U[MAX_NUM], D[MAX_NUM], L[MAX_NUM], R[MAX_NUM];

int first[MAX_NUM], cnt[MAX_NUM], row[MAX_NUM], col[MAX_NUM], total;

void remove(int c) {
  L[R[c]] = L[c];
  R[L[c]] = R[c];
  for (int i = D[c]; i != c; i = D[i]) {
    for (int j = R[i]; j != i; j = R[j]) {
      U[D[j]] = U[j];
      D[U[j]] = D[j];
      --cnt[col[j]];
    }
  }
}

void recover(int c) {
  for (int i = U[c]; i != c; i = U[i]) {
    for (int j = L[i]; j != i; j = L[j]) {
      U[D[j]] = D[U[j]] = j;
      ++cnt[col[j]];
    }
  }
  R[L[c]] = L[R[c]] = c;
}

void build(int /*r */, int c) {
  rep(i, 0, c + 1) {
    L[i] = i - 1;
    R[i] = i + 1;
    U[i] = D[i] = i;
  }
  L[0] = c;
  R[c] = 0;
  mem(first, 0);
  mem(cnt, 0);
  total = c + 1;
}

void insert(int r, int c) {
  row[total] = r;
  col[total] = c;
  ++cnt[c];
  D[total] = D[c];
  U[D[c]] = total;
  U[total] = c;
  D[c] = total;
  if (first[r]) {
    L[total] = first[r];
    R[total] = R[first[r]];
    R[first[r]] = L[R[total]] = total;
  } else {
    first[r] = L[total] = R[total] = total;
  }
  ++total;
}

int ans[MAX_NUM], a[9][9];

void choose(int row) {
  int val = row % 9;
  if (!val) val = 9;
  row = (row - val) / 9;
  a[row / 9][row % 9] = val;
}

bool dance(int dep) {
  if (!R[0]) {
    rep(i, 0, dep) choose(ans[i]);
    return true;
  }
  int x = R[0];
  for (int i = R[x]; i; i = R[i]) {
    if (cnt[i] < cnt[x]) x = i;
  }
  remove(x);
  for (int i = D[x]; i != x; i = D[i]) {
    ans[dep] = row[i];
    for (int j = R[i]; j != i; j = R[j]) remove(col[j]);
    if (dance(dep + 1)) return true;
    for (int j = L[i]; j != i; j = L[j]) recover(col[j]);
  }
  recover(x);
  return false;
}

inline int get_block(int x, int y) {
  x /= 3;
  y /= 3;
  return 3 * x + y;
}

int main() {
  rep(i, 0, 9) rep(j, 0, 9) scanf("%d", &a[i][j]);
  build(729, 324);
  rep(i, 0, 9) rep(j, 0, 9) {
    rep(x, 1, 10) {
      if (!a[i][j] || (a[i][j] == x)) {
        insert(81 * i + 9 * j + x, 9 * i + x);
        insert(81 * i + 9 * j + x, 9 * j + 81 + x);
        insert(81 * i + 9 * j + x, 9 * get_block(i, j) + 162 + x);
        insert(81 * i + 9 * j + x, 9 * i + j + 244);
      }
    }
  }
  dance(0);
  rep(i, 0, 9) rep(j, 0, 9) printf("%d%c", a[i][j], " \n"[j == 8]);
}
```

- 例题3: [P1074 [NOIP2009 提高组] 靶形数独](https://www.luogu.com.cn/problem/P1074)

```c++
#include <bits/extc++.h>
#include <bits/stdc++.h>
using namespace std;
using LL = long long;
using Pii = pair<int, int>;
using Pll = pair<LL, LL>;
using VI = vector<int>;
using VP = vector<pair<int, int>>;
#define rep(i, a, b) for (auto i = (a); i < (b); ++i)
#define rev(i, a, b) for (auto i = (b - 1); i >= (a); --i)
#define grep(i, u) for (auto i = gh[u]; i != -1; i = gn[i])
#define mem(x, v) memset(x, v, sizeof(x))
#define cpy(x, y) memcpy(x, y, sizeof(x))
#define SZ(V) static_cast<int>(V.size())
#define pb push_back
#define mp make_pair

constexpr int MAX_NUM = 10100;

int U[MAX_NUM], D[MAX_NUM], L[MAX_NUM], R[MAX_NUM];

int first[MAX_NUM], cnt[MAX_NUM], row[MAX_NUM], col[MAX_NUM], total;

void remove(int c) {
  L[R[c]] = L[c];
  R[L[c]] = R[c];
  for (int i = D[c]; i != c; i = D[i]) {
    for (int j = R[i]; j != i; j = R[j]) {
      U[D[j]] = U[j];
      D[U[j]] = D[j];
      --cnt[col[j]];
    }
  }
}

void recover(int c) {
  for (int i = U[c]; i != c; i = U[i]) {
    for (int j = L[i]; j != i; j = L[j]) {
      U[D[j]] = D[U[j]] = j;
      ++cnt[col[j]];
    }
  }
  R[L[c]] = L[R[c]] = c;
}

void build(int /*r */, int c) {
  rep(i, 0, c + 1) {
    L[i] = i - 1;
    R[i] = i + 1;
    U[i] = D[i] = i;
  }
  L[0] = c;
  R[c] = 0;
  mem(first, 0);
  mem(cnt, 0);
  total = c + 1;
}

void insert(int r, int c) {
  row[total] = r;
  col[total] = c;
  ++cnt[c];
  D[total] = D[c];
  U[D[c]] = total;
  U[total] = c;
  D[c] = total;
  if (first[r]) {
    L[total] = first[r];
    R[total] = R[first[r]];
    R[first[r]] = L[R[total]] = total;
  } else {
    first[r] = L[total] = R[total] = total;
  }
  ++total;
}

constexpr int rk[9][9] = {
    {6, 6, 6, 6, 6, 6, 6, 6, 6}, {6, 7, 7, 7, 7, 7, 7, 7, 6},  {6, 7, 8, 8, 8, 8, 8, 7, 6},
    {6, 7, 8, 9, 9, 9, 8, 7, 6}, {6, 7, 8, 9, 10, 9, 8, 7, 6}, {6, 7, 8, 9, 9, 9, 8, 7, 6},
    {6, 7, 8, 8, 8, 8, 8, 7, 6}, {6, 7, 7, 7, 7, 7, 7, 7, 6},  {6, 6, 6, 6, 6, 6, 6, 6, 6},
};

int ans[MAX_NUM], a[9][9], score;

void choose(int row) {
  int val = row % 9;
  if (!val) val = 9;
  row = (row - val) / 9;
  a[row / 9][row % 9] = val;
}

void dance(int dep) {
  if (!R[0]) {
    rep(i, 0, dep) choose(ans[i]);
    int sum = 0;
    rep(i, 0, 9) rep(j, 0, 9) sum += rk[i][j] * a[i][j];
    if (sum > score) score = sum;
    return;
  }
  int x = R[0];
  for (int i = R[x]; i; i = R[i]) {
    if (cnt[i] < cnt[x]) x = i;
  }
  remove(x);
  for (int i = D[x]; i != x; i = D[i]) {
    ans[dep] = row[i];
    for (int j = R[i]; j != i; j = R[j]) remove(col[j]);
    dance(dep + 1);
    for (int j = L[i]; j != i; j = L[j]) recover(col[j]);
  }
  recover(x);
}

inline int get_block(int x, int y) {
  x /= 3;
  y /= 3;
  return 3 * x + y;
}

int main() {
  rep(i, 0, 9) rep(j, 0, 9) scanf("%d", &a[i][j]);
  build(729, 324);
  rep(i, 0, 9) rep(j, 0, 9) {
    rep(x, 1, 10) {
      if (!a[i][j] || (a[i][j] == x)) {
        insert(81 * i + 9 * j + x, 9 * i + x);
        insert(81 * i + 9 * j + x, 9 * j + 81 + x);
        insert(81 * i + 9 * j + x, 9 * get_block(i, j) + 162 + x);
        insert(81 * i + 9 * j + x, 9 * i + j + 244);
      }
    }
  }
  score = -1;
  dance(0);
  printf("%d\n", score);
}
```
