### 题目分析与解法

这道题是环形区间减一的最小代价问题，核心思路是**差分转化 + 环分解 + 贪心优化**。

---

#### 1. 差分方程推导

设 $x_i$ 表示第 $i$ 种操作的执行次数。每次操作 $i$ 会让区间 $[i, i+k-1] \pmod{n}$ 所有数减 1，因此对于每个位置 $j$，所有覆盖 $j$ 的操作次数之和等于 $a_j$：
$$\sum_{i = j-k+1}^j x_i = a_j$$
对该式做差分（$d_j = a_j - a_{j-1}$，下标模 $n$），可得：
$$x_j - x_{j-k} = a_j - a_{j-1} = d_j$$
即 $x_j = x_{j-k} + d_j$。

---

#### 2. 环分解

令 $g = \gcd(n, k)$，$m = n/g$。下标每次跳 $k$ 步模 $n$，会将数组分成 $g$ 个独立的环，每个环长度为 $m$。

- 第 $r$ 个环包含的下标为：$r,\ r+k,\ r+2k,\ \dots,\ r+(m-1)k \pmod{n}$
- 每个环内的 $x$ 可以表示为 **基准值 + 全局偏移量**：$x_i = \text{base}_i + t_r$，其中 $t_r$ 是第 $r$ 个环的整体偏移量。

---

#### 3. 基准值计算

对每个环，令环首的基准值为 0，通过递推式 $\text{base}[p_t] = \text{base}[p_{t-1}] + d[p_t]$ 计算环上所有位置的基准值。
由于题目保证有解，环的首尾会自动闭合（基准值绕环一周后回到 0）。

---

#### 4. 偏移量约束

给每个环加上偏移量 $t_r$ 后，每个长度为 $k$ 的窗口和会增加 $\frac{k}{g} \cdot \sum_{r=0}^{g-1} t_r$。结合窗口和需等于 $a_j$ 的条件，可推导出所有环的偏移量总和为固定值 $T$：
$$T = \frac{(a_j - \text{sum\_base}_j) \cdot g}{k}$$
其中 $\text{sum\_base}_j$ 是基准值数组在窗口 $j$ 内的和。

---

#### 5. 贪心最小化代价

每个环的偏移量 $t_r$ 必须满足 $x_i = \text{base}*i + t_r \geq 0$，即 $t_r \geq -\min*{i\in \text{环}r} \text{base}_i$（下界 $L_r$）。

总代价为：
$$\text{总代价} = \sum \text{base}*i \cdot c_i + \sum*{r=0}^{g-1} t_r \cdot w_r$$
其中 $w_r = \sum_{i\in \text{环}r} c_i$ 是第 $r$ 个环的单位偏移代价。

在 $\sum t_r = T$ 且 $t_r \geq L_r$ 的约束下，最小化加权和的最优策略是：

- 所有环先取下界 $L_r$
- 剩余的偏移量全部加到**单位代价最小**的环上

---

### C++ 实现代码

```
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;
typedef long long ll;

ll gcd(ll a, ll b) {
    while (b) {
        a %= b;
        swap(a, b);
    }
    return a;
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    
    int n, k;
    cin >> n >> k;
    
    vector<ll> a(n);
    for (int i = 0; i < n; ++i) {
        cin >> a[i];
    }
    
    vector<int> c(n);
    for (int i = 0; i < n; ++i) {
        cin >> c[i];
    }
    
    // 计算差分d
    vector<ll> d(n);
    for (int i = 0; i < n; ++i) {
        int prev = (i - 1 + n) % n;
        d[i] = a[i] - a[prev];
    }
    
    int g = gcd(n, k);
    int m = n / g;
    
    vector<ll> base(n);
    ll base_cost = 0;
    vector<ll> min_base(g);
    vector<ll> w(g);
    
    // 遍历每个环，计算base、min_base、w和base_cost
    for (int r = 0; r < g; ++r) {
        ll cur_base = 0;
        ll min_b = 1e18;
        ll sum_c = 0;
        int p = r;
        for (int t = 0; t < m; ++t) {
            base[p] = cur_base;
            if (cur_base < min_b) {
                min_b = cur_base;
            }
            sum_c += c[p];
            base_cost += cur_base * c[p];
            
            int next_p = (p + k) % n;
            cur_base += d[next_p];
            p = next_p;
        }
        min_base[r] = min_b;
        w[r] = sum_c;
    }
    
    // 前缀和，用于计算窗口base和
    vector<ll> s(n + 1, 0);
    for (int i = 0; i < n; ++i) {
        s[i + 1] = s[i] + base[i];
    }
    
    // 计算j=0处的窗口base和
    int l = (0 - k + 1 + n) % n;
    int r_win = 0;
    ll sum_base;
    if (l <= r_win) {
        sum_base = s[r_win + 1] - s[l];
    } else {
        sum_base = s[n] - s[l] + s[r_win + 1];
    }
    
    // 计算总偏移量T
    ll T = (a[0] - sum_base) * g / k;
    
    // 计算下界和、下界代价和，找最小代价的环
    ll sum_L = 0;
    ll sum_Lw = 0;
    ll min_w = 1e18;
    int min_r = 0;
    for (int i = 0; i < g; ++i) {
        ll L = -min_base[i];
        sum_L += L;
        sum_Lw += L * w[i];
        if (w[i] < min_w) {
            min_w = w[i];
            min_r = i;
        }
    }
    
    // 计算额外代价和总答案
    ll add_cost = sum_Lw + (T - sum_L) * min_w;
    ll ans = base_cost + add_cost;
    
    cout << ans << '\n';
    
    return 0;
}
```

---

### 复杂度分析

- **时间复杂度**：$(O(n))$，环分解、基准值计算、前缀和计算均为线性时间。
- **空间复杂度**：$(O(n))$，存储 base 数组、前缀和数组等。

该算法可以高效处理 $n \leq 2 \times 10^5$ 的数据规模。