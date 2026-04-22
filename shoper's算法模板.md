

[toc]

## 数据结构

### 树状数组

```c++
//树状数组封装
class TR {
private:
    int n;
    vector<int> tr;

public: 
    TR(int n) : n(n) {tr = vector<int>(n + 1, 0);}
    int lowbit(int x) {return (x & (-x));}
    void add(int x, int k) {
        for (int i = x; i <= n; i += lowbit(i)) {
            tr[i] += k;
        }
    }
    ll ask(int x) {
        ll ans = 0;
        for (int i = x; i >= 1; i -= lowbit(i)) {
            ans += tr[i];
        }
        return ans;
    }
};
```

```c++
//查询多个区间内有几个不同的数
const int N = 1e5 + 10;
int c[N];
int n;
void add(int x, int y) {
    for (; x <= n; x += lowbit(x)) {
        c[x] += y;
    }
}
int query_sum(int x) {
    int sum = 0;
    for (; x; x -= lowbit(x)) {
        sum += c[x];
    }
    return sum;
}
struct MM {
    int id, l, r;
    bool friend operator<(const MM& m1, const MM& m2) {
        return m1.r < m2.r;
    }
}qr[N];
int ans[N];
int a[N];
int vis[N];
int main() {
    scanf("%d", &n);
    memset(vis,0,sizeof vis);
    for (int i = 1; i <= n; i++)scanf("%d", a + i);
    int q; scanf("%d", &q);
    for (int i = 1; i<=q; i++) {
        scanf("%d%d", &qr[i].l, &qr[i].r);
        qr[i].id = i;//因为是离线做法，所以需要记录这个区间是第几个查询
    }
    sort(qr + 1, qr + 1 + q);//按照r值升序排序
    int p = 1;//指针初值从1开始
    for (int i = 1; i <= q; i++) {
        for (int j = p; j <= qr[i].r; j++) {
            if (vis[a[j]]) {//判断a[j]在前面是否出现过
                add(vis[a[j]], -1);//将a[j]上一次出现的位置上-1（其实本质就是把上次的+1减回来）
            }
            add(j, 1);//a[j]最新出现的位置上+1
            vis[a[j]] = j;//记录a[j]这次出现的位置为j
        }
        p = qr[i].r + 1;//指针移动
        ans[qr[i].id] = query_sum(qr[i].r) - query_sum(qr[i].l - 1);//保存答案
    }
    for (int i = 1; i <= q; i++)printf("%d\n", ans[i]);
}

//树状数组就是可以求前缀和，同理不就可以有前缀积，前缀异或......

```

### ST表

```c++
#include<bits/stdc++.h>
#define int long long
using namespace std;
using ll = long long;
using pii = pair<int, int>;
using pdd = pair<double, double>;
const int N = (int)2e5 + 9;
const int M = (int)1e5 + 9;
const int mod = (int)1e9 + 7;
const ll INF = LLONG_MAX;

void solve() {
    int n, m;
    cin >> n >> m;
    vector<int> lg(n + 5);
    lg[1] = 0;
    for (int i = 2; i <= n; i++) lg[i] = lg[i >> 1] + 1;
    vector<vector<int>> f(n + 5, vector<int>(lg[n] + 5));
    for (int i = 1; i <= n; i++) {
        cin >> f[i][0];
    }
    for (int j = 1; j <= lg[n]; j++) {
        for (int i = 1; i + (1 << j) - 1 <= n; i++) {
            f[i][j] = max(f[i][j - 1], f[i + (1 << (j - 1))][j - 1]);
        }
    }
    while (m--) {
        int l, r;
        cin >> l >> r;
        int k = lg[r - l + 1];
        cout << max(f[l][k], f[r - (1 << k) + 1][k]) << "\n";
    }
}
signed main()
{
    ios::sync_with_stdio(false);
    cin.tie(0);
    int _ = 1;
    //cin >> _;
    while(_--) {
        solve();
    }
    return 0;
}
```

### 倍增，维护前缀和

```c++
#include <bits/stdc++.h>
#include <bits/extc++.h>
#define int long long
using namespace std;
using namespace __gnu_pbds;
using ll = long long;
using i128 = __int128;
using arr2 = array<int, 2>;
using arr3 = array<int, 3>;
const int N = (int)2e5 + 9;
const int M = (int)1e5 + 9;
const int mod = (int)1e9 + 7;
template <class T>
using ordered_set = tree<
    T,
    null_type,
    less<T>,
    rb_tree_tag,
    tree_order_statistics_node_update>;

void solve() {
    int n, t;
    cin >> n >> t;
    vector<int> a(n + 5);
    for (int i = 1; i <= n; i++) {
        cin >> a[i];
    }
    vector<int> nxt(n + 5);
    stack<int> st;
    for (int i = n; i >= 1; i--) {
        while (st.size() && a[st.top()] <= a[i]) st.pop();
        nxt[i] = st.size() ? st.top() : n + 1;
        st.push(i);
    }
    vector<int> lg(n + 5);
    lg[1] = 0;
    for (int i = 2; i <= n; i++) lg[i] = lg[i >> 1] + 1;
    vector<vector<int>> f(n + 5, vector<int>(lg[n] + 5));
    vector<vector<int>> sum(n + 5, vector<int>(lg[n] + 5));
    for (int i = 1; i <= n; i++) {
        f[i][0] = nxt[i];
        sum[i][0] = a[i] * (nxt[i] - i);
    }
    for (int i = 0; i <= lg[n]; i++) {
        f[n + 1][i] = n + 1;
        sum[n + 1][i] = 0;
    }
    for (int j = 1; j <= lg[n]; j++) {
        for (int i = 1; i <= n; i++) {
            f[i][j] = f[f[i][j - 1]][j - 1];
            sum[i][j] = sum[i][j - 1] + sum[f[i][j - 1]][j - 1];
        }
    }
    int lastans = 0;
    while (t--) {
        int u, v;
        cin >> u >> v;
        int l = 1 + ((u ^ lastans) % n);
        int q = 1 + ((v ^ (lastans + 1)) % (n - l + 1));
        int r = l + q - 1;
        int ans = 0;
        for (int i = lg[n]; i >= 0; i--) {
            if (f[l][i] <= r) {
                ans += sum[l][i];
                l = f[l][i];
            }
        }
        ans += a[l] * (r - l + 1);
        lastans = ans;
        cout << ans << "\n";
    }
}

signed main()
{
    ios::sync_with_stdio(false);
    cin.tie(0);
    int _ = 1;
    //cin >> _;
    while (_--)
    {
        solve();
    }
    return 0;
}
/*
 *   /\_/\
 *  (= ._.)
 *  / >  \>
 */
```

### 线段树

#### 普通线段树

```c++
//普通线段树
struct ty {
    int l, r, mx, f;
}tr[4 * N];
#define ls(x) (tr[x].l)
#define rs(x) (tr[x].r)
#define mx(x) (tr[x].mx)
#define f(x) (tr[x].f)
void pushup(int x) {
    mx(x) = max(mx(ls(x)), mx(rs(x)));
}
void build(int x, int l, int r) {
    if (l == r) {
        mx(x) = 0;
        f(x) = 0;
        return ;
    }
    mx(x) = 0;
    f(x) = 0;
    ls(x) = x << 1;
    rs(x) = x << 1 | 1;
    int mid = l + (r - l) / 2;
    build(ls(x), l, mid);
    build(rs(x), mid + 1, r);
    pushup(x);
}
void pushdown(int x) {
    if (!f(x)) return ;
    int v = f(x);
    f(x) = 0;
    mx(ls(x)) += v;
    f(ls(x)) += v; 
    mx(rs(x)) += v;
    f(rs(x)) += v; 
}
void add(int x, int l, int r, int al, int ar, int val) {
    if (al <= l && ar >= r) {
        mx(x) += val;
        f(x) += val;
        return ;
    }
    pushdown(x);
    int mid = l + (r - l) / 2;
    if (al <= mid) add(ls(x), l, mid, al, ar, val);
    if (ar > mid) add(rs(x), mid + 1, r, al, ar, val);
    pushup(x);
}
int ask(int x, int l, int r, int al, int ar) {
    if (al <= l && ar >= r) {
        return mx(x);
    }
    pushdown(x);
    int res = 0;
    int mid = l + (r - l) / 2;
    if (al <= mid) res = max(res, ask(ls(x), l, mid, al, ar));
    if (ar > mid) res = max(res, ask(rs(x), mid + 1, r, al, ar));
    return res;
} 
```

#### 动态开点线段树

```c++
//这里取模了（注意去掉）
//内存要开 NlogN
struct ty {
    int l = 0, r = 0;
    ll sum = 0, f = 0;
}tr[8000000];

#define ls(x) (tr[x].l)
#define rs(x) (tr[x].r)
#define f(x) (tr[x].f)
#define sum(x) (tr[x].sum) 

int tot = 1, mod;

void pushup(int x) {
    sum(x) = (sum(ls(x)) + sum(rs(x))) % mod;
}
void pushdown(int x, int l, int r) {
    if (f(x)) {
        if (!ls(x)) ls(x) = ++tot;
        if (!rs(x)) rs(x) = ++tot;
        int mid = l + (r - l) / 2;
        ll v = f(x);
        f(x) = 0;
        f(ls(x)) = (f(ls(x)) + v) % mod;
        sum(ls(x)) = (sum(ls(x)) + v * (mid - l + 1) % mod) % mod;
        f(rs(x)) = (f(rs(x)) + v) % mod;
        sum(rs(x)) = (sum(rs(x)) + v * (r - mid) % mod) % mod;
    }
}
void add(int& x, int l, int r, int al, int ar, ll val) {
    if (!x) x = ++tot;
    if (al <= l && ar >= r) {
        sum(x) = (sum(x) + val * (r - l + 1)) % mod;
        f(x) = (f(x) + val) % mod;
        return ;
    }
    pushdown(x, l, r);
    int mid = l + (r - l) / 2;
    if (al <= mid) add(ls(x), l, mid, al, ar, val);
    if (ar > mid) add(rs(x), mid + 1, r, al, ar, val);
    pushup(x);
}
int ask(int x, int l, int r, int al, int ar) {
    if (al <= l && ar >= r) {
        return sum(x);
    }
    ll res = 0;
    pushdown(x, l, r);
    int mid = l + (r - l) / 2;
    if (al <= mid) res = (res + ask(ls(x), l, mid, al, ar)) % mod;
    if (ar > mid) res = (res + ask(rs(x), mid + 1, r, al, ar)) % mod;
    return res;
}
```

#### 线段树合并

```c++
//线段树合并，必须是动态开点
struct ty {
    int l = 0, r = 0, v = 0;
}tr[8000000];
#define v(x) (tr[x].v)
#define ls(x) (tr[x].l)
#define rs(x) (tr[x].r)
int tot = 1;
void pushup(int x) {
    v(x) = v(ls(x)) + v(rs(x));
}
void add(int x, int l, int r, int pos, int val) {
    if (l == r && l == pos) {
        v(x) += val;
        return ;
    }
    int mid = l + (r - l) / 2;
    if (pos <= mid) {
        if (!ls(x)) ls(x) = ++tot;
        add(ls(x), l, mid, pos, val);
    }
    else {
        if (!rs(x)) rs(x) = ++tot;
        add(rs(x), mid + 1, r, pos, val);
    }
    pushup(x);
}
int ask(int x, int l, int r, int al, int ar) {
    if (al <= l && ar >= r) {
        return v(x);
    }
    int res = 0;
    int mid = l + (r - l) / 2;
    if (al <= mid) {
        res += ask(ls(x), l, mid, al, ar);
    }
    if (ar > mid) {
        res += ask(rs(x), mid + 1, r, al, ar);
    }
    return res;
}
int kth(int x, int l, int r, int k) {
    if (l == r) return l;
    int mid = l + (r - l) / 2;
    if (v(ls(x)) >= k) return kth(ls(x), l, mid, k);
    else return kth(rs(x), mid + 1, r, k - v(ls(x)));
}

int merge(int x, int y, int l, int r) {
    if (!x || !y) return x == 0 ? y : x;
    if (l == r) {
        v(x) += v(y);
        return ;
    }
    int mid = l + (r - l) / 2;
    ls(x) = merge(ls(x), ls(y), l, mid);
    rs(x) = merge(rs(x), rs(y), mid + 1, r);
    pushup(x);
    return x;
}
```

#### 可持久化线段树

```c++
//单点修改（区间修改在后面）
int a[N];
struct ty {
    int l, r, v;
}tr[M];
#define ls(x) (tr[x].l)
#define rs(x) (tr[x].r)
#define v(x) (tr[x].v)
int tot = 1;
void pushup(int x) {
    v(x) = v(ls(x)) + v(rs(x));
}
void build(int x, int l, int r) {
    if (l == r) {
        v(x) = a[l];
        return ;
    }
    ls(x) = ++tot, rs(x) = ++tot;
    int mid = l + (r - l) / 2;
    build(ls(x), l, mid);
    build(rs(x), mid + 1, r);
    pushup(x);
}
void add(int x, int y, int l, int r, int p, int val) {
    tr[y] = tr[x];
    if (l == r) {
        v(y) = val;
        return ;
    }
    //ls(y) = ls(x), rs(y) = rs(x);
    int mid = l + (r - l) / 2;
    if (p <= mid) {
        ls(y) = ++tot;
        add(ls(x), ls(y), l, mid, p, val);
    }
    else {
        rs(y) = ++tot;
        add(rs(x), rs(y), mid + 1, r, p, val);
    }
    pushup(y);
}
int ask(int x, int l, int r, int p) {
    if (l == r) {
        return v(x);
    }
    int mid = l + (r - l) / 2;
    if (p <= mid) {
        return ask(ls(x), l, mid, p);
    }
    else {
        return ask(rs(x), mid + 1, r, p);
    }
}
```

```c++
int a[N];
struct ty {
    int l, r;
    ll sum, f;
}tr[M];
#define ls(x) (tr[x].l)
#define rs(x) (tr[x].r)
#define sum(x) (tr[x].sum)
#define f(x) (tr[x].f)
int tot = 1;
void pushup(int x, int l, int r) {
    sum(x) = sum(ls(x)) + sum(rs(x)) + f(x) * (r - l + 1);
}
void build(int x, int l, int r) {
    if (l == r) {
        sum(x) = a[l];
        return ;
    }
    ls(x) = ++tot, rs(x) = ++tot;
    int mid = l + (r - l) / 2;
    build(ls(x), l, mid);
    build(rs(x), mid + 1, r);
    pushup(x, l, r);
}
void add(int x, int y, int l, int r, int al, int ar, ll val) {
    tr[y] = tr[x];
    if (al <= l && ar >= r) {
        f(y) += val;
        sum(y) += val * (r - l + 1);
        return ;
    }
    //ls(y) = ls(x), rs(y) = rs(x);
    int mid = l + (r - l) / 2;
    if (al <= mid) {
        ls(y) = ++tot;
        add(ls(x), ls(y), l, mid, al, ar, val);
    }
    if (ar > mid) {
        rs(y) = ++tot;
        add(rs(x), rs(y), mid + 1, r, al, ar, val);
    }
    pushup(y, l, r);
}
ll ask(int x, int l, int r, int al, int ar, ll lazy) {
    if (al <= l && ar >= r) {
        return sum(x) + lazy * (r - l + 1);
    }
    ll res = 0;
    int mid = l + (r - l) / 2;
    lazy += f(x);
    if (al <= mid) res += ask(ls(x), l, mid, al, ar, lazy);
    if (ar > mid) res += ask(rs(x), mid + 1, r, al, ar, lazy);
    return res;
}
```

### 扫描线
- 注意坚持同一个区间原则，左闭右开！！！
- 这里离散用二分不要用哈希
- 线段树维护区间长

```c++
#include<bits/stdc++.h>
#include<bits/extc++.h>
#define int long long
using namespace std;
using namespace __gnu_pbds;
using ll = long long;
using ull = unsigned long long;
using i128 = __int128;
using arr2 = array<int, 2>;
using arr3 = array<int, 3>;
using arr4 = array<int, 4>;
const int N = (int)4e5 + 9;
const int M = (int)1e5 + 9;
const int mod = (int)998244353;
template<class T>
using ordered_set = tree<
    T,
    null_type,
    less<T>,
    rb_tree_tag,
    tree_order_statistics_node_update>;
int yy[N + 5];
struct ty {
    int l, r, len, f;
}tr[4 * N];
#define ls(x) (tr[x].l)
#define rs(x) (tr[x].r)
#define len(x) (tr[x].len)
#define f(x) (tr[x].f)

void push(int x, int l, int r) {
    if (f(x) > 0) {
        len(x) = yy[r + 1] - yy[l];
    }
    else if (l == r) {
        len(x) = 0;
    }
    else {
        len(x) = len(ls(x)) + len(rs(x));
    }
}

void build(int x, int l, int r) {
    if (l == r) {
        len(x) = 0;
        f(x) = 0;
        return ;
    }
    int mid = l + (r - l) / 2;
    ls(x) = x << 1;
    rs(x) = x << 1 | 1;
    build(ls(x), l, mid);
    build(rs(x), mid + 1, r);
}

void add(int x, int l, int r, int al, int ar, int v) {
    if (al <= l && ar >= r) {
        f(x) += v;
        push(x, l, r);
        return ;
    }
    int mid = l + (r - l) / 2;
    if (al <= mid) add(ls(x), l, mid, al, ar, v);
    if (ar > mid) add(rs(x), mid + 1, r, al, ar, v);
    push(x, l, r);
}
int ask() {
    return len(1);
}

bool cmp(arr2 a , arr2 b) {
    return a[1] < b[1];
}
void solve() {
    int n, m, h, w, nn;
    cin >> n >> m >> h >> w >> nn;

    int nh = n - h + 1;
    int mw = m - w + 1;

    int ans = nh * mw;

    vector<arr4> ev;
    vector<int> ys;
    ev.push_back({});
    ys.push_back(0);
    for (int i = 1; i <= nn; i++) {
        int R, C;
        cin >> R >> C;

        int x1 = max(1LL, C - w + 1);
        int x2 = min(mw, C);
        int y1 = max(1LL, R - h + 1);
        int y2 = min(nh, R);
        ev.push_back({x1, y1, y2 + 1, 1});
        ev.push_back({x2 + 1, y1, y2 + 1, -1});

        ys.push_back(y1);
        ys.push_back(y2 + 1);
    }
    sort(ys.begin(), ys.end());
    ys.erase(unique(ys.begin(), ys.end()), ys.end());
    for (int i = 1; i < ys.size(); i++) yy[i] = ys[i];
    int mm = ys.size() - 1;
    if (mm == 0) {
        cout << ans << "\n";
        return ;
    }
    build(1, 1, mm);
    sort(ev.begin(), ev.end());
    int sum = 0;
    int prex = ev[1][0];
    int i = 1;
    while (i < ev.size()) {
        int x = ev[i][0];
        sum += ask() * (x - prex);
        while (i < ev.size() && ev[i][0] == x) {
            int l = lower_bound(ys.begin(), ys.end(), ev[i][1]) - ys.begin();
            int r = lower_bound(ys.begin(), ys.end(), ev[i][2]) - ys.begin();
            add(1, 1, mm, l, r - 1, ev[i][3]);
            i++;
        }
        prex = x;
    }
    cout << ans - sum << "\n";
}

signed main()
{
    ios::sync_with_stdio(false);
    cin.tie(0);
    int _ = 1;
    //cin >> _;
    while(_--) {
        solve();
    }
    return 0;
}
/*
*   /\_/\
*  (= ._.)
*  / >  \>
*/
```

### pbds库

```c++
//ordered_set(去重)
#include <bits/extc++.h>
using namespace std;
using namespace __gnu_pbds;

template<class T>
using ordered_set = tree<
    T,
    null_type,
    less<T>,
    rb_tree_tag,
    tree_order_statistics_node_update>;

int main() {
    
    ordered_set<int> s;
    s.insert(3);
    s.insert(1);
    s.insert(5);

    cout << s.order_of_key(4) << '\n';   // 2, 因为有 1,3 小于 4
    //find_by_order(k) 是 0-based
    cout << *s.find_by_order(1) << '\n'; // 3, 第 1 个(0-based)
    return 0;
}


//ordered_multiset
#include <bits/extc++.h>
using namespace std;
using namespace __gnu_pbds;

using PII = pair<int,int>;

template<class T>
using ordered_set = tree<
    T,
    null_type,
    less<T>,
    rb_tree_tag,
    tree_order_statistics_node_update>;

int main() {
    ordered_set<PII> s;
    int tim = 0;

    auto insert_val = [&](int x) {
        s.insert({x, tim++});
    };

    auto erase_one = [&](int x) {
        auto it = s.lower_bound({x, -1});
        if (it != s.end() && it->first == x) s.erase(it);
    };

    insert_val(5);
    insert_val(5);
    insert_val(3);

    // 严格小于 5 的个数
    cout << s.order_of_key({5, -1}) << '\n'; // 1

    // 小于等于 5 的个数
    cout << s.order_of_key({5, INT_MAX}) << '\n'; // 3

    // 第 2 小（0-based）
    cout << s.find_by_order(2)->first << '\n'; // 5
    return 0;
}


//pbds priority_queue（会 modify）
#include <bits/extc++.h>
using namespace std;
using namespace __gnu_pbds;

using PII = pair<int,int>;

int main() {
    __gnu_pbds::priority_queue<PII, greater<PII>, pairing_heap_tag> pq;
    using Heap = __gnu_pbds::priority_queue<PII, greater<PII>, pairing_heap_tag>;

    Heap::point_iterator it1 = pq.push({5, 1});
    Heap::point_iterator it2 = pq.push({10, 2});

    pq.modify(it2, {3, 2}); // 把原来的 {10,2} 改成 {3,2}

    while (!pq.empty()) {
        auto [d, x] = pq.top();
        cout << d << ' ' << x << '\n';
        pq.pop();
    }
    return 0;
}
```

### FHQ-treap

#### 区间翻转

```c++
//区间翻转问题，维护位置关键字
#include <bits/stdc++.h>
using namespace std;
const int N = 100000 + 5;
mt19937 rng(chrono::steady_clock::now().time_since_epoch().count());
struct Node {
    int l, r;      // 左右儿子
    int val;       // 节点值
    int pri;       // 随机优先级
    int sz;        // 子树大小
    bool rev;      // 翻转懒标记
} tr[N];
int root, tot;
int newnode(int val) {
    ++tot;
    tr[tot].l = tr[tot].r = 0;
    tr[tot].val = val;
    tr[tot].pri = (int)rng();
    tr[tot].sz = 1;
    tr[tot].rev = false;
    return tot;
}
void pushup(int u) {
    tr[u].sz = tr[tr[u].l].sz + tr[tr[u].r].sz + 1;
}
void pushdown(int u) {
    if (!u || !tr[u].rev) return;
    swap(tr[u].l, tr[u].r);
    if (tr[u].l) tr[tr[u].l].rev ^= 1;
    if (tr[u].r) tr[tr[u].r].rev ^= 1;
    tr[u].rev = false;
}
// 按前 k 个元素分裂：x = 前 k 个，y = 其余
void split(int u, int k, int &x, int &y) {
    if (!u) {
        x = y = 0;
        return;
    }
    pushdown(u);
    if (tr[tr[u].l].sz >= k) {
        y = u;
        split(tr[u].l, k, x, tr[u].l);
        pushup(y);
    } else {
        x = u;
        split(tr[u].r, k - tr[tr[u].l].sz - 1, tr[u].r, y);
        pushup(x);
    }
}
// 合并两棵树，要求 x 整体在 y 前面
int merge(int x, int y) {
    if (!x || !y) return x ? x : y;
    if (tr[x].pri < tr[y].pri) {
        pushdown(x);
        tr[x].r = merge(tr[x].r, y);
        pushup(x);
        return x;
    } else {
        pushdown(y);
        tr[y].l = merge(x, tr[y].l);
        pushup(y);
        return y;
    }
}
void inorder(int u) {
    if (!u) return;
    pushdown(u);
    inorder(tr[u].l);
    cout << tr[u].val << ' ';
    inorder(tr[u].r);
}
int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    int n, m;
    cin >> n >> m;
    // 初始序列 1...n
    for (int i = 1; i <= n; i++) {
        root = merge(root, newnode(i));
    }
    while (m--) {
        int l, r;
        cin >> l >> r;
        int a, b, c;
        split(root, r, a, c);      // a: 前 r 个
        split(a, l - 1, a, b);     // b: [l, r]
        tr[b].rev ^= 1;            // 翻转中间区间
        root = merge(a, merge(b, c));
    }
    inorder(root);
    cout << '\n';
    return 0;
}
```

#### 区间加

**维护的不再是区间位置，而是值的大小作为关键字**

```c++
//维护值大小关键字，区间加
#include<bits/stdc++.h>
#include<bits/extc++.h>
//#define int long long
using namespace std;
using namespace __gnu_pbds;
using ll = long long;
using i128 = __int128;
using arr2 = array<int, 2>;
using arr3 = array<int, 3>;
const int N = (int)2e6 + 9;
const int M = (int)8e6;
const int mod = (int)998244353;
template<class T>
using ordered_set = tree<
    T,
    null_type,
    less<T>,
    rb_tree_tag,
    tree_order_statistics_node_update>;

using pr = pair<string, int>;

mt19937 rng(chrono::steady_clock::now().time_since_epoch().count());
struct ty {
    int l, r, sz, pri;
    ll v, f;
    //ll sum;
}tr[N];
int tot;
#define v(x) (tr[x].v)
#define f(x) (tr[x].f)
//#define sum(x) (tr[x].sum)
#define ls(x) (tr[x].l)
#define rs(x) (tr[x].r)
#define sz(x) (tr[x].sz)
#define pri(x) (tr[x].pri)
int newnode(int val) {
    ++tot;
    v(tot) = val;
    ls(tot) = rs(tot) = f(tot) = 0;
    sz(tot) = 1;
    pri(tot) = (int)rng();
    //sum(tot) = val;
    return tot;
}
void pushup(int x) {
    //sum(x) = sum(ls(x)) + sum(rs(x)) + v(x);
    sz(x) = sz(ls(x)) + sz(rs(x)) + 1;
}
void pushdown(int x) {
    if (!f(x) || !x) return ;
    ll ff = f(x);
    f(x) = 0;
    //sum(ls(x)) += ff * sz(ls(x));
    f(ls(x)) += ff;
    v(ls(x)) += ff;
    //sum(rs(x)) += ff * sz(rs(x));
    f(rs(x)) += ff;
    v(rs(x)) += ff;
}
void split(int u, int k, int& x, int& y) {
    if (!u) {
        x = 0, y = 0;
        return ;
    }
    pushdown(u);
    if (v(u) > k) {
        y = u;
        split(ls(u), k, x, ls(u));
        pushup(y);
    }
    else {
        x = u;
        split(rs(u), k, rs(u), y);
        pushup(x);
    }
}
int merge(int x, int y) {
    if (!x || !y) return x ? x : y;
    if (pri(x) > pri(y)) {
        pushdown(x);
        rs(x) = merge(rs(x), y);
        pushup(x);
        return x;
    }
    else {
        pushdown(y);
        ls(y) = merge(x, ls(y));
        pushup(y);
        return y;
    }
}
//一定要记得这个！！
void dfs(int u) {
    if (!u) return;
    pushdown(u);
    dfs(ls(u));
    dfs(rs(u));
}
void solve() {
    tot = 0;
    int n;
    cin >> n;
    vector<int> a(n + 5);
    int root = 0;
    for (int i = 1; i <= n; i++) {
        cin >> a[i];
        int x, y;
        split(root, a[i] - 1, x, y);
        v(y) += a[i];
        f(y) += a[i];
        x = merge(x, newnode(a[i]));
        root = merge(x, y);
    }   
    dfs(root);
    for (int i = 1; i <= n; i++) {
        cout << v(i) << " ";
    }
    cout << "\n";
}

signed main()
{
    ios::sync_with_stdio(false);
    cin.tie(0);
    int _ = 1;
    cin >> _;
    while(_--) {
        solve();
    }
    return 0;
}
/*
*   /\_/\
*  (= ._.)
*  / >  \>
*/
```

### 主席树

```c++
#include<bits/stdc++.h>
#include<bits/extc++.h>
//#define int long long
using namespace std;
using namespace __gnu_pbds;
using ll = long long;
using i128 = __int128;
using arr2 = array<int, 2>;
using arr3 = array<int, 3>;
const int N = (int)2e5 + 9;
const int M = (int)8e6;
const int mod = (int)998244353;
template<class T>
using ordered_set = tree<
    T,
    null_type,
    less<T>,
    rb_tree_tag,
    tree_order_statistics_node_update>;

using pr = pair<string, int>;

int a[N];
struct ty {
    int l, r, v;
}tr[M];
#define ls(x) (tr[x].l)
#define rs(x) (tr[x].r)
#define v(x) (tr[x].v)
int tot = 0;
void pushup(int x) {
    v(x) = v(ls(x)) + v(rs(x));
}
void add(int x, int y, int l, int r, int p, int val) {
    tr[y] = tr[x];
    if (l == r) {
        v(y) += val;
        return ;
    }
    int mid = l + (r - l) / 2;
    if (p <= mid) {
        ls(y) = ++tot;
        add(ls(x), ls(y), l, mid, p, val);
    }
    else {
        rs(y) = ++tot;
        add(rs(x), rs(y), mid + 1, r, p, val);
    }
    pushup(y);
}
int kth(int x, int y, int l, int r, int k) {
    if (l == r) {
        return l;
    }
    int mid = l + (r - l) / 2;
    int num = v(ls(y)) - v(ls(x));
    if (num >= k) {
        return kth(ls(x), ls(y), l, mid, k);
    }
    else {
        return kth(rs(x), rs(y), mid + 1, r, k - num);
    }
}

void solve() {
    int n, m;
    cin >> n >> m;
    vector<int> b;
    for (int i = 1; i <= n; i++) {
        int x;
        cin >> x;
        b.push_back(x);
    }
    auto c = b;
    unordered_map<int, int> mp;
    sort(b.begin(), b.end());
    b.erase(unique(b.begin(), b.end()), b.end());
    for (int i = 0; i < b.size(); i++) {
        mp[b[i]] = i + 1;
    }
    for (int i = 1; i <= n; i++) {
        a[i] = mp[c[i - 1]];
    }
    vector<int> rt(n + 5);
    for (int i = 1; i <= n; i++) {
        rt[i] = ++tot;
        add(rt[i - 1], rt[i], 1, n, a[i], 1);
    }
    for (int i = 1; i <= m; i++) {
        int l, r, k;
        cin >> l >> r >> k;
        cout << b[kth(rt[l - 1], rt[r], 1, n, k) - 1] << "\n";
    }

}

signed main()
{
    ios::sync_with_stdio(false);
    cin.tie(0);
    int _ = 1;
    //cin >> _;
    while(_--) {
        solve();
    }
    return 0;
}
/*
*   /\_/\
*  (= ._.)
*  / >  \>
*/
```

### 树链剖分

#### 单点修改

```c++
//单点修改
#include <bits/stdc++.h>
using namespace std;

using ll = long long;

const int N = 200005;

int n, q;
vector<int> g[N];
ll val[N];           // 原始点权

int fa[N], dep[N], siz[N], son[N];
int top[N], dfn[N], rnk_[N], timer_;

// ---------- 第一遍 DFS：求 fa / dep / siz / son ----------
void dfs1(int u, int father) {
    fa[u] = father;
    dep[u] = dep[father] + 1;
    siz[u] = 1;
    son[u] = 0;

    for (int v : g[u]) {
        if (v == father) continue;
        dfs1(v, u);
        siz[u] += siz[v];
        if (!son[u] || siz[v] > siz[son[u]]) {
            son[u] = v;
        }
    }
}

// ---------- 第二遍 DFS：求 top / dfn / rnk ----------
void dfs2(int u, int topf) {
    top[u] = topf;
    dfn[u] = ++timer_;
    rnk_[timer_] = u;

    if (son[u]) {
        dfs2(son[u], topf); // 重儿子优先，保证重链连续
    }

    for (int v : g[u]) {
        if (v == fa[u] || v == son[u]) continue;
        dfs2(v, v); // 轻儿子开新链
    }
}

// ---------- 线段树 ----------
struct SegTree {
    ll sum[N << 2];

    void push_up(int p) {
        sum[p] = sum[p << 1] + sum[p << 1 | 1];
    }

    void build(int p, int l, int r) {
        if (l == r) {
            sum[p] = val[rnk_[l]]; // dfn 对应的点权
            return;
        }
        int mid = (l + r) >> 1;
        build(p << 1, l, mid);
        build(p << 1 | 1, mid + 1, r);
        push_up(p);
    }

    void point_modify(int p, int l, int r, int pos, ll x) {
        if (l == r) {
            sum[p] = x;
            return;
        }
        int mid = (l + r) >> 1;
        if (pos <= mid) point_modify(p << 1, l, mid, pos, x);
        else point_modify(p << 1 | 1, mid + 1, r, pos, x);
        push_up(p);
    }

    ll query(int p, int l, int r, int ql, int qr) {
        if (ql <= l && r <= qr) return sum[p];
        int mid = (l + r) >> 1;
        ll res = 0;
        if (ql <= mid) res += query(p << 1, l, mid, ql, qr);
        if (qr > mid) res += query(p << 1 | 1, mid + 1, r, ql, qr);
        return res;
    }
} seg;

// ---------- 查询 u 到 v 路径上的点权和 ----------
ll query_path(int u, int v) {
    ll res = 0;
    while (top[u] != top[v]) {
        if (dep[top[u]] < dep[top[v]]) swap(u, v);
        res += seg.query(1, 1, n, dfn[top[u]], dfn[u]);
        u = fa[top[u]];
    }
    if (dep[u] > dep[v]) swap(u, v);
    res += seg.query(1, 1, n, dfn[u], dfn[v]);
    return res;
}

// ---------- 查询 u 子树点权和 ----------
ll query_subtree(int u) {
    return seg.query(1, 1, n, dfn[u], dfn[u] + siz[u] - 1);
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    cin >> n >> q;
    for (int i = 1; i <= n; i++) cin >> val[i];

    for (int i = 1; i < n; i++) {
        int u, v;
        cin >> u >> v;
        g[u].push_back(v);
        g[v].push_back(u);
    }

    // 默认以 1 为根
    dfs1(1, 0);
    dfs2(1, 1);
    seg.build(1, 1, n);

    // 示例操作：
    // 1 u x : 修改点 u 权值为 x
    // 2 u v : 查询 u 到 v 路径和
    // 3 u   : 查询 u 子树和
    while (q--) {
        int op;
        cin >> op;
        if (op == 1) {
            int u;
            ll x;
            cin >> u >> x;
            seg.point_modify(1, 1, n, dfn[u], x);
        } else if (op == 2) {
            int u, v;
            cin >> u >> v;
            cout << query_path(u, v) << '\n';
        } else if (op == 3) {
            int u;
            cin >> u;
            cout << query_subtree(u) << '\n';
        }
    }

    return 0;
}
```

#### 区间修改

```c++
//区间修改
#include<bits/stdc++.h>
#include<bits/extc++.h>
//#define int long long
using namespace std;
using namespace __gnu_pbds;
using ll = long long;
using i128 = __int128;
using arr2 = array<int, 2>;
using arr3 = array<int, 3>;
const int N = (int)2e5 + 9;
const int M = (int)1e5 + 9;
//const int mod = (int)1e9 + 7;
template<class T>
using ordered_set = tree<
    T,
    null_type,
    less<T>,
    rb_tree_tag,
    tree_order_statistics_node_update>;
int n, m, rt, mod;
vector<int> g[N];
ll val[N];
int fa[N], dep[N], sz[N], son[N];
int top[N], dfn[N], rk[N], ti;
void dfs1(int u, int f) {
    fa[u] = f;
    dep[u] = dep[f] + 1;
    sz[u] = 1;
    son[u] = 0;
    for (auto v : g[u]) {
        if (v == f) continue;
        dfs1(v, u);
        sz[u] += sz[v];
        if (!son[u] || sz[v] > sz[son[u]]) {
            son[u] = v;
        }
    }
}
void dfs2(int u, int t) {
    top[u] = t;
    dfn[u] = ++ti;
    rk[ti] = u;
    if (son[u]) dfs2(son[u], t);
    for (auto v : g[u]) {
        if (v == fa[u] || v == son[u]) continue;
        dfs2(v, v);
    }
}
struct ty {
    int l, r;
    ll sum, f;
}tr[N << 2];
#define ls(x) (tr[x].l)
#define rs(x) (tr[x].r)
#define sum(x) (tr[x].sum)
#define f(x) (tr[x].f)
void pushup(int x) {
    sum(x) = (sum(ls(x)) + sum(rs(x))) % mod;
}
void build(int x, int l, int r) {
    if (l == r) {
        sum(x) = val[rk[l]] % mod;
        return ;
    }
    int mid = l + (r - l) / 2;
    ls(x) = x << 1;
    rs(x) = x << 1 | 1;
    build(ls(x), l, mid);
    build(rs(x), mid + 1, r);
    pushup(x);
}
void pushdown(int x, int l, int r) {
    if (!f(x)) return ;
    ll v = f(x);
    f(x) = 0;
    int mid = l + (r - l) / 2;
    f(ls(x)) = (f(ls(x)) + v) % mod;
    sum(ls(x)) = (sum(ls(x)) + v * (mid - l + 1) % mod) % mod;
    f(rs(x)) = (f(rs(x)) + v) % mod;
    sum(rs(x)) = (sum(rs(x)) + v * (r - mid) % mod) % mod;
}
void add(int x, int l, int r, int al, int ar, ll d) {
    if (al <= l && ar >= r) {
        f(x) = (f(x) + d) % mod;
        sum(x) = (sum(x) + d * (r - l + 1) % mod) % mod;
        return ;
    }
    pushdown(x, l, r);
    int mid = l + (r - l) / 2;
    if (al <= mid) add(ls(x), l, mid, al, ar, d);
    if (ar > mid) add(rs(x), mid + 1, r, al, ar, d);
    pushup(x);
}
ll ask(int x, int l, int r, int al, int ar) {
    if (al <= l && ar >= r) {
        return sum(x);
    }
    ll res = 0;
    pushdown(x, l, r);
    int mid = l + (r - l) / 2;  
    if (al <= mid) res = (res + ask(ls(x), l, mid, al, ar)) % mod;
    if (ar > mid) res = (res + ask(rs(x), mid + 1, r, al, ar)) % mod;
    return res;
}
ll askpa(int u, int v) {
    ll res = 0;
    while (top[u] != top[v]) {
        if (dep[top[u]] < dep[top[v]]) swap(u, v);
        res = (res + ask(1, 1, n, dfn[top[u]], dfn[u])) % mod;
        u = fa[top[u]];
    }
    if (dep[u] > dep[v]) swap(u, v);
    res = (res + ask(1, 1, n, dfn[u], dfn[v])) % mod;
    return res;
}
void addpa(int u, int v, int d) {
    while (top[u] != top[v]) {
        if (dep[top[u]] < dep[top[v]]) swap(u, v);
        add(1, 1, n, dfn[top[u]], dfn[u], d);
        u = fa[top[u]];
    }
    if (dep[u] > dep[v]) swap(u, v);
    add(1, 1, n, dfn[u], dfn[v], d);
}
ll asksub(int u) {
    return ask(1, 1, n, dfn[u], dfn[u] + sz[u] - 1);
}

void solve() {
    cin >> n >> m >> rt >> mod;
    for (int i = 1; i <= n; i++) cin >> val[i];
    for (int i = 1; i < n; i++) {
        int u, v;
        cin >> u >> v;
        g[u].push_back(v);
        g[v].push_back(u);
    }
    dfs1(rt, 0);
    dfs2(rt, rt);
    build(1, 1, n);
    for (int i = 1; i <= m; i++) {
        int op;
        cin >> op;
        if (op == 1) {
            int x, y, z;
            cin >> x >> y >> z;
            addpa(x, y, z);
        }
        else if (op == 2) {
            int x, y;
            cin >> x >> y;
            cout << askpa(x, y) << "\n";
        }
        else if (op == 3) {
            int x, z;
            cin >> x >> z;
            add(1, 1, n, dfn[x], dfn[x] + sz[x] - 1, z);
        }
        else {
            int x;
            cin >> x;
            cout << asksub(x) << "\n";
        }
    }
}

signed main()
{
    ios::sync_with_stdio(false);
    cin.tie(0);
    int _ = 1;
    //cin >> _;
    while(_--) {
        solve();
    }
    return 0;
}
```

### LCA

```c++
#include <bits/stdc++.h>
using namespace std;

struct LCA {
    int n, LOG, root;
    vector<vector<int>> g;
    vector<vector<int>> up;  // up[v][k] = 2^k-th ancestor of v
    vector<int> depth;

    LCA(int n_, int root_ = 1) : n(n_), root(root_) {
        LOG = 1;
        while ((1 << LOG) <= n) LOG++;
        g.assign(n + 1, {});
        up.assign(n + 1, vector<int>(LOG, 0));
        depth.assign(n + 1, 0);
    }

    void add_edge(int u, int v) {
        g[u].push_back(v);
        g[v].push_back(u);
    }

    // Build with BFS (safe for large n)
    void build() {
        vector<int> parent(n + 1, 0);
        queue<int> q;
        q.push(root);
        parent[root] = 0;
        depth[root] = 0;
        up[root][0] = 0;

        while (!q.empty()) {
            int v = q.front(); q.pop();
            for (int to : g[v]) {
                if (to == parent[v]) continue;
                parent[to] = v;
                depth[to] = depth[v] + 1;
                up[to][0] = v;
                q.push(to);
            }
        }

        for (int k = 1; k < LOG; k++) {
            for (int v = 1; v <= n; v++) {
                up[v][k] = up[ up[v][k - 1] ][k - 1];
            }
        }
    }

    int jump(int v, int steps) const {
        for (int k = 0; k < LOG; k++) {
            if (steps & (1 << k)) v = up[v][k];
            if (v == 0) break;
        }
        return v;
    }

    int lca(int a, int b) const {
        if (a == 0 || b == 0) return a ^ b; // if one is 0
        if (depth[a] < depth[b]) swap(a, b);

        // lift a to same depth
        a = jump(a, depth[a] - depth[b]);
        if (a == b) return a;

        // lift both
        for (int k = LOG - 1; k >= 0; k--) {
            if (up[a][k] != up[b][k]) {
                a = up[a][k];
                b = up[b][k];
            }
        }
        return up[a][0];
    }

    int dist(int a, int b) const {
        int c = lca(a, b);
        return depth[a] + depth[b] - 2 * depth[c];
    }
};

// Example usage
int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int n, q;
    cin >> n >> q;

    LCA solver(n, 1);
    for (int i = 0; i < n - 1; i++) {
        int u, v;
        cin >> u >> v;
        solver.add_edge(u, v);
    }
    solver.build();

    while (q--) {
        int u, v;
        cin >> u >> v;
        cout << solver.lca(u, v) << "\n";
    }
    return 0;
}




#include<bits/stdc++.h>
#define int long long
using namespace std;
using ll = long long;
using arr2 = array<int, 2>;
using arr3 = array<int, 3>;
const int N = (int)2e5 + 9;
const int M = (int)1e5 + 9;
const int mod = (int)1e9 + 7;

void solve() {
    int n, m, s;
    cin >> n >> m >> s;
    vector<vector<int>> e(n + 5);
    for (int i = 1; i < n; i++) {
        int u, v;
        cin >> u >> v;
        e[u].push_back(v);
        e[v].push_back(u);
    }
    vector<int> d(n + 5);
    int log = 1;
    while ((1 << log) <= n) log++;
    vector<vector<int>> fa(n + 5, vector<int>(log + 5, 0));
    vector<int> pa(n + 5, 0);
    queue<int> q;
    q.push(s);
    while (!q.empty()) {
        int u = q.front(); q.pop();
        for (int v : e[u]) {
            if (pa[u] == v) continue;
            pa[v] = u;
            d[v] = d[u] + 1;
            fa[v][0] = u;
            q.push(v);
        }
    }
    for (int k = 1; k < log; k++) {
        for (int i = 1; i <= n; i++) {
            fa[i][k] = fa[fa[i][k - 1]][k - 1];
        }
    }
    auto jump = [&](int a, int b) -> int {
        for (int k = 0; k < log; k++) {
            if ((1 << k) & b) a = fa[a][k];
            if (a == 0) break;
        }
        return a;
    };
    auto lca = [&](int a, int b) -> int {
        if (d[a] < d[b]) swap(a, b);
        a = jump(a, d[a] - d[b]);
        if (a == b) return a;
        for (int k = log - 1; k >= 0; k--) {
            if (fa[a][k] != fa[b][k]) {
                a = fa[a][k];
                b = fa[b][k];
            }
        }
        return fa[a][0];
    };
    auto dist = [&](int a, int b) -> int {
        int c = lca(a, b);
        return d[a] + d[b] - 2 * d[c];
    };
    while (m--) {
        int a, b;
        cin >> a >> b;
        cout << lca(a, b) << "\n";
    }
}

signed main()
{
    ios::sync_with_stdio(false);
    cin.tie(0);
    int _ = 1;
    //cin >> _;
    while(_--) {
        solve();
    }
    return 0;
}

```



## 数学

### 质因数分解

```c++
void a() {
    bool isprime[N];
    int prime[N], minp[N];
    int cnt = 0;
    vector<pair<int, int>> shu[N];

    minp[1] = 1;
    for (int i = 2; i < N; i++) {
        if (!isprime[i]) {
            prime[++cnt] = i;
            minp[i] = i;
        }
        for (int j = 1; j <= cnt && i * prime[j] < N; j++) {
            isprime[i * prime[j]] = 1;
            minp[i * prime[j]] = prime[j];
            if (i % prime[j] == 0) break;
        }
    }

    for (int i = 1; i < N; i++) {
        map<int, int> mp;
        int tmp = i;
        while (tmp != 1) {
            mp[minp[tmp]]++;
            tmp /= minp[tmp];
        }
        for (auto [x, y] : mp) {
            shu[i].push_back({x, y});
        }   
    }
} 
```

### 组合数阶层预处理

```c++
int ksm(int a, int b) {
    int res = 1;
    while (b) {
        if (b & 1) res = res * a % mod;
        b >>= 1;
        a = a * a % mod;
    }
    return res;
}

int jc[N], inv[N];

void init() {
    jc[0] = 1;
    for (int i = 1; i < N; i++) jc[i] = jc[i - 1] * i % mod;
    inv[N - 1] = ksm(jc[N - 1], mod - 2);
    for (int i = N - 2; i >= 0; i--) {
        inv[i] = inv[i + 1] * (i + 1) % mod; 
    }
}

int C(int n, int k) {
    if (k < 0 || k > n) return 0;
    return (jc[n] * inv[k] % mod * inv[n - k] % mod);
}
```

### 求一个数的欧拉函数

```c++
int oula(int n) {
    int ans = n;
    for (int i = 2; i * i <= n; i++) {
        if (n % i == 0) {
            ans = ans / i * (i - 1);
            while (n % i == 0) n /= i;
        }
    }
    if (n > 1) ans = ans / n * (n - 1);
    return ans;
}
```

### 筛法求欧拉函数表

```c++
vector<int> pri;
bool notprime[N];
int phi[N];
void pre(int n) {
    phi[1] = 1;
    for (int i = 2; i <= n; i++) {
        if (!notprime[i]) {
            pri.push_back(i);
            phi[i] = i - 1;
        }
        for (int prij : pri) {
            if (i * prij > i) break;
            notprime[i * prij] = 1;
            if (i % prij == 0) {
                phi[i * prij] = phi[i] * prij;
                break;
            }
            phi[i * prij] = phi[i] * phi[prij];
        }
    }
}
```

### 预处理所有因数

```c++
//nlogn
int N = (int)1e5;
vector<int> num[N];
int init() {
    for (int i = 1; i < N; i++) {
        for (int j = i; j < N; j += i) {
            num[j].push_back(i);
        }
    }
}
```

### 计数模型

#### 隔板法（Stars and Bars）：分配/拆分的万能工具

- 非负整数解个数：

  $$x_1+\cdots+x_m = n,\ x_i\ge 0 \Rightarrow \binom{n+m-1}{m-1}$$

- 正整数解个数：

  $$x_1+\cdots+x_m = n,\ x_i\ge 1 \Rightarrow \binom{n-1}{m-1}$$

- 上界约束（$x_i\le b_i$）通常结合**容斥**：先不管上界用隔板法计数，再对超界变量做减法计数。

#### 经典数列与标准计数模型（经常直接套）

##### 4.1 错排（Derangement）

- $D_n$：没有元素在原位的排列数

  $$D_n = n!\sum_{k=0}^n \frac{(-1)^k}{k!}$$

- 递推：$$D_n=(n-1)(D_{n-1}+D_{n-2})$$

##### 4.2 卡特兰数（Catalan）

常见场景：合法括号序列、栈序列、二叉树形状、路径不越界等

- 定义：

  $$C_n=\frac{1}{n+1}\binom{2n}{n}$$

  - $1, 1, 2, 5, 14, 42, 132, 429, 1430, 4862, 16796, ...$

- 变体（Ballot/Reflection）：路径计数常用“反射法”得到不越界的组合数差。

##### 4.3 斯特林数（Stirling）

- 第二类 $S(n,k)$：把 $n$ 个元素分成 $k$ 个**非空无标号**集合

  $$S(n,k)=S(n-1,k-1)+kS(n-1,k)$$

  常用于“划分成 k 组”的题。

- 第一类（有符号/无符号）与“排列的环结构”相关：把排列分成 kkk 个循环。

### 线性基

```c++
#include <bits/stdc++.h>
using namespace std;

struct Basis {
    static const int K = 60;              // long long 常用到 0~60 位

    long long b[K + 1]{};                 // 原始线性基：b[k] 的最高位是 k
    unsigned long long m[K + 1]{};        // b[k] 对应的“独立向量选择状态”

    long long rb[K + 1]{};                // 规范化后的线性基
    unsigned long long rm[K + 1]{};       // 规范化后对应的方案状态

    int id[64]{};                         // 第 t 个独立向量对应原数组下标
    int c = 0;                            // 线性基维数（独立向量个数）

    bool built = false;                   // 是否已经做过规范化

    // 插入一个数 x，原下标为 idx
    void ins(long long x, int idx) {
        if (!x) return;
        unsigned long long s = 1ULL << c; // 先认为它自己是一个新的独立向量

        for (int k = K; k >= 0; --k) {
            if (((x >> k) & 1LL) == 0) continue;

            // 当前最高位还没有基底，直接放进去
            if (!b[k]) {
                b[k] = x;
                m[k] = s;
                id[c++] = idx;
                built = false;
                return;
            }

            // 否则用已有基底消掉最高位
            x ^= b[k];
            s ^= m[k];
        }

        // 如果最后 x 变成 0，说明它可以被已有线性基表示，不增加维数
    }

    // 判断 x 是否可被当前线性基表示
    // 若可表示，则 s 返回所选独立向量状态
    bool rep(long long x, unsigned long long &s) const {
        s = 0;
        for (int k = K; k >= 0; --k) {
            if (((x >> k) & 1LL) == 0) continue;
            if (!b[k]) return false;
            x ^= b[k];
            s ^= m[k];
        }
        return true;
    }

    // 把当前线性基规范化：
    // 1. 先拷贝一份
    // 2. 再把每个基底中的低位尽量消掉
    //
    // 规范化后有个很实用的性质：
    // - 最小非零异或值 = 所有非零规范基底中的最小值
    void build() {
        for (int k = 0; k <= K; ++k) {
            rb[k] = b[k];
            rm[k] = m[k];
        }

        // 从高位往低位做消元：
        // 让每个 rb[i] 的低位尽量干净
        for (int i = K; i >= 0; --i) {
            if (!rb[i]) continue;
            for (int j = i - 1; j >= 0; --j) {
                if (((rb[i] >> j) & 1LL) == 0) continue;
                if (!rb[j]) continue;
                rb[i] ^= rb[j];
                rm[i] ^= rm[j];
            }
        }

        built = true;
    }

    // 求可表示的最大异或值
    // s 返回所选独立向量状态
    long long queryMax(unsigned long long &s) const {
        long long res = 0;
        s = 0;
        for (int k = K; k >= 0; --k) {
            if (!b[k]) continue;
            if ((res ^ b[k]) > res) {
                res ^= b[k];
                s ^= m[k];
            }
        }
        return res;
    }

    // 如果允许空集，最小异或值永远是 0
    long long queryMinZero(unsigned long long &s) const {
        s = 0;
        return 0;
    }

    // 求最小非零异或值
    // 若不存在非零可表示值（即线性基为空），返回 -1
    // s 返回所选独立向量状态
    long long queryMinNonZero(unsigned long long &s) {
        if (!built) build();

        long long ans = -1;
        unsigned long long state = 0;

        for (int k = 0; k <= K; ++k) {
            if (!rb[k]) continue;
            if (ans == -1 || rb[k] < ans) {
                ans = rb[k];
                state = rm[k];
            }
        }

        s = state;
        return ans;
    }

    // 把状态 s 还原成原数组下标集合
    vector<int> recover(unsigned long long s) const {
        vector<int> pos;
        for (int t = 0; t < c; ++t) {
            if ((s >> t) & 1ULL) pos.push_back(id[t]);
        }
        return pos;
    }
};

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    // 示例：读入一组数，输出：
    // 1) 最大异或值
    // 2) 达到最大异或值所选的原下标
    // 3) 最小非零异或值
    // 4) 达到最小非零异或值所选的原下标
    //
    // 输入格式：
    // n
    // a1 a2 ... an

    int n;
    cin >> n;
    vector<long long> a(n + 1);

    Basis lb;
    for (int i = 1; i <= n; ++i) {
        cin >> a[i];
        lb.ins(a[i], i);
    }

    // 求最大异或值
    unsigned long long sMax;
    long long mx = lb.queryMax(sMax);
    auto maxPos = lb.recover(sMax);

    cout << "最大异或值 = " << mx << '\n';
    cout << "达到最大异或值所选下标：";
    if (maxPos.empty()) cout << "空集";
    else {
        for (int i = 0; i < (int)maxPos.size(); ++i) {
            cout << maxPos[i] << (i + 1 == (int)maxPos.size() ? '\n' : ' ');
        }
    }
    if (!maxPos.empty()) {
        cout << "对应选中的数：";
        for (int i = 0; i < (int)maxPos.size(); ++i) {
            cout << a[maxPos[i]] << (i + 1 == (int)maxPos.size() ? '\n' : ' ');
        }
    } else {
        cout << "对应选中的数：空集\n";
    }

    // 求最小非零异或值
    unsigned long long sMin;
    long long mn = lb.queryMinNonZero(sMin);

    if (mn == -1) {
        cout << "最小非零异或值不存在（线性基为空）\n";
    } else {
        auto minPos = lb.recover(sMin);
        cout << "最小非零异或值 = " << mn << '\n';
        cout << "达到最小非零异或值所选下标：";
        for (int i = 0; i < (int)minPos.size(); ++i) {
            cout << minPos[i] << (i + 1 == (int)minPos.size() ? '\n' : ' ');
        }
        cout << "对应选中的数：";
        for (int i = 0; i < (int)minPos.size(); ++i) {
            cout << a[minPos[i]] << (i + 1 == (int)minPos.size() ? '\n' : ' ');
        }
    }

    return 0;
}
```

### 01字典树

```c++
#include <bits/stdc++.h>
using namespace std;

struct BinaryTrie {
    static const int LOG = 60;  // 如果题目是 int 非负数，通常开到 30 即可
    // 若题目是 long long，可改成 60

    struct Node {
        int ch[2];   // 0/1 两个儿子
        int cnt;     // 经过这个节点的数的个数
        Node() {
            ch[0] = ch[1] = 0;
            cnt = 0;
        }
    };

    vector<Node> tr;

    BinaryTrie() {
        tr.reserve(1 << 20);
        tr.push_back(Node()); // 0 号点不用
        tr.push_back(Node()); // 1 号点作为根
    }

    // 插入一个数 x
    void insert(int x) {
        int u = 1;
        tr[u].cnt++;
        for (int k = LOG; k >= 0; --k) {
            int b = (x >> k) & 1;
            if (!tr[u].ch[b]) {
                tr[u].ch[b] = (int)tr.size();
                tr.push_back(Node());
            }
            u = tr[u].ch[b];
            tr[u].cnt++;
        }
    }

    // 删除一个数 x
    // 要求：x 之前一定被插入过，且当前还在 trie 中
    void erase(int x) {
        int u = 1;
        tr[u].cnt--;
        for (int k = LOG; k >= 0; --k) {
            int b = (x >> k) & 1;
            u = tr[u].ch[b];
            tr[u].cnt--;
        }
    }

    // 查询：集合中与 x 异或最大的那个数
    // 返回 {最大异或值, 配对的数}
    pair<int, int> askmax(int x) const {
        int u = 1;
        if (tr[u].cnt == 0) return {-1, -1}; // 空集合

        int y = 0;
        for (int k = LOG; k >= 0; --k) {
            int b = (x >> k) & 1;
            int want = b ^ 1; // 想走相反位，使当前位异或为 1

            if (tr[u].ch[want] && tr[tr[u].ch[want]].cnt > 0) {
                y |= (want << k);
                u = tr[u].ch[want];
            } else {
                int go = b;
                y |= (go << k);
                u = tr[u].ch[go];
            }
        }
        return {x ^ y, y};
    }

    // 查询：集合中与 x 异或最小的那个数
    // 返回 {最小异或值, 配对的数}
    pair<int, int> askmin(int x) const {
        int u = 1;
        if (tr[u].cnt == 0) return {-1, -1}; // 空集合

        int y = 0;
        for (int k = LOG; k >= 0; --k) {
            int b = (x >> k) & 1;

            // 想走相同位，使当前位异或为 0
            if (tr[u].ch[b] && tr[tr[u].ch[b]].cnt > 0) {
                y |= (b << k);
                u = tr[u].ch[b];
            } else {
                int go = b ^ 1;
                y |= (go << k);
                u = tr[u].ch[go];
            }
        }
        return {x ^ y, y};
    }

    // 判断当前 trie 是否为空
    bool empty() const {
        return tr[1].cnt == 0;
    }
};

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    BinaryTrie T;

    // 示例：先插入一些数
    vector<int> a = {3, 10, 5, 25, 2, 8};
    for (int x : a) T.insert(x);

    int x = 5;

    auto [mx, y1] = T.askmax(x);
    cout << "与 " << x << " 异或最大值 = " << mx << '\n';
    cout << "配对的数是 " << y1 << "，因为 " << x << " ^ " << y1 << " = " << mx << '\n';

    auto [mn, y2] = T.askmin(x);
    cout << "与 " << x << " 异或最小值 = " << mn << '\n';
    cout << "配对的数是 " << y2 << "，因为 " << x << " ^ " << y2 << " = " << mn << '\n';

    return 0;
}
```

### 扩展欧几里得

```c++
int exgcd(int a, int b, int& x, int& y) {
    if (b == 0) {
        x = 1, y = 0;
        return a;
    }
    int x1, y1;
    int g = exgcd(b, a % b, x1, y1);
    x = y1;
    y = x1 - (a / b) * y1;
    return g;
}
```

## 计算几何

### 极角排序

```c++
struct ty {
    int x, y;
};

bool cmp(ty a, ty b) {
    int ah = (a.y < 0 || (a.y == 0 && a.x < 0));
    int bh = (b.y < 0 || (b.y == 0 && b.x < 0));
    if (ah != bh) return ah < bh;
    return a.x * b.y - a.y * b.x > 0;
}
```

## 动态规划

### 最长上升子序列

```c++
    int len = 0;
    vector<int> dp(n + 5);
    for (int i = 1; i <= n; i++) {
        if (dp[len] < a[i]) {
            dp[++len] = a[i];
        }
        else {
            *lower_bound(dp.begin() + 1, dp.begin() + len + 1, a[i]) = a[i];
        }
    }
    cout << len << "\n";
```

### 数位dp

```c++
LL dfs(int pos, bool limit, int sum)
{
    if(!pos) //递归边界
        return sum;
    if(!limit && ~f[pos][sum]) //没限制并且dp值已搜索过
        return f[pos][sum];
    int up = limit ? a[pos] : 9;
    LL res = 0;
    for(int i = 0; i <= up; i ++)
        res = (res + dfs(pos - 1, limit && i == up, sum + i)) % md;
    if(!limit) //记搜，可复用
        f[pos][sum] = res;
    return res;
}
```

## 字符串

### 字符串哈希

```c++
// 单哈希
#include <bits/stdc++.h>
using namespace std;

using ull = unsigned long long;
const ull base = 131;

struct StringHash {
    int n;
    string s;              // 1-based
    vector<ull> h, p;      // 前缀哈希、base幂
    // !!!!!后面用到的时候一定不要忘记是ull！！！！！！
    StringHash() {}
    StringHash(const string& str) {
        init(str);
    }

    void init(const string& str) {
        n = (int)str.size();
        s = " " + str;
        h.assign(n + 1, 0);
        p.assign(n + 1, 0);
        p[0] = 1;
        for (int i = 1; i <= n; i++) {
            h[i] = h[i - 1] * base + (ull)s[i];
            p[i] = p[i - 1] * base;
        }
    }

    ull get(int l, int r) {
        return h[r] - h[l - 1] * p[r - l + 1];
    }
};

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    string s;
    cin >> s;

    StringHash hs(s);

    int q;
    cin >> q;
    while (q--) {
        int l1, r1, l2, r2;
        cin >> l1 >> r1 >> l2 >> r2;
        if ((r1 - l1) != (r2 - l2)) {
            cout << "No\n";
        } else {
            cout << (hs.get(l1, r1) == hs.get(l2, r2) ? "Yes\n" : "No\n");
        }
    }

    return 0;
}



// 双哈希
#include <bits/stdc++.h>
using namespace std;

const int MOD1 = 998244353;
const int MOD2 = 1000000007;
const int BASE = 131;

struct HashVal {
    int x, y;
    HashVal(int _x = 0, int _y = 0) : x(_x), y(_y) {}
    bool operator == (const HashVal& other) const {
        return x == other.x && y == other.y;
    }
};

HashVal operator + (const HashVal& a, const HashVal& b) {
    int nx = a.x + b.x;
    int ny = a.y + b.y;
    if (nx >= MOD1) nx -= MOD1;
    if (ny >= MOD2) ny -= MOD2;
    return HashVal(nx, ny);
}

HashVal operator - (const HashVal& a, const HashVal& b) {
    int nx = a.x - b.x;
    int ny = a.y - b.y;
    if (nx < 0) nx += MOD1;
    if (ny < 0) ny += MOD2;
    return HashVal(nx, ny);
}

HashVal operator * (const HashVal& a, const HashVal& b) {
    return HashVal(
        1LL * a.x * b.x % MOD1,
        1LL * a.y * b.y % MOD2
    );
}

struct StringHash {
    int n;
    string s;   // 1-based
    vector<HashVal> h, p;

    void init(const string& str) {
        n = (int)str.size();
        s = " " + str;
        h.assign(n + 1, HashVal());
        p.assign(n + 1, HashVal());
        p[0] = HashVal(1, 1);
        HashVal base(BASE, BASE);
        for (int i = 1; i <= n; i++) {
            int val = s[i];
            h[i] = h[i - 1] * base + HashVal(val, val);
            p[i] = p[i - 1] * base;
        }
    }

    HashVal get(int l, int r) {
        return h[r] - h[l - 1] * p[r - l + 1];
    }
};
```

### string内置函数

---

#### 1) `cin >>` 后接 `getline`

```cpp
int n; 
cin >> n;
cin.ignore();
string line;
getline(cin, line); // 读一整行（可为空）
```

#### 2) 基础
```cpp
string s = "hello";
s.push_back('!');      // 追加字符
s.pop_back();          // 删除末尾字符
s += " world";         // 追加字符串
s.assign("new");       // 重新赋值
int n = (int)s.size(); // 长度
```

#### 3) 查找 find / rfind
```cpp
string s = "a/b/c";
size_t p1 = s.find('/');      // 第一次出现
size_t p2 = s.rfind('/');     // 最后一次出现
if (s.find("b/") != string::npos) { /* 存在子串 */ }

string s = "abcabcabc";
int pos = s.find("abc", 3);   // 从某个位置开始找
```

#### 4) 截取 substr
```cpp
string s = "abcdefg";
cout << s.substr(3) << "\n";     // defg：从 pos 到末尾
cout << s.substr(2, 3) << "\n";  // cde：从 pos 开始取 len 个
```

#### 5) 替换

5.1 s.replace(pos, cnt, str);
```cpp
string s = "abcdef";
s.replace(1, 3, "xyz"); // bcd -> xyz
// s == "axyzef"
// 把 s[pos ... pos+cnt-1] 替换成 str
```

5.2 s.replace(pos, cnt, num, ch);
```cpp
string s = "abcdef";
s.replace(2, 3, 4, 'x');
// abxxxxf
// 把一段替换成 num 个 ch
```

5.2 std::replace（替换单个字符）
```cpp
string s = "12,34,56";
replace(s.begin(), s.end(), ',', ' '); // 逗号变空格
// s == "12 34 56"
```

#### 6) 字符串 ↔ 数字
```cpp
6.1 字符串 -> 数字 stoi / stoll

6.2 数字 -> 字符串 to_string
```

## 杂项

### int128

```c++
using i128 = __int128;

void print128(i128 x) {
    if (x == 0) { cout << 0; return; }
    if (x < 0) { cout << '-'; x = -x; }
    string s;
    while (x) {
        s += '0' + x % 10;
        x /= 10;
    }
    reverse(s.begin(), s.end());
    cout << s;
}

i128 read128() {
    string s;
    cin >> s;
    i128 x = 0, sign = 1;
    int i = 0;
    if (s[0] == '-') sign = -1, i++;
    for (; i < s.size(); i++) x = x * 10 + (s[i] - '0');
    return x * sign;
}
```







