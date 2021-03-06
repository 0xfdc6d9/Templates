# Nepenthe8's tricks

## 模型

### 柱形图最大矩阵面积

给出一个 $n \times m$ 矩阵，要求从中找到每一列都是不下降数列的最大子矩阵，输出它的大小。

使用单调栈解决。[HDU-6957](https://vjudge.net/problem/HDU-6957)

~~~c++
#include <bits/stdc++.h>
using namespace std;
using ll = long long;
const int N = 2010;

ll n, m;
ll mp[N][N], h[N][N];

ll cal(ll h[]) {
    h[m + 1] = 0;
    stack<ll> sta;
    ll ans = 0;
    for (ll i = 1; i <= n; i++) {
        while (!sta.empty() && h[sta.top()] > h[i]) {
            ll now_h = h[sta.top()]; //栈顶矩形将会被弹出，以弹出的单个矩形条的高计算矩形面积
            sta.pop();
            ans = max(ans, ((i - 1) - (sta.empty() ? 1 : (sta.top() + 1)) + 1) * now_h); //确定左右边界(rt-lf+1)
        }
        sta.push(i);
    }
    while (!sta.empty()) {
        ll now_h = h[sta.top()];
        sta.pop();
        ans = max(ans, (m - (sta.empty() ? 1 : (sta.top() + 1)) + 1) * now_h); //这里的m为高度数组h的右端点位置
    }
    return ans;
}

int main() {
    ios::sync_with_stdio(false); cin.tie(0); cout.tie(0);
    int T; cin >> T;
    while (T--) {
        cin >> n >> m;
        for (int i = 1; i <= n; i++)
            for (int j = 1; j <= m; j++) {
                cin >> mp[i][j];
                if (mp[i][j] >= mp[i - 1][j])
                    h[i][j] = h[i - 1][j] + 1;
                else
                    h[i][j] = 1;
            }
        ll ans = 0;
        for (int i = 1; i <= n; i++)
            ans = max(ans, cal(h[i]));
        cout << ans << "\n";
    }
    return 0;
}
~~~

### 求每个元素产生的逆序对数量

[P3149 - 排序](https://www.luogu.com.cn/problem/P3149)

~~~c++
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
const ll N = 300010;

struct Node {
    ll val, pos; //pos为第几个出现
    ll opt; //该点贡献的逆序对
    bool operator<(const Node& t) const {
        if (val == t.val) return pos < t.pos;
        else return val < t.val;
    }
} a[N];
ll tr[N], ta[N], atot;
ll n, m;

ll num[N]; //每个元素的逆序对贡献

void update(ll x, ll k) {
    for (; x <= n; x += x & -x)
        tr[x] += k;
}

ll query(ll x) {
    ll res = 0;
    for (; x; x -= x & -x) 
        res += tr[x];
    return res;
}

int main() {
    ios::sync_with_stdio(false); cin.tie(0); cout.tie(0);
    //init
    // memset(tr, 0, sizeof(tr));
    // atot = 0;
    
    cin >> n >> m;
    for (ll i = 1; i <= n; i++) {
        cin >> a[i].val;
        a[i].pos = i;
        ta[++atot] = a[i].val;
    }

    sort(ta + 1, ta + 1 + atot);
    atot = unique(ta + 1, ta + 1 + atot) - ta - 1; //atot为离散后的表长

    for (ll i = n; i >= 1; i--) { //倒着求就可以算出每个点所产生的逆序对的数量
        a[i].val = lower_bound(ta + 1, ta + 1 + atot, a[i].val) - ta; //离散化，即原来的数->原来的数是第几小的
        a[i].opt = query(a[i].val - 1); //在他之后还比他小的 即 a[i].val贡献的逆序对数量
        // cout << a[i].opt << endl;
        num[i] = a[i].opt;
        update(a[i].val, 1);
    }
    for (int i = 1; i <= n; i++) {
        cout << num[i] << " \n"[i == n];
    }
    return 0;
}
~~~

### 运算式带括号的处理

若可以改变运算数的相对位置。

本质上就是将给定的运算数任选两个进行运算，考虑暴力枚举出所有的运算数组合，即可认为是枚举出所有带括号的情况。

如[24dian](https://ac.nowcoder.com/acm/contest/view-submission?submissionId=48558895)。

~~~c++
/* find all the possible set of cards that has a valid solution but any solution involves fraction in the calculation.  */
/* 找到所有可能的具有有效解决方案的卡片组，但任何解决方案都涉及计算中的分数。 */

bool curF(double x) { //出现分数？
    return fabs(ll(x) - x) > eps;
}

vector<vector<double> > ans;
bool ok = 0;
bool allCurF = 1; //是否所有的可行方案都存在分数

void dfs2(vector<double> a, int &m, bool tg) {
    if ((int)a.size() == 1) {
        if (fabs(a[0] - m) < eps) {
            if (!tg) 
                allCurF = 0;
            ok = 1;
        }
        return;
    }
    for (int i = 0; i < (int)a.size(); i++) {
        for (int j = 0; j < (int)a.size(); j++) {
            if (i == j)
                continue;
            vector<double> tmp = a;
            double t1 = a[i], t2 = a[j];
            //+
            a.erase(a.begin() + max(i, j)); //保证删除正确位置的元素
            a.erase(a.begin() + min(i, j));
            a.emplace_back(t1 + t2);
            dfs2(a, m, tg);

            //-
            a = tmp;
            a.erase(a.begin() + max(i, j));
            a.erase(a.begin() + min(i, j));
            a.emplace_back(t1 - t2);
            dfs2(a, m, tg);

            //*
            a = tmp;
            a.erase(a.begin() + max(i, j));
            a.erase(a.begin() + min(i, j));
            a.emplace_back(t1 * t2);
            dfs2(a, m, tg);

            //divide
            if (t2 != 0) {
                a = tmp;
                a.erase(a.begin() + max(i, j));
                a.erase(a.begin() + min(i, j));
                a.emplace_back(t1 / t2);
                dfs2(a, m, tg | curF(t1 / t2));
            }

            //restore
            a = tmp;
        }
    }
}

void dfs1(vector<double> a, int &m, int Last) { //顺序枚举4个数
    if ((int)a.size() == 4) {
        ok = 0;
        allCurF = 1;
        dfs2(a, m, 0);
        if (ok && allCurF) {
            ans.push_back(a);
        }
        return;
    }
    for (int i = Last; i <= 13; i++) {
        a.push_back((double)i);
        dfs1(a, m, i);
        a.pop_back();
    }
}

int main() {
    ios::sync_with_stdio(false); cin.tie(0); cout.tie(0);
    int n, m; cin >> n >> m;
    if (n < 4) {
        cout << 0 << "\n";
        return 0;
    }
    dfs1({}, m, 1);
    ll sz = ans.size();

    cout << sz << "\n";
    for (int i = 0; i < sz; i++) {
        for (int j = 0; j < 4; j++) {
            cout << ans[i][j] << " \n"[j == 3];
        }
    }
    return 0;
}
~~~

### 求两个序列公共子序列的数量

~~~c++
for (int i = 1; i <= na; i++) { //找a、b公共子序列的数量
    for (int j = 1; j <= nb; j++) {
        if (a[i] != b[j])
            (dp[i][j] = dp[i - 1][j] + dp[i][j - 1] - dp[i - 1][j - 1] + mod) %= mod;
        else
            // dp[i][j] = dp[i - 1][j] + dp[i][j - 1] - dp[i - 1][j - 1] + (dp[i - 1][j - 1] + 1);
            //a、b末尾字符不一样的方案数+(a、b末尾字符一样，可以接上去，为dp[i - 1][j - 1]种方案 + a[i]和b[j]也可以形成一个公共子序列)
            (dp[i][j] = dp[i - 1][j] + dp[i][j - 1] + 1) %= mod;
    }
}
~~~

[例题](https://ac.nowcoder.com/acm/contest/view-submission?submissionId=48654923&headNav=acm)

## STL

### 容器

#### deque

deque 是一种优化了的、对序列两端元素进行添加和删除操作的基本序列容器。它允许较为快速地随机访问，但它不像 vector 把所有的对象保存在一块连续的内存块，而是采用多个连续的存储块，并且在一个映射结构中保存对这些块及其顺序的跟踪。向 deque 两端添加或删除元素的开销很小。它不需要重新分配空间，所以向末端增加元素比 vector 更有效。

在两端进行push和pop的时间复杂度都为$O(1)$，随机访问的时间复杂度也为$O(1)$。

[参考](https://blog.csdn.net/like_that/article/details/98446479)

#### String

##### 构造函数

一些常见的构造函数形式

~~~c++
    string a; //定义一个空字符串
    string str_1 (str); //构造函数，全部复制
    string str_2 (str, 2, 5); //构造函数，从字符串str的第2个元素开始，复制5个（长度）元素，赋值给str_2
    string str_3 (ch, 5); //将字符串ch的前5个元素赋值给str_3
    string str_4 (5,'X'); //将 5 个 'X' 组成的字符串 "XXXXX" 赋值给 str_4
    string str_5 (str.begin(), str.end()); //复制字符串 str 的所有元素，并赋值给 str_5
~~~

##### erase() 和 substr()

substr() 没有参数为迭代器的重载

~~~c++
string str("0123456789");
cerr << str.substr(2, 4) << endl; //获得str中从2开始且长度为4的子串，输出2345
~~~

erase() 有整数或迭代器的重载

~~~c++
// 假定下面的操作都是相互独立的
//使用下标
str.erase(3);//从3开始到最后，输出012
str.erase(3, 5);//从3开始删除五个元素，即删除34567，输出01289

//使用迭代器
str.erase(str.begin() + 3);//删除位置3的元素，输出012456789
str.erase(str.begin() + 3,str.end());//删除位置3到末尾的元素（尾后迭代器之前），输出012
str.erase(str.begin() + 3, str.begin() + 8);//删除位置[3, 8)的元素，输出01289
~~~

##### 原始字符串

~~~c++
cout << R"(Hello,"C++".)" << endl; // Hello,"C++".
~~~

R"()" 中包裹的字符串就是原始字符，不用转义序列，空格回车都包含其中。

原始字符串会在遇到第一个 )" 时结束，如果字符串中包含 )" ，那么可以添加任意数量的对称字符来自定义界定符，但是注意空格、左括号、右括号、斜杠和控制字符（如制表符和换行符）除外。

~~~c++
cout << R"*("(hey)" + *()* hello.)*" << endl; // "(hey)" + *()* hello.
~~~

##### find()

对于形如 $s.find(t)$ 这样的调用，若在 $s$ 中找到 $t$，则返回 $t$ 第一次在 $s$ 中出现的下标。

~~~c++
string a = "1111", b = "1010", c = "101011010", d = "11010";
cout << a.find(b) << endl; // 18446744073709551615
cout << c.find(b) << endl; // 0
cout << d.find(b) << endl; // 1
~~~

#### bitset 相关函数

对于一个叫做 foo 的 bitset ：
`foo.size()` 返回大小（位数）
`foo.count()` 返回1的个数
`foo.any()` 返回是否有1
`foo.none()` 返回是否没有1
`foo.set()` 全都变成1
`foo.set(p)` 将第p + 1位变成1
`foo.set(p, x)` 将第p + 1位变成x
`foo.reset()` 全都变成0
`foo.reset(p)` 将第p + 1位变成0
`foo.flip()` 全都取反
`foo.flip(p)` 将第p + 1位取反
`foo.to_ulong()` 返回它转换为unsigned long的结果，如果超出范围则报错
`foo.to_ullong()` 返回它转换为unsigned long long的结果，如果超出范围则报错
`foo.to_string()` 返回它转换为string的结果

#### 二维 vector 初始化

~~~c++
// 全部初始化为1
// 方法1
vector<vector<int>> vec(row, vector<int> (col,1));

//方法2
vector<vector<int>> vec;
vec.resize(row);
for(int i = 0; i < vec.size(); i++)
    vec[i].resize(col);
for(int i = 0; i < row; i++)
    for (int j = 0; j < col; j++)
        vec[i][j] = 1;
~~~

#### tuple

tuple 实例化的对象可以存储任意数量、任意类型的数据。

~~~c++
// 通过下标进行访问和赋值
tuple<int, int, string> a;
// get<int>(a) = 1; // compile error
get<0>(a) = 1;
cout << get<0>(a) << " " << get<1>(a) << " " << get<2>(a) << endl; // 1 0

// c++14以上，也可以通过类型名（unique）进行访问和赋值
tuple<int, double, string> b;
get<int>(b) = 1;
get<string>(b) = "hello";
// get<char>(b) = 'a'; // compile error
cout << get<0>(b) << " " << get<1>(b) << " " << get<2>(b) << endl; // 1 0 hello
~~~

### 函数

#### lower_bound()

返回第一个值大于等于 x 的数的下标。

##### 返回值

~~~c++
vector<int> a = {1, 2, 5};
auto p = lower_bound(a.begin(), a.end(), 1);
cerr << (p == a.begin()) << endl; //True

auto p1 = lower_bound(a.begin(), a.end(), 5);
cerr << (p1 == a.end() - 1) << endl; //True

auto p2 = lower_bound(a.begin(), a.end(), 8);
cerr << (p2 == a.end()) << endl; //True
~~~

##### 在降序序列中使用

如果需要在一个降序的序列 $a$ 里面进行二分查找，可以考虑新建一个序列 $b$，$b$ 中存放 $a$ 对应位置的相反数，就可以继续使用 lower_bound。

##### 在结构体或者vector中使用

如果需要在结构体或者 vector 进行 lower_bound 和 upper_bound，把我们需要查找的数**封装成一个结构体**，才可以在结构体中进行查找。即使我们只需要针对某一维进行查找，也需要把整个结构体构造出来。

~~~c++
struct Node {
    int a, b;
    Node() {}
    Node(int a, int b) : a(a), b(b) {}
    bool operator<(const Node m) const { return b < m.b; } //定义比较方式，定义查找哪一维，使用重载运算符，lambda表达式报错
};

int main() {
    vector<Node> v;
    for (int i = 0; i < 10; i++) {
        v.push_back(Node(10 - i + 1, 2 * i));
        cout << v[i].a << "  " << v[i].b << endl;
    }
    int num; cin >> num;
    sort(v.begin(), v.end()); //进行二分之前需要排序
    int pos = lower_bound(v.begin(), v.end(), Node(0, num)) - v.begin(); //需要把我们查找的数封装成一个结构体才能进行查找。
    // int pos = lower_bound(v.begin(), v.end(), Node(0, num), [](const Node &A, const Node &B) { return A.b < B.b; }) - v.begin();
    cout << pos << endl;
    return 0;
}
~~~

##### 离散化

~~~c++
auto getRid = [&](int x) {
        return lower_bound(rows.begin(), rows.end(), x) - rows.begin() + 1;
    };
~~~

不加 1，表示取离散化后下标为 id 的元素；加 1，表示取离散化后第 id 个元素。

##### 注意

~~~c++
multiset<int> ms;
...
auto p = ms.lower_bound(last + 1);
if (p == ms.end() || *p != last + 1) {
    ok = false;
    break;
}
~~~

在 set/multiset 中使用二分时也应注意判断是否不存在满足条件的值，即与 end() 进行比较。

比如

~~~c++
multiset<int> ms = {0, 0}
auto p = ms.lower_bound(2);
//p == ms.end()
~~~

#### __builtin_popcount()

__builtin_popcount()用于计算一个 32 位无符号整数有多少个位为1

- __builtin_popcount = int
- __builtin_popcountl = long int
- __builtin_popcountll = long long

#### partial_sum()与adjacent_difference()

partial_sum()用于求前缀和，adjacent_difference()用于求差分。

差分的每次修改都相当于对[L:]区间进行修改。

~~~c++
int a[5] = {15, 10, 6, 3, 1};
    int b[5] = {0};
    int c[5] = {0};
    adjacent_difference(a, a + 5, b); //15, -5, -4, -3, -2
    partial_sum(a, a + 5, c); //15, 25, 31, 34, 35
    for (int i = 0; i < 5; i++)
        cout << c[i] << ", ";
~~~

#### accumulate()

假设 vec 是一个 int 型的 vector 对象，下面的代码：

```c++
//sum the elements in vec starting the summation with the value 42
int sum = accumulate(vec.begin() , vec.end() , 42);
```

将 sum 设置为 vec 的元素之和再加上 42 。

accumulate 带有**三个形参**：头两个形参指定要**累加的元素范围**，第三个形参则是**累加的初值**。

accumulate 函数将它的一个内部变量设置为指定的初始值，然后在此初值上累加输入范围内所有元素的值。accumulate 算法返回累加的结果，**其返回类型就是其第三个实参的类型**。

用于指定累加起始值的第三个参数是必要的，因为 accumulate 对将要累加的元素类型一无所知，除此之外，没有别的办法创建合适的起始值或者关联的类型。

#### iota()

定义在 numeric 头文件中的 iota() 函数模板会用连续的 T 类型值填充序列。前两个参数是定义序列的正向迭代器，第三个参数是初始的 T 值。第三个指定的值会被保存到序列的第一个元素中。保存在第一个元素后的值是通过对前面的值运用自增运算符得到的。当然，这意味着 T 类型必须支持 operator++()。

~~~c++
vector<int> a(5);
iota(a.begin(), a.end(), 2);
for (auto i : a) {
    cout << i << " ";
}
//2 3 4 5 6 
~~~

#### prev()、next()、advance()

例如在多重集合 $S$ 中求：

- Find the minimum element in $S$ that is greater than $x$
- Find the maximum element in $S$ that is less than $x$

~~~c++
multiset<int> ms = {-1, 0, 1, 3, 3, 3, 7};
auto p = ms.lower_bound(3);
debug(*p); //## *p = 3
debug(*prev(p)); //## *prev(p) = 1
debug(*prev(p, 2)); //## *prev(p, 2) = 0
advance(p, 3); //第二个参数可正可负
debug(*p); //## *p = 7
~~~

#### hypot()

返回欧几里德范数 $\sqrt{x^2+y^2} $。

#### 进制转换

##### 将任意字符串形式的 2~36 进制数转化为 10 进制数

使用 strtol()函数。

函数原型：long int strtol(const char *nptr, char **endptr, int base)

base 是要转化的数的进制，非法字符会赋值给 endptr，nptr 是要转化的字符。

~~~c++
string s = "0x16opsd";
char *fw; //非法字符
int a = strtol(s.c_str(), &fw, 16); //22, 0x16 = 22
s = "123opop";
int b = strtol(s.c_str(), &fw, 4); //将字符串中的四进制数(123)_4 -> 十进制数(27)_10
cout << a << endl; // 22
cout << b << endl; // 27
cout << fw << endl; // opop
~~~

##### 将 10 进制数转化为任意字符串形式的 2~36 进制数

使用 itoa() 函数。

函数原型：char* itoa(int value, char* string, int radix);

例如：itoa(num,str,2); num 是一个 int 型的，是要转换的 10 进制数，str 是转换结果，后面的值为目标进制。

~~~c++
int num = 27;
char ret[6]; // 转换结果
_itoa(num, ret, 4); // 第三个参数为目标进制
cout << ret << endl; // 123，将十进制数(27)_10 -> 四进制数(123)_4
~~~

#### is_sorted()

判断序列是否有序，`less_equal<>()` 和 `greater_equal<>()` 表示严格单调，`less<>()` 和 `greater<>()` 表示单调。

~~~c++
    vector<int> a = {1, 2, 2, 2, 3};
    vector<int> b = {1, 2, 3, 2, 3};

    vector<int> c = {3, 2, 2, 2, 1};
    vector<int> d = {1, 2, 2, 2, 1};
    cerr << is_sorted(a.begin(), a.end(), less<>()) << endl; // 1
    cerr << is_sorted(b.begin(), b.end()) << endl; // 0

    cerr << is_sorted(c.rbegin(), c.rend()) << endl; // 1
    cerr << is_sorted(d.rbegin(), d.rend()) << endl; // 0

    vector<int> e = {1, 2, 3, 4, 5}; //严格单调
    cerr << is_sorted(e.begin(), e.end(), less_equal<>()) << endl; //1

    vector<int> f = {5, 4, 3, 2, 1};
    cerr << is_sorted(f.begin(), f.end(), greater_equal<>()) << endl; // 1

    vector<int> g = {5, 5, 3, 3, 1};
    cerr << is_sorted(g.begin(), g.end(), greater<>()) << endl; // 1
    cerr << is_sorted(g.begin(), g.end(), greater_equal<>()) << endl; // 0
~~~

### 流

#### ostringstream

将格式化的数据写入字符串。

~~~c++
std::ostringstream os;
os << "赤红" << "'s age is " << 11 << "\n";
std::string s = os.str();
~~~

### 扩展 STL

#### rope

[例题](https://ac.nowcoder.com/acm/problem/17242)

来自 pb_ds 库 (Policy-Based Data Structures)。

类似封装好的块状链表，复杂度 $\log n$ 或者 $n^{0.5}$，适合用于大量、冗长的串操作。

基本操作：

~~~c++
#include <ext/rope>  // 头文件
using namespace __gnu_cxx;  // 注意名称空间

rope<int> rp;

int main() {
    rp.push_back(x); // 在末尾插入 x
    rp.insert(pos, x); // 在 pos 处插入 x
    rp.erase(pos, x); // 在 pos 处删除 x 个元素
    rp.length(); // 返回 rp 的大小
    rp.size(); // 同上
    rp.replace(pos, x); // 将 pos 处的元素替换成 x
    rp.substr(pos, x); // 从 pos 处开始提取 x 个元素
    rp.copy(pos, x, s); // 从 pos 处开始复制 x 个元素到 s 中
    rp[x]; // 访问第 x 个元素
    rp.at(x); // 同上
    return 0;
}

// Shuffle Cards 代码
void solve() {
    rope<int> rp;
    int n, m;
    cin >> n >> m;
    for (int i = 0; i < n; i++) {
        rp.push_back(i + 1);
    }
    for (int p, s; m--; ) {
        cin >> p >> s;
        rp = rp.substr(p - 1, s) + rp.substr(0, p - 1) + rp.substr(p + s - 1, n - s - (p - 1));
    }
    for (int i = 0; i < n; i++) {
        cout << rp[i] << " \n"[i == n - 1];
    }
}
~~~

### 报错

#### 不能使用某个类

~~~
auto tmp = array<int, 3> {1, 1, 2}; // 不允许使用不完整的类型
deque<std::array<int, 3>> q{ {0, -1, 0} }; // 没有与参数列表匹配的重载函数

unordered_set<int> st; // 未定义标识符
~~~

解决办法：include 该类的头文件。

## 数学

### 注意（WA的原因）

1. 模意义下出现减法时及时 + mod，遇到乘法就及时取 mod，以免后面要模多次或者爆 long long 造成 WA。
2. 取模之后算出的 gcd 就不是原先的 gcd。[如](https://ac.nowcoder.com/acm/contest/23479/J)
3. 直接 cout 浮点数一定要设置小数点位数。
4. 出现除法时注意除数是否可能为 0。

### 二进制拆分

一个数 $n$ 可以拆分为 $x$ 个数字，分别为：
$$
2^0,2^1,2^2,...,2^{k - 1},n-2^k+1, 其中k是满足n-2^k+1>0的最大整数
$$
满足使得这 $x$ 个数可以组合得到任意小于等于 $n$ 的正整数。

比如计算 $5$ 的二进制拆分，首先计算得到 $k$ 的取值，$k$ 是满足 $5 - 2^k + 1 > 0$ 的最大整数，得到 $k=2$，那么 $5$ 的二进制拆分得到的数字为 $2^0,2^1,5-2^2+1$，即 $1,2,2$，容易验证这三个数可以组合得到任意小于等于 $5$ 的正整数。

另外，$i$ 个 $2$ 的次幂刚好可以组合出所有二进制下 $1$ 的个数为 $i$  的数。

[p-binary](https://codeforces.com/problemset/problem/1247/C)就用到了二进制拆分。

化简下式子可以得到
$$
    n-i*p=\sum_{1}^{i} 2^k，k是任意自然数
$$

那么我们可以枚举 $i$，如果 $n - i * p$ 的二进制表示下$1$的数量等于 $i$，说明我们可以用 $i$ 个 $2$ 的幂次组合得到 $n - i * p$ 。注意枚举 $i$ 过程中 $i$ 的上限，等式右边的最小值，当且仅当 $k$ 全取 $0$，那么等式右边的最小值为 $i$，即如果 $n - i * p < i$，可以 break。

### 求和公式

平方和公式： $\sum_{k=1}^{n}k^2=1^2+2^2+3^2+...+n^2=\frac{n(n+1)(2n+1)}{6}$

立方和公式：$[\frac{(n \times (n + 1))}{2}]^2$

### 韦达定理

设一元二次方程 $ax^2+bx+c=0(a, b, c\in R, a \neq 0)$，两根 $x_1$、$x_2$ 有如下关系：
$$
x_1 + x_2 = - \frac{b}{a}
$$

$$
x_1 x_2 = \frac{c}{a}
$$

### 是否出现小数/分数

如出现小数/分数，将 double 转为 int 的过程中将会丢掉小数部分。

~~~c++
bool curF(double x) { //出现分数？
    return fabs(ll(x) - x) > eps;
}
~~~

### 同余定理

给定一个正整数 $m$，如果两个整数 $a$ 和 $b$ 满足 $a-b$ 能够被 $m$ 整除，即 $\frac {a-b}{m}$ 得到一个整数，那么就称整数 $a$ 与 $b$ 对模 $m$ 同余，记作 $a≡b$(mod m)。对模 $m$ 同余是整数的一个等价关系。

例题：[D - Integers Have Friends](https://codeforces.com/contest/1549/problem/D)。

### 组合数的各种性质和定理

[链接](https://blog.csdn.net/litble/article/details/75913032)

### 二项式定理

根据此定理，可以将 $x+y$ 的任意次幂展开成和的形式。
$$
(a+b)^{n}=\sum_{r=0}^{n} C_{n}^{r} a^{n-r} b^{r}=C_{n}^{0} a^{n}+C_{n}^{1} a^{n-1} b+\cdots+C_{n}^{r} a^{n-r} b^{r}+\cdots+C_{n}^{n} b^{n}
$$

### 整除符号

若整数 $b$ 除以非零整数 $a$，商为整数，且余数为零， 我们就说 $b$ 能被 $a$ 整除（或说 $a$ 能整除 $b$ ），$b$ 为被除数，$a$ 为除数，即 $a|b$（“|”是整除符号），读作“$a$ 整除 $b$”或“$b$ 能被 $a$ 整除”。$a$ 叫做 $b$ 的约数（或因数），$b$ 叫做 $a$ 的倍数。

例如 $2 | 4$，即 2 整除 4。

### 裴蜀定理

> $3x+2y=18$

如果 $a$、$b$ 是整数，那么一定存在整数 $x$、$y$ 使得 $ax+by=gcd(a,b)$。

换句话说，如果 $ax+by=m$ 有解，那么 $m$ 一定是 $gcd(a,b)$ 的若干倍，可以来判断一个这样的式子有没有解。运用扩展欧几里得算法可以求出满足该等式的一组 $x$，$y$。

一个直接的应用就是：如果 $ax+by=1$ 有解，那么 $gcd(a,b)=1$。

可以推广到 $n$ 个整数间的裴蜀定理：

设 $a_1,a_2,...,a_n$为 $n$ 个整数，$d$ 是它们的最大公约数，那么存在整数$x_1,x_2,...,x_n$ 使得 $x_1*a_1+x_2*a_2+...x_n*a_n=d$。

### 整数唯一分解定理

任何一个大于 1 的整数 $n$ 都可以分解成若干个素因数的连乘积，如果不计各个素因数的顺序，那么这种分解是唯一的。

### 约数个数定理 / 约数和定理

对于一个大于 1 正整数 $n$ 可以分解质因数：
$$
n=\prod_{i=1}^{k} p_{i}^{a_{i}}=p_{1}^{a_{1}} \cdot p_{2}^{a_{2}} \cdots p_{k}^{a_{k}}
$$
则 $n$ 的正约数的个数就是：
$$
f(n)=\prod_{i=1}^{k}\left(a_{i}+1\right)=\left(a_{1}+1\right)\left(a_{2}+1\right) \cdots\left(a_{k}+1\right)
$$
其中 $a_1, a_2, a_3, \cdots , a_k$ 是 $p_1, p_2, p_3, \cdots , p_k$ 的指数。

$n$ 的 $f(n)$ 个正约数的和为：
$$
f(n)=(p_1^0+p_1^1+p_1^2+…p_1^{a_1})(p_2^0+p_2^1+p_2^2+…p_2^{a_2}) \cdots (p_k^0+p_k^1+p_k^2+…p_k^{a_k}）
$$

## 计算几何

### 利用球面一般方程的系数计算球心坐标和半径

写出一般方程：$x^{2}+y^{2}+z^{2}+2ax+2by+2cz+d=0$

化作标准方程：$(x+a)^{2}+(y+b)^{2}+(z+c)^{2}=a^{2}+b^{2}+c^{2}-d$

易得球心坐标$O(-a,-b,-c)$，半径 $r=\sqrt{a^{2}+b^{2}+c^{2}-d}$

在球面的一般方程中，如果将等号改为小于号，则表示球内部所有的点。

### 球缺

一个球被平面截下的一部分叫做球缺。截面叫做球缺的底面，垂直于截面的直径被截后，剩下的线段长叫做球缺的高。球缺曲面部分的面积（球冠面积）$S=2πRH$，球缺体积公式 $V = \pi H^2(R- \frac{H}{3})$（ $R$ 是球的半径, $H$ 是球缺的高)。

## 图论

### 注意

1. 使用邻接矩阵存图时，如题目无特殊说明，记得考虑重边、自环的情况
2. 注意题目给出的图是不是联通的，[比如](https://atcoder.jp/contests/abc231/tasks/abc231_d)判环时若图不连通则需要对 $n$ 个点分别进行 1 次 dfs。

### 打印路径的两种方法

1. $pre[v] = u$，表示 $v$ 的前驱为 $u$，在更新了最小值之后紧接着更新 $pre$ 数组。在打印结果时，从终点一路循环遍历到源点，将路过的节点依次储存在双端队列中。最后顺序遍历双端队列即可得到路径。

   ~~~c++
   ll tmp = end;
   while (pre[tmp] != -1) {
       road.push_front({pre[tmp], tmp});
       tmp = pre[tmp];
   }
   ~~~

   

2. $path[i][j] = k$，表示 $i$ 到 $j$ 的路径为 $i$ 先到 $k$，再从 $k$ 到 $j$。在打印结果时也可以通过 while 循环实现。初始化时 $path[i][i] = i$，在每次松弛操作成功后更新 $path$ 数组。

   ~~~c++
   printf("Path: %d", x);
   while (x != y) {
       printf("-->%d", path[x][y]);
       x = path[x][y];
   }
   ~~~

### 根节点的位置

在一棵树中，除根节点外，其余每个节点都有父节点（可以用来判断根节点的位置）

### 网络流

点的限制考虑拆点 https://vjudge.net/solution/32496101。

### 最短路

#### 用 Bellman Ford 求解有限制的最短路问题

「限制最多经过不超过 kk 个点」等价于「限制最多不超过 k + 1 条边」，而解决「有边数限制的最短路问题」是 SPFA 所不能取代 Bellman Ford 算法的经典应用之一（SPFA 能做，但不能直接做）。

Bellman Ford/SPFA 都是基于动态规划，其原始的状态定义为 $f[i][k]$ 代表从起点到 $i$ 点，且经过最多 $k$ 条边的最短路径。这样的状态定义引导我们能够使用 Bellman Ford 来解决有边数限制的最短路问题。

同样多源汇最短路算法 Floyd 也是基于动态规划，其原始的三维状态定义为 $f[i][j][k]$ 代表从点 $i$ 到点 $j$，且经过的所有点编号不会超过 $k$（即可使用点编号范围为 $[1, k]$）的最短路径。这样的状态定义引导我们能够使用 Floyd 求最小环或者求“重心点”（即删除该点后，最短路值会变大）。

[787. K 站中转内最便宜的航班 - 力扣（LeetCode）](https://leetcode-cn.com/problems/cheapest-flights-within-k-stops/)

### 找环/判环

[Forest Program](https://codeforces.com/gym/102361/problem/F)

~~~c++
/* 给定一张无向简单图，同时规定一条边只属于一个环。可以删除任意条边使得这张图变成森林，也就是使得每一个连通块都是树。求一共有多少种方案。 */

vector<int> g[N];
int dfn[N];
ll num = 0, ans = 1;
bool vis[N];

void dfs(int idx, int u, int fa) {
    dfn[u] = idx;
    for (auto v : g[u]) {
        if (v == fa)
            continue;
        if (!dfn[v])
            dfs(idx + 1, v, u);
        else if (dfn[v] < dfn[u]) {
            ll tmp = dfn[u] - dfn[v] + 1; //tmp为当前环上边的数量
            num += tmp;
            ans = (ans * (powmod(2, tmp) - 1) + mod) % mod;
        }
    }
    vis[u] = true;
}

int main() {
    ios::sync_with_stdio(false); cin.tie(0); cout.tie(0);
    int n, m; cin >> n >> m;
    for (int i = 0, u, v; i < m; i++) {
        cin >> u >> v;
        g[u].emplace_back(v);
        g[v].emplace_back(u);
    }
    for (int i = 1; i <= n; i++) { //图可能不连通
        if (!vis[i])
            dfs(1, i, -1);
    }
    cout << ans * powmod(2, m - num) % mod << "\n";
    return 0;
}
~~~

### 欧拉通路/回路

#### 定义

如果图 G（有向图或者无向图）中所有边一次仅且一次行遍所有顶点的通路称作欧拉通路。

如果图 G 中所有边一次仅且一次行遍所有顶点的回路称作欧拉回路。

具有欧拉回路的图称为欧拉图（简称E图）。具有欧拉通路但不具有欧拉回路的图称为半欧拉图。

#### 定理及推论

欧拉通路和欧拉回路的判定是很简单的，请看下面的定理及推论。

##### 无向图 G 存在欧拉通路的充要条件

G 为连通图，并且 G 仅有两个奇度结点（度数为奇数的顶点）或者无奇度结点。

推论1：

- 当 G 是仅有两个奇度结点的连通图时，G 的欧拉通路必以此两个结点为端点。
- 当 G 是无奇度结点的连通图时，G 必有欧拉回路。
- G 为欧拉图（存在欧拉回路）的充分必要条件是 G 为无奇度结点的连通图。

##### 有向图 D 存在欧拉通路的充要条件

D 为有向图，D 的基图连通，并且所有顶点的出度与入度都相等；或者除两个顶点外，其余顶点的出度与入度都相等，而这两个顶点中一个顶点的出度与入度之差为 1，另一个顶点的出度与入度之差为 -1。

推论2：

- 当 D 除出、入度之差为 1，-1 的两个顶点之外，其余顶点的出度与入度都相等时，D 的有向欧拉通路必以出、入度之差为 1 的顶点作为始点，以出、入度之差为 -1 的顶点作为终点。
- 当 D 的所有顶点的出、入度都相等时，D 中存在有向欧拉回路。
- 有向图 D 为有向欧拉图的充分必要条件是 D 的基图为连通图，并且所有顶点的出、入度都相等。

#### 欧拉通路回路存在的判断

根据定理和推论，我们可以很好的找到欧拉通路回路的判断方法，定理和推论是来自离散数学的内容，这里就给出简明的判断方法：

##### 判断欧拉通路是否存在的方法

有向图：图连通，有一个顶点出度大入度 1，有一个顶点入度大出度 1，其余都是出度=入度。

无向图：图连通，只有两个顶点是奇数度，其余都是偶数度的。

##### 判断欧拉回路是否存在的方法

有向图：图连通，所有的顶点出度=入度。

无向图：图连通，所有顶点都是偶数度。

### 拓扑排序

一直删入度为 0 的点，判断是否能删除所有的点。

将 DAG 上的问题转化为线性序列上的问题。

### 双连通分量

在一张连通的无向图中，对于两个点 $\mathcal u$ 和 $\mathcal v$，如果无论删去哪条边（只能删去一条）都不能使它们不连通，我们就说 $\mathcal u$ 和 $\mathcal v$ **边双连通**。

在一张连通的无向图中，对于两个点 $\mathcal u$ 和 $\mathcal v$，如果无论删去哪个点（只能删去一个，且不能删 $\mathcal u$ 和 $\mathcal v$ 自己）都不能使它们不连通，我们就说 $\mathcal u$ 和 $\mathcal v$ **点双连通**。

边双连通具有传递性，点双连通不具有传递性。

## 数据结构

### 线段树

#### 维护前缀和的最小值

[例题](https://atcoder.jp/contests/abc223/submissions/29656001)

合并时：

~~~c++
res.sum = sum + node.sum;
res.minn = min(minn, sum + node.minn);
~~~

比较**左子树的前缀和最小值** 与 **左子树的区间和 + 右子树的前缀和最小值**，用较小值更新父节点的前缀和最小值。

## 搜索

### 注意

1. dfs 的中间变量，尤其是还要用于回溯的，老老实实定义成局部变量。比如在[天元突破 红莲螺岩](https://ac.nowcoder.com/acm/contest/16976/F)一题中，如果将 dfs 中临时记录的变量 tmp 定义成全局变量，那么在回溯的时候就会[出错](https://ac.nowcoder.com/acm/contest/view-submission?submissionId=47957932)。
2. 在进行 bfs 时注意已经走过的点不可能再更新它的最小距离。
3. bfs 时可以通过改变枚举方向的顺序使得在找到最短路径的情况下方向序列的字典序最小。如 [Penguins](https://ac.nowcoder.com/acm/contest/view-submission?submissionId=48251717)。

~~~
/*(D<L<R<U)
     U
   L @ R
     D
     D
     D
 @LLLD
 
*/
~~~

4. 留意 dfs 的返回结果是否可以直接 return，还是放在 if 中作为条件判断。如[网格路径 - HDU 7037](https://vjudge.net/problem/HDU-7037)这题中， 如果直接 return dfs 的结果，可能由于找不到路径直接返回 false 了，实际上可以通过下面（下一行）的 dfs 可以找到一条路径，而由于在这一层递归中已经 return false 而造成答案错误。（就像 dfs 的匈牙利算法那样）

### 记忆化搜索

[天元突破 红莲螺岩](https://ac.nowcoder.com/acm/contest/16976/F)，$dp[pos][lf][rt]$ 表示在 $pos$ 位置，左边剩 $lf$ 个，右边剩 $rt$ 个时的最小花费。[code](https://ac.nowcoder.com/acm/contest/view-submission?submissionId=47958190)

## dp

### 状态转移

状态转移时，依赖关系不好找，而被依赖的关系容易找到，所以对于每个状态，更新被这个状态依赖的关系。

也就是说，对于逆拓扑计算很难划分，可以采用正拓扑的计算方法，从第 $i-1$ 个物品的角度看会递推到那两个集合中

[例](https://ac.nowcoder.com/acm/contest/23106#submit/%22onlyMyStatusFilter%22%3Atrue%2C%22problemIdFilter%22%3A231939)

~~~c++
for (int i = 1; i <= n; i++) {
    dp[i][f(a[i])] = 1;
    for (int j = 1; j <= 9; j++) {
        dp[i][f(a[i] + j)] += dp[i - 1][j];
        dp[i][j] += dp[i - 1][j];
    }
}
~~~

## 博弈论

### 必胜态与必败态的转化

如果一个状态 A 在一次操作后变成了必败态，则这个状态 A 是必胜态。（巴不得对手死，有一种死法就行）

如果一个状态 B 在一次操作后都是必胜态，则这个状态 B 是必败态。（垂死挣扎，但只能死）

## 杂项

### 关于签到那些事

#### 签到的思路
1. 首先观察数据范围，确定是不是可以直接暴力/模拟/爆搜。
2. 如果没有很好的思路，手玩一下样例/打表看看是否存在规律。
3. 连续区间，考虑双指针、前缀和、差分。
4. 求最优解问题，先寻找局部最优解，确定使用贪心或 dp。
5. 遇到构造题，先确定最终的目标是什么，慢慢往目标上靠。（比如 [Matrix Problem](https://ac.nowcoder.com/acm/contest/16092/M)，需要让构造出来的矩阵具有很强的连通性，想到像手一样的模型）
6. 逆向思维有时会大大降低实现难度。
7. 考虑抽屉原理可以起到剪枝的效果。（比如 [Shortest Cycle](https://codeforces.com/contest/1206/problem/D)，抽屉原理使得n的规模缩减，从而可以使用 floyd 算法）
8. 模拟样例的时候尽量写出表达式，以便看出规律。按照模拟出来的表达式写代码，变量名不要误写成常数。（如[C - Sweets Eating](https://codeforces.com/contest/1253/problem/C)）
9. 有显然的递推式时可以考虑记忆化搜索或剪枝。（如[HDU - 6983](https://vjudge.net/problem/HDU-6983/origin)）
10. 关于异或的题牢牢抓住 $ a \oplus a = 0$ 这个性质，考虑转换和抵消。
11. Σ式子化简时，可以展开之后再找规律。（[如](https://blog.csdn.net/qq_51354600/article/details/119638157)

#### 这为什么会WA呢？
1. 特判。（n=0, n=1?)
2. 答案是否是非负的，~~有可能在特判的时候神志不清~~。（比如[Chess Cheater](https://codeforces.com/contest/1427/submission/118950139)，在特判全为L的情况时使答案为 -1，那么答案是不可能为负的，及时修改特判）
3. 初始化（省赛
4. 数组开小了（省赛
5. 爆 int（做前缀和的时候开 long long
6. 是否多组样例
7. 划分区间时特别注意最后一块（可能没有处理完，如[Silly Mistake](https://codeforces.com/contest/1253/problem/B))
8. 注意在更新答案的时候是不是在 if 中才更新，如果在 if 中是不是能够保证结果能够得到更新。（常见于 dp 中
9. 运行后直接死机注意是不是内存爆了的问题，比如关于 vector 的 emplace_back 函数死循环了。
10. 多组输入且需要 memset 的时候数组大小开的精确一些，不然有可能会 TLE。
11. 模拟加法减法等的时候注意处理进位或借位（可以先reverse，最后再reverse回来）。
12. 碰到很烦人的边界问题时（加 1 还是 -1 还是不加），WA 了之后一定要造一些数据验证边界的处理。

### 矩阵表示中一维和二维表示的关系

#### 0-index

~~~
   0  1  2  3
  __________
0| 0  1  2  3
1| 4  5  6  7
2| 8  9  10 11
3| 12 13 14 15
~~~

设一维坐标为 pos，二维坐标为 (x, y)，矩阵大小为 n * m。

二维转一维：pos = x * m + y

一维转二维：x = pos / m, y = pos % m

#### 1-index

~~~
   1  2  3  4
  __________
1| 1  2  3  4
2| 5  6  7  8
3| 9  10 11 12
4| 13 14 15 16
~~~

设一维坐标为 pos，二维坐标为 (x, y)，矩阵大小为 n * m。

二维转一维：pos = (x - 1) * m + y

一维转二维：x = pos + (m - 1) / m, y = pos % m == 0 ? m : pos % m

### 二维前缀和/差分

[二维差分 - 新之守护者](https://www.cnblogs.com/LMCC1108/p/10753451.html)

二维前缀和
$$
sum[i][j]=a[i][j]+sum[i-1][j]+sum[i][j-1]-sum[i-1][j-1]
$$
二维差分

求左上角是 $(x1,y1)$，右下角是 $(x2,y2)$ 的矩形区间内的值：
$$
sum[x2][y2]-sum[x2][y1-1]-sum[x1-1][y2]+sum[x1-1][x1-1]
$$

### 单调队列

~~~c++
//求滑动窗口的最大值
ll h = 0, t = -1;
for (ll i = 1; i <= n; i++) { //为了方便入队出队，单调队列中存储的是元素下标
    //如果队头q[h]已经不属于窗口[i-m+1, i]以内，则队头出队
    if (h <= t && q[h] < i - m + 1)
        h++;
    //如果待插入元素>=当前队尾，队尾出队（比你小还比你强
    while (h <= t && a[i] >= a[q[t]]) //如果求滑动窗口的最小值，则把>=改位<=
        t--;
    q[++t] = i;
    if (i > m - 1)
        cout << a[q[h]] << " ";
}
cout << "\n";
~~~

### 尺取法

给定长度为 $n$ 的数列整数 $a_0,a_1,…,a_{n-1}$以及整数 $S$。求出总和不小于 $S$ 的连续子序列的长度的最小值。如果解不存在，则输出 0。

~~~c++
ll n, S, a[N];

void solve() {
    ll res = n + 1;
    ll s = 0, t = 0, sum = 0;
    for (;;) {
        while (t < n && sum < S) { //移动下标的循环
            sum += a[t++];
        }
        if (sum < S) //退出尺取法搜索的条件
            break;
        res = min(res, t - s); //更新答案
        sum -= a[s++]; //更新左边界
    }
    if (res > n) { //解不存在
        res = 0;
    }
}
~~~

类似的题目：

[小幼稚买蛋糕](https://ac.nowcoder.com/acm/contest/16976/I)

~~~c++
ll n, x, ans, sum, v[N], pre[N];
 
int main() {
    ios::sync_with_stdio(false); cin.tie(0); cout.tie(0);
    cin >> n >> x;
    for (ll i = 0; i < n; i++) {
        cin >> v[i];
        if (i == 0)
            pre[i] = v[i];
        else
            pre[i] = pre[i - 1] + v[i];
    }
    ll lf = 0, rt = 0;
    for (;;) {
        while (rt < n && sum + v[rt] <= x) {
            sum += v[rt++];
            ans = max(ans, pre[rt - 1] - (lf - 1 < 0 ? 0 : pre[lf - 1]));
        }
        if (lf >= rt)
            break;
        sum -= v[lf++];
    }
    cout << ans << "\n";
    return 0;
}
~~~

### 随机数mt19937

周期长度：$2^{19937}-1$

产生的随机数的范围在 int 类型范围内。

也可以限制随机数的范围在$[lf, rt]$区间内。

~~~c++
template <class T>
T randint(T l, T r = 0) // 生成随机数建议用<random>里的引擎和分布，而不是rand()模数，那样保证是均匀分布
{
    static mt19937 eng(time(0));
    if (l > r)
        swap(l, r);
    uniform_int_distribution<T> dis(l, r);
    return dis(eng);
}

//var = randint(1, 1000000);
//var = radint(1000000);
~~~

### function类模板

~~~c++
vector<vector<int>> allPathsSourceTarget(vector<vector<int>>& graph) {
    vector<vector<int>> ans;
    function<void(int, vector<int>, bitset<20>)> dfs = [&](int x, vector<int> now, bitset<20> vis) {
    // auto dfs = [&](int x, vector<int> now, bitset<20> vis) { //使用 auto 类型说明符声明的变量不能出现在其自身的初始值设定项中
        now.emplace_back(x);
        vis.set(x);
        if (x == int(graph.size()) - 1) {
            ans.emplace_back(now);
            return;
        }
        for (int i = 0; i < graph[x].size(); i++) {
            int v = graph[x][i];
            if (vis[v])
                continue;
            dfs(v, now, vis);
            vis.reset(x);
        }
        return;
    };
    bitset<20> bs;
    dfs(0, {}, bs);
    return ans;
}
~~~

### 本地测试运行时间

在 main() 开始时给 startTime 赋值：`startTime = clock();`

~~~c++
clock_t startTime;
double getCurrentTime() {
    return (double)(clock() - startTime) / CLOCKS_PER_SEC;
}
~~~