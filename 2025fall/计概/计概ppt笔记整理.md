

## 几种结构式类型
## 枚举
• typedef enum{枚举常量名1, ...} 枚举类型名; 
• typefdef enum {WHITE, BLACK} COLOR1;
• COLOR1 cr3,cr4;

## 结构体
```cpp
struct square { 
	struct point { 
		int x, y; 
	} p1, p2; 
} sq1;
```
结构体变量之间进行赋值时，系统将按成员一一对应赋值
对分量的访问一般通过“变量名.分量名”完成
分量的类型不能是正在定义的结构体类型

### 结构体直接init():
```cpp
struct student { 
	int num; 
	char name[20]; 
	char sex; 
} stu[3]={{89031, “Li Lin”, ‘M’}, 
		{89032, “Liu Ying”, ‘M’}, 
		{89036, “Wang Min”, ‘F’}
		};
```

一个指向结构体变量的指针就是该变量占据的内存段的起始地址。可以设一个指针变量，用来指向一个结构类型变量。

• 以下三种形式等价：
	①结构类型变量名.成员名 
	②(\*p).成员名 
	③P->成员名

### typedef
• 可以用typedef声明新的类型名来代替已有的类型名
• typedef int INTEGER; 
- int i,j; 与 INTERGER i,j; 等价
• typedef int DataType ; 
• DateType \*element ; 与int \*element; 等价

### 结构体支持构造类似__init()__函数初始化

```cpp
struct Person {
    string name;
    int age;

    // 无参构造
    Person() : name("未知"), age(0) {}

    // 仅传姓名（年龄默认0）
    Person(string n) : name(n), age(0) {}

    // 传姓名+年龄
    Person(string n, int a) : name(n), age(a < 0 ? 0 : a) {}
};

int main() {
    Person p1;          // 无参：name=未知，age=0
    Person p2("李四");  // 单参：name=李四，age=0
    Person p3("王五", 25); // 双参：name=王五，age=25
    return 0;
}
```

## 递归

## 辗转相除法的递归
```cpp
int gcd(int a, int b){
	int c;
	while(b){
		c = a%b;
		a = b;
		b = c;
	}
	return a;
}

int gcd_plus(int a, int b){
	if(!b) return a;
	return gcd_plus(b, a%b);
}

int lcm(int a, int b){
	return a*b / gcd(a, b);
}
```

### 全排列的递归实现
```cpp
void AllRange(char *pszStr, int k, int m) { 
	if (k == m) { 
		static int s_i = 1; 
		printf("第%3d个排列\t%s\n", s_i++, pszStr); 
		} 
		else { 
		for (int i = k; i <= m; i++) {
		//第i个字母（用的下标表示）分别与它后面的字母交换就能得到新的排列 
			Swap(pszStr + k, pszStr + i); 
			AllRange(pszStr, k + 1, m); 
			Swap(pszStr + k, pszStr + i); 
		} 
	} 
}
```

### 汉诺塔
```cpp
int move(int n, char a, char b, char c){
	static int cnt = 0;
	if(n == 1){
		printf("%d : %c -> %c\n", ++cnt, a, c);
		return cnt;
	}
	move(n-1, a, c, b);
	printf("%d : %c -> %c\n", ++cnt, a, c);
	move(n-1, b, a, c);
	return cnt;
}
```

## 贪心法
## 1. 烙饼问题的拓展——电池配对使用问题
### 问题描述：
电池只能两个一起使用，而每个电池的寿命不一致，输入n个电池分别的寿命，然后给出利用最充分的方案，以及最多能使用多长时间

### 思路：
我们先讨论一个小规模的问题
- 如果是两个电池，那么只能用完那个寿命较短的
- 如果是三个电池，我们假设寿命分别为$a, b, c\text{ and }a<b<c$ 考虑烙饼问题的方法，b, c, 先一起用k，然后c-k刚好可以与a和b-k一起用完$$c-k = a + b-k$$$$k = \frac{a+b-c}{2} > 0$$即满足三角形三边关系即可
- 如果有更多的电池呢？如果是整数，那么很好的思路是取出最小分度值的一半`0.5` 这样只需要维护一个堆，对堆顶的两个元素同时减去`0.5`,一直进行这样的操作，显然可以利用所有的能利用的电池寿命
	在这种情况下，什么时候用不完所有的电量呢？
	不难想到，如果有一个特别大，达到超过了其他所有之和，那么他和其他所有的搭配完依然会剩下$$\omega = x_n - \Sigma^{n-1}_{i=1}x_i$$这里的$\omega$的电量是无法使用的，则$$T_{max} = \frac{\Sigma_{i=1}^{n}x_i - \omega}{2} = \frac{1}{2}x_n + \Sigma_{i=1}^{n-1}x_i$$
- 如果电池的寿命都是实数呢？之前的构造方案好像行不通了！
	但是我们还是可以想要把n个电池的问题化归成n-1规模的问题
	于是乎我们猜想如果一直用最大的电池减去最小的电池，每次操作之后都能继续满足我们设置的条件`最大的电池容量小于其他之和`，这样就一定能回到n=3的情况，问题解决
- 我们现在证明这个构造是正确的：
	假设$x_1, x_2, ..., x_n$是已经排好序的电池，且满足$$x_n < \Sigma^{n-1}_{i=1}x_i$$
	那么现在同时使用$x_1$和$x_n$： $x_1$耗尽，$x_n$变成$x_n - x_1$
	然后重新排序
	case 1. $x_n-x_1$依然最大
		此时n-1个电池排序为$x_2, ..., x_{n-1}, x_n-x_1$， 于是$$(x_n-x_1) - \Sigma^{n-1}_{i=2}x_i = x_n - \Sigma^{n-1}_{i=1}x_i<0$$则依然满足$$(x_n-x_1) < \Sigma^{n-1}_{i=2}x_i$$
	case 2. $x_{n-1}$最大
		此时n-1个电池排序为$x_2, ...,x_n-x_1, ..., x_{n-1}$， 于是$$x_{n-1} - (x_n-x_1) - \Sigma^{n-2}_{i=2}x_i < x_{n-1} - x_n + x_1 - x_2 < 0$$
		则依然满足$$x_{n-1} < (x_n - x_1)+ \Sigma^{n-2}_{i=2}x_i$$
	综上所述可知，所有满足我们的前置条件的电池组都能化归到n=3的情况然后用完，
	而不满足的都用不完（这个之前已经讨论过了）证毕
### 结果
$$\text{Rank all as }x_1, x_2, ..., x_n \text{ Then, }$$$$ T_{\max} = \begin{cases} \frac{1}{2}x_n + \sum_{i=1}^{n-1}x_i & \text{if } x_n \le \sum_{i=1}^{n-1}x_i \\ \frac{1}{2}\sum_{i=1}^n x_i & \text{if } x_n > \sum_{i=1}^{n-1}x_i \end{cases} $$
## 2. 田忌赛马问题
## 活动兼容问题
### 证明过程
设定：
- $S = {1, 2, ..., n}$ 是所有活动的集合，已经按 结束时间 从小到大排好序了（即活动1是结束最早的）。
- 贪心解 $G$ ：这是我们算法选出来的方案。它的第一个活动肯定是 活动1 （因为结束最早）。
  - 记 $G = {g_1, g_2, ..., g_k}$，其中 $g_1 = 1$。
- 最优解 $O$ ：假设存在一个最优方案（活动数量最多）。
  - 记 $O = {o_1, o_2, ..., o_m}$，其中 $o_1$ 是最优解里时间最早的那个活动。
**关键步骤：替换**
1. 比较第一个活动：   
   - 贪心解选的是 $g_1$（即活动1，全场结束最早）。
   - 最优解选的是 $o_1$。
   - 显然，因为 $g_1$ 是全场结束最早的，所以 $g_1$ 的结束时间 $\le o_1$ 的结束时间。即 $finish(g_1) \le finish(o_1)$。
2. 执行替换：
   - 如果我们把最优解 $O$ 中的第一个活动 $o_1$ 换成 $g_1$，会发生什么？
   - 新方案 $O' = {g_1, o_2, o_3, ..., o_m}$。
   - 合法性检查 ：
     - 原来的 $o_2$ 是接在 $o_1$ 后面的，说明 $start(o_2) \ge finish(o_1)$。
     - 因为 $finish(g_1) \le finish(o_1)$（贪心选的结束得更早），所以 $start(o_2)$ 肯定也大于 $finish(g_1)$。
     - 结论 ：$g_1$ 不会和后面的 $o_2...o_m$ 冲突。新方案 $O'$ 依然是合法的！
   - 数量检查 ：
     - $O'$ 的活动数量和 $O$ 一样，都是 $m$ 个。
     - 所以，$O'$ 也是一个最优解。
1. 归纳推导：
   - 现在我们证明了： 一定存在一个包含活动1的最优解。
   - 接下来，问题就变成了：在选了活动1之后，剩下的时间里，怎么选出最多的活动？
   - 这其实就是一个 规模变小的子问题 。
   - 在这个子问题里，我们依然用贪心策略选结束最早的...
   - 通过数学归纳法，每一步我们都可以把最优解里的活动替换成贪心策略选的活动，且总数不减少。
**最终结论： 贪心解的活动数量 $k$ 一定等于最优解的活动数量 $m$。贪心策略是正确的。

## DP
### 最大路径
### LIS
### LCS

## 重复读入

```cpp
//1.scanf %s会自动忽略前面的'\n'' ''tab', 遇到空格或者换行则停下，会遗留换行符'\n'
while(scanf("%s%s", s1, s2) != EOF){}

//2.gets()，遇到空格不停下，遇到换行才停下, 读入结束之后会自动清理掉行末的\n
//但是必须要小心前面残余的\n！这会导致空白读入！
while(gets(a) != NULL){}()//尽量不用，c++14已经删除

//3.getline 这是gets的平替，行为几乎一致（除了返回bool值）
while(getline(cin, s)){}
//配合stringstream有妙用
string line; 
stringstream ss;
while (getline(cin, line)) { 
	ss.clear(); // 1. 重置流的状态标志（如 EOF 标志） 
	ss.str(line);// 2. 将新的一行内容放入流中 
	string s, int num; 
	while (ss >> string >> num) { 
		// 处理数据 
	} 
}
//局部变量更加稳妥
while (getline(cin, line)) { 
	stringstream ss(line); // 每次都是新的，不需要 clear() 
	int num; 
	while (ss >> num) { 
		// ... 
	} 
}

//4.scanf重复读入特定格式
while(scanf("%d %d,", &a, &b) == 2){}
if(scanf("%d %d", &a, &b) == 2){}
```

## Scanf的读入原则
### 核心定义
==**scanf 返回的是 成功匹配并赋值的参数个数 。**== 

### 情况一：完全成功 (Positive Integer)
返回值 > 0

这是最正常的情况。返回值等于你格式串里 % 的个数（不包括 %*d 这种忽略项）。

- scanf("%d", &a)
  - 输入 10 -> 返回 1
- scanf("%d %d", &a, &b)
  - 输入 10 20 -> 返回 2
- scanf("%d,%d", &a, &b)
  - 输入 10,20 -> 返回 2
### 情况二：匹配失败 / 部分成功 (0 或 小于预期)
返回值 >= 0，但小于你期望的数量

这表示输入流里的数据格式不对， scanf 读到一半读不下去了。

- scanf("%d %d", &a, &b)
  - 输入 a 20 （第一个就是字母）
    - scanf 试图读 %d ，看到 a ，匹配失败。
    - 返回值 0 。
    - 后果 ： a 还是缓冲区里的第一个字符，下一次读还是它（死循环预警）。
  - 输入 10 a （第一个对，第二个错）
    - 读 10 成功（ a 被赋值）。
    - 读 %d 时遇到 a ，失败。
    - 返回值 1 。
    - 后果 ：变量 a 有值，变量 b 没值（保持原样）。
### 情况三：文件结束 / 读取错误 (EOF)
返回值 == EOF (通常是 -1)

这表示在 还没读到任何有效数据之前 ，就已经遇到了文件结尾或者发生了硬件错误。

- scanf("%d", &a)
  - 输入流已空（或者用户按了 Ctrl+Z/Ctrl+D）。
  - 返回值 -1 。
- 注意区别 ：如果文件里只剩一个换行符 \n ， scanf 会跳过它去尝试读后面，如果跳过换行符后发现没东西了，也会返回 EOF。
![[Pasted image 20251208220034.png]]

## 输入输出格式
### 1. 基础类型对照表

| **类型**          | **scanf 读入**    | **printf 输出**      | **备注**                           |                      |
| --------------- | --------------- | ------------------ | -------------------------------- | -------------------- |
| **int**         | `%d`            | `%d`               | 常用                               |                      |
| **long long**   | `%lld`          | `%lld`             | **重要**：大数字必用                     |                      |
| **float**       | `%f`            | `%f`               | 默认 6 位小数                         |                      |
| **double**      | **`%lf`**       | `%f`               | 读入 double 必须用 `%lf`              |                      |
| **char**        | `%c`            | `%c`               | 会读入空格和换行                         |                      |
| **char[]**      | `%s`            | `%s`               | 遇到空格或换行停止                        |                      |
| **符号**          | **类型/含义**       | **scanf 输入示例**     | **printf 输出示例**                  | **备注**               |
| **`%d`**        | 有符号十进制 int      | `scanf("%d", &n);` | `printf("%d", 10);` // 10        | 最常用                  |
| **`%o`**        | **八进制** (octal) | `scanf("%o", &n);` | `printf("%o", 10);` // 12        | 不带前缀 `0`             |
| **`%x` / `%X`** | **十六进制** (hex)  | `scanf("%x", &n);` | `printf("%x", 255);` // ff       | `X` 输出大写 `FF`        |
| **`%u`**        | 无符号十进制          | `scanf("%u", &n);` | `printf("%u", n);`               | 处理正数范围更大             |
| **`%e` / `%E`** | **科学计数法**       | `scanf("%e", &f);` | `printf("%e", 0.01);` // 1.0e-02 | 用于极小或极大浮点数           |
| **`%g` / `%G`** | 自动选择            | (不常用)              | `printf("%g", 0.00001);`         | 自动在 `%f` 和 `%e` 中选短的 |
| **`%p`**        | 指针地址            | (不常用)              | `printf("%p", &a);`              | 调试时查看内存地址            |
|**`%%`**|字符 `%` 本身|`scanf("%%");`|`printf("10%%");` // 10%|在格式串中表示百分号|

---
### 2. `printf` 进阶：控制输出格式
这是模拟题和日期题的**高频考点**：
- **补齐位数 (`%0nd`)**：
    - `printf("%02d", 5);` $\rightarrow$ 输出 `05`   
    - `printf("%04d", 2024);` $\rightarrow$ 输出 `2024`
    - _用途：处理日期 YYYY-MM-DD。_
- **控制小数位数 (`%.nf`)**：
    - `printf("%.2f", 3.1415);` $\rightarrow$ 输出 `3.14`（会自动四舍五入）。
- **左对齐与右对齐**：
    - `printf("%5d", x);` $\rightarrow$ 右对齐，宽度为 5，不足补空格。
    - `printf("%-5d", x);` $\rightarrow$ 左对齐，宽度为 5。

---
### 3. `scanf` 进阶：精准提取输入
`scanf` 最强大的地方在于它可以**跳过无关字符**或**限制长度**。
- **跳过固定字符**：
    - `scanf("%d-%d-%d", &y, &m, &d);`
    - _输入 `2024-12-19` 时，会自动过滤掉 `-`。_
- **忽略某些输入 (`%*`)**：
    - `scanf("%d %*s %d", &a, &b);`
    - _输入 `100 score 95` 时，会把 `100` 给 `a`，跳过 `score`，把 `95` 给 `b`。_
- **限制读入长度 (`%nd`)**：
    - `scanf("%4d%2d%2d", &y, &m, &d);`
    - _输入 `20241219` 时，会自动切分为 `2024`, `12`, `19`。_
- **读入一行到字符数组 (不含换行)**：
    - `scanf("%[^\n]", s);`
    - _这会读取直到遇到 `\n` 为止的所有字符。_

---
### 4. 考场避坑指南

#### ① `%c` 的空格问题
`scanf(" %c", &c);` （注意 `%c` 前面有一个**空格**）。
- **作用**：这个空格会告诉 `scanf` 自动跳过前面所有的空白符（空格、回车、制表符），直到读到一个真正的字符。在循环读入字符时这是**救命技巧**。
#### ② `scanf` 的返回值
`while (scanf("%d", &n) != EOF)`
- 在文件输入环境下，当读到文件末尾时，`scanf` 返回 `EOF`（通常是 -1）。
#### ③ `double` 的读写差异
- **读入**：必须用 `%lf`。
- **输出**：用 `%f` 或 `%lf` 均可（建议统一用 `%f` 或根据题目要求）。


### 进制转换

如果在计概B考试中遇到需要**不同进制转换**（例如把一个 16 进制字符串转为 10 进制），除了 `scanf("%x")`，你还可以利用 C++ 的 `strtol` 函数（在 `<cstdlib>` 中）：
```cpp
string s = "FF";
// 将字符串 s 按 16 进制转为 long
long val = strtol(s.c_str(), NULL, 16); 
```

### 几个库函数

- **`lower_bound(begin, end, val)`**：返回第一个 **大于或等于** `val` 的元素的迭代器。
- **`upper_bound(begin, end, val)`**：返回第一个 **严格大于** `val` 的元素的迭代器。
- **`binary_search(begin, end, val)`**：返回 `bool` 值，判断 `val` 是否存在。
### 1. `sort` (排序)
将 `a[1...n]` 按升序排序。
```cpp
sort(a + 1, a + n + 1);// 如果要自定义比较函数 cmp
sort(a + 1, a + n + 1, cmp);
```
### 2. `lower_bound` & `upper_bound` (二分查找位置)
注意：数组必须已升序排好。
- **`lower_bound`**：查找第一个 **$\ge x$** 的位置。
- **`upper_bound`**：查找第一个 **$> x$** 的位置。
```cpp
// 找第一个 >= x 的下标
int pos1 = lower_bound(a + 1, a + n + 1, x) - a;
// 找第一个 > x 的下标
int pos2 = upper_bound(a + 1, a + n + 1, x) - a;
// 检查是否存在：如果 pos > n 或者 a[pos] != x，则说明没找到
```
### 3. `binary_search` (判断是否存在)
返回 `bool` 值。
```cpp
bool exists = binary_search(a + 1, a + n + 1, x);
```
### 4. `unique` (去重)
注意：使用前必须先 `sort`。`unique` 返回的是去重后“新数组”末尾的下一个地址。
```cpp
sort(a + 1, a + n + 1);
int newLen = unique(a + 1, a + n + 1) - (a + 1); 
// newLen 就是去重后剩余元素的个数
// 此时有效数据存放在 a[1] 到 a[newLen]
```
### 5. `reverse` (翻转)
将 `a[1...n]` 的顺序颠倒。
```cpp
reverse(a + 1, a + n + 1);
```
### 6.`max_element` & `min_element`(类似`argmax`)
```cpp
// 找 a[1...n] 中的最大值
int maxVal = *max_element(a + 1, a + n + 1);
// 找最大值的下标
int maxIdx = max_element(a + 1, a + n + 1) - a;
```


## 并查集
### 看到以下描述，基本就是并查集：
- **“亲戚关系”**：如果 A 是 B 的亲戚，B 是 C 的亲戚，那么 A 和 C 也是亲戚。
- **“连通分量”**：求一个图中共有多少个独立的部落/群体。
- **“最小生成树”**：Kruskal 算法的内核就是并查集。
- **“动态增边”**：不断告诉你哪两个点连通了，问你现在的连通状态。
---
### 实战技巧：下标从 1 开始的完整模板
```cpp
#include <iostream>
#include <vector>

using namespace std;

const int MAXN = 10005;
int fa[MAXN];
// 初始化
void init(int n) {
    for (int i = 1; i <= n; i++) fa[i] = i;
}
// 查找（路径压缩版）
int find(int x) {
    return fa[x] == x ? x : (fa[x] = find(fa[x]));
}
// 合并
void merge(int i, int j) {
    fa[find(i)] = find(j);
}
int main() {
    int n, m; // n个元素, m个操作
    cin >> n >> m;
    init(n); //========初始化是重中之重！===========
    while (m--) {
        int z, x, y;
        cin >> z >> x >> y;
        if (z == 1) { // 合并
            merge(x, y);
        } else { // 查询是否在同一集合
            if (find(x) == find(y)) cout << "Y" << endl;
            else cout << "N" << endl;
        }
    }
    return 0;
}
```

## 差分算法
### 一、 一维差分模板题

**场景：** 长度为 $L$ 的公路上，有 $M$ 个操作，每个操作让 $[l, r]$ 区间的树木高度增加 $v$，最后输出每棵树的高度。

C++

```
#include <iostream>
#include <vector>
using namespace std;

const int MAXN = 100005; // 根据题目范围调整
long long a[MAXN];       // 原数组（如果初始全是0可不写）
long long d[MAXN];       // 差分数组

int main() {
    int n, m;
    scanf("%d %d", &n, &m);
    
    // 1. 初始化差分数组
    // 如果题目给了初始高度，则构造差分；如果初始全是0，此步跳过
    for (int i = 1; i <= n; i++) {
        scanf("%lld", &a[i]);
        d[i] = a[i] - a[i-1];
    }
    
    // 2. 区间修改 [l, r] 增加 v
    while (m--) {
        int l, r, v;
        scanf("%d %d %d", &l, &r, &v);
        d[l] += v;
        d[r + 1] -= v; // 注意越界保护，MAXN要开够
    }
    
    // 3. 前缀和还原
    for (int i = 1; i <= n; i++) {
        a[i] = a[i-1] + d[i];
        printf("%lld%c", a[i], i == n ? '\n' : ' ');
    }
    
    return 0;
}
```

---

### 二、 二维差分模板题

**场景：** 在一个 $N \times M$ 的矩阵中，有 $K$ 个操作。每个操作将左上角 $(x1, y1)$ 到右下角 $(x2, y2)$ 的子矩阵内所有数加上 $v$。最后输出整个矩阵。

**原理口诀：** “一加二减三补回”。

C++

```
#include <iostream>
using namespace std;

const int MAXN = 1005;
int d[MAXN][MAXN]; // 差分矩阵
int a[MAXN][MAXN]; // 结果矩阵

int main() {
    int n, m, k;
    scanf("%d %d %d", &n, &m, &k);
    
    // 1. 区间修改（核心操作）
    while (k--) {
        int x1, y1, x2, y2, v;
        scanf("%d %d %d %d %d", &x1, &y1, &x2, &y2, &v);
        d[x1][y1] += v;
        d[x2 + 1][y1] -= v;
        d[x1][y2 + 1] -= v;
        d[x2 + 1][y2 + 1] += v;
    }
    
    // 2. 二维前缀和还原
    // 公式：a[i][j] = a[i-1][j] + a[i][j-1] - a[i-1][j-1] + d[i][j]
    for (int i = 1; i <= n; i++) {
        for (int j = 1; j <= m; j++) {
            a[i][j] = a[i - 1][j] + a[i][j - 1] - a[i - 1][j - 1] + d[i][j];
            printf("%d%c", a[i][j], j == m ? '\n' : ' ');
        }
    }
    
    return 0;
}
```

---

### 💡 考场避坑关键点

1. 数组越界（最容易错）：
    
    在一维 d[r+1] 和二维 d[x2+1][y1] 等操作中，如果 $r$ 或 $x2$ 是最大范围 $N$，那么 $r+1$ 就会越界。
    
    - **对策：** 数组大小 `MAXN` 一定要比题目给的 $N$ 多开 5 到 10 个空间。
        
2. Long Long 溢出：
    
    如果修改次数 $M$ 很多，或者增加的值 $v$ 很大，还原后的结果可能会超过 int 的 $2 \times 10^9$ 限制。
    
    - **对策：** 涉及结果累加的数组（`a` 和 `d`）统一开 `long long`。
        
3. 二维差分的图形记忆：
    
    你可以想象在 $(x1, y1)$ 点按了一下“开关”开始加分，但在两个边界处需要减去，最后在重叠交点处补回来。