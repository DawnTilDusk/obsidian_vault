- [[#1. 最长回文子串：算法全景图|1. 最长回文子串：算法全景图]]
- [[#2. 核心算法笔记|2. 核心算法笔记]]
	- [[#2. 核心算法笔记#A. 中心扩展法 (The Intuitive Way)|A. 中心扩展法 (The Intuitive Way)]]
	- [[#2. 核心算法笔记#B. Manacher 算法 (The Professional Way)|B. Manacher 算法 (The Professional Way)]]
		- [[#B. Manacher 算法 (The Professional Way)#1. 解决奇偶性：预处理|1. 解决奇偶性：预处理]]
		- [[#B. Manacher 算法 (The Professional Way)#2. 三大核心变量|2. 三大核心变量]]
		- [[#B. Manacher 算法 (The Professional Way)#3. 镜像跳跃逻辑 (最精妙处)|3. 镜像跳跃逻辑 (最精妙处)]]
		- [[#B. Manacher 算法 (The Professional Way)#4. 为什么是 $O(n)$？|4. 为什么是 $O(n)$？]]
- [[#3. 代码示例|3. 代码示例]]
		- [[#B. Manacher 算法 (The Professional Way)#1.中心扩展|1.中心扩展]]
		- [[#B. Manacher 算法 (The Professional Way)#2. dp+二维状态压缩一维|2. dp+二维状态压缩一维]]
		- [[#B. Manacher 算法 (The Professional Way)#3. Manacher（马拉车）|3. Manacher（马拉车）]]

## 1. 最长回文子串：算法全景图

|**方法**|**时间复杂度**|**空间复杂度**|**核心思想**|**适用场景**|
|---|---|---|---|---|
|**暴力枚举**|$O(n^3)$|$O(1)$|遍历所有子串，逐一判断回文|仅限初学者理解|
|**动态规划 (DP)**|$O(n^2)$|$O(n^2) \to O(n)$|利用 `dp[i][j]` 依赖 `dp[i+1][j-1]`|需记录所有子串回文状态时|
|**中心扩展法**|$O(n^2)$|$O(1)$|以每个点（或点间隙）为中心向两边扩展|**面试首选**，内存最优|
|**Manacher (马拉车)**|$O(n)$|$O(n)$|利用对称性跳过重复计算|**性能巅峰**，竞赛必备|

---

## 2. 核心算法笔记

### A. 中心扩展法 (The Intuitive Way)

**逻辑：** 回文串是中心对称的。遍历字符串，以每个字符为中心（奇数长度）或每两个字符中间为中心（偶数长度）向外扩散，直到字符不相等或越界。
```c++
// 伪代码逻辑
for (int i = 0; i < n; i++) {
    expand(i, i);   // 奇数长度回文
    expand(i, i+1); // 偶数长度回文
}
```

### B. Manacher 算法 (The Professional Way)

#### 1. 解决奇偶性：预处理

通过插入特殊字符（如 `#`），让所有回文串都变成奇数长度。

- `aba` $\to$ `^#a#b#a#$`
- `aa` $\to$ `^#a#a#$`
#### 2. 三大核心变量

- `P[i]`：以 $i$ 为中心的回文**半径**（在处理后的字符串中）。
- `C` (Center)：当前已知扩展最远的回文串的**中心**。
- `R` (Right)：该回文串的**右边界**（$R = C + P[C]$）。
#### 3. 镜像跳跃逻辑 (最精妙处)

计算 `P[i]` 时，先找 `i` 关于 `C` 的镜像点 `mirror = 2*C - i`。

$$P[i] = \begin{cases} \min(R - i, P[mirror]) & \text{if } i < R \\ 0 & \text{if } i \ge R \end{cases}$$

> **笔记：** `R - i` 是一个“安全区”。在大回文串的遮伞下，`i` 处的情况一定和 `mirror` 处对称。但超出 `R` 的部分我们看不见，必须用 `while` 循环继续手动匹配。

#### 4. 为什么是 $O(n)$？

虽然有 `while` 循环，但 `R` 只会增加，不会回退。`R` 最多从字符串开头移动到结尾，所以总时间复杂度是线性的。


## 3. 代码示例

#### 1.中心扩展
我的写法（中心扩展）
**这样写分开了奇偶，不够优雅**
```cpp
class Solution {
public:
    string longestPalindrome(string s) {
        int id = 0, len = 0;
        for(int i=0; i<s.length(); i++){
            int l = i, r = i, cnt = 0;
            while(l-1>=0 && r+1<s.length() && s[l-1]==s[r+1]){
                l--, r++, cnt++;
            }
            //cout<<i<<" "<<cnt<<endl;
            if(cnt > len)
                len = cnt, id = i;
        }
        string ans = s.substr(id-len, len*2+1);
        //cout <<"sdofjo"<<endl;
        for(int i=0; i<s.length()-1; i++){
            int l = i+1, r = i, cnt = 0;
            while(l-1>=0 && r+1<s.length() && s[l-1] == s[r+1]){
                l--, r++, cnt++;
            }
            //cout<<i<<" "<<cnt<<endl;
            if(cnt > len)
                len = cnt, id = i;
        }
        return ans.length() > len*2 ? ans : s.substr(id-len+1, len*2);
    }
};
```

**更优雅的写法**
```cpp
class Solution {
public:
    string longestPalindrome(string s) {
        if (s.length() < 2) return s;
        int start = 0, maxLen = 0;
        for (int i = 0; i < s.length(); i++) {
            // 策略：以 i 为中心（奇数），或者以 i 和 i+1 为中心（偶数）
            expand(s, i, i, start, maxLen);      // 奇数扩展
            expand(s, i, i + 1, start, maxLen);  // 偶数扩展
        }
        
        return s.substr(start, maxLen);
    }

private:
    void expand(const string& s, int l, int r, int& start, int& maxLen) {
        while (l >= 0 && r < s.length() && s[l] == s[r]) {
            l--;
            r++;
        }
        // 循环跳出时，s[l] != s[r]，所以有效长度是 (r-1) - (l+1) + 1 = r - l - 1
        int currentLen = r - l - 1;
        if (currentLen > maxLen) {
            maxLen = currentLen;
            start = l + 1; // 回退到上一个匹配的位置
        }
    }
};
```

#### 2. dp+二维状态压缩一维
```cpp
class Solution {
public:
    string longestPalindrome(string s) {
        int n = s.length();
        if (n < 2) return s;

        // dp[i] 表示当前遍历到的子串 [i...j] 是否为回文
        // 这里的 j 是外层循环变量
        vector<bool> dp(n, false);
        int maxLen = 1;
        int start = 0;

        for (int j = 0; j < n; j++) {
            for (int i = 0; i <= j; i++) {
                // 如果头尾相等，且（子串长度<=2 或 去掉头尾后的子串是回文）
                // 注意：由于 i 是正序遍历，计算 dp[i] 时，
                // 这里的 dp[i+1] 其实还是上一轮 j-1 时的状态，即 dp[i+1][j-1]
                if (s[i] == s[j] && (j - i < 3 || dp[i + 1])) {
                    dp[i] = true;
                    if (j - i + 1 > maxLen) {
                        maxLen = j - i + 1;
                        start = i;
                    }
                } else {
                    dp[i] = false; // 必须重置，因为这一轮 [i...j] 不是回文
                }
            }
        }
        return s.substr(start, maxLen);
    }
};
```

#### 3. Manacher（马拉车）
```cpp
class Solution {
public:
    string longestPalindrome(string s) {
        // 1. 预处理
        string t = "^#";
        for (char c : s) {
            t += c;
            t += "#";
        }
        t += "$";

        int n = t.length();
        vector<int> P(n, 0);
        int C = 0, R = 0;

        for (int i = 1; i < n - 1; i++) {
            // 2. 核心：尝试利用对称性跳过计算
            if (i < R) {
                P[i] = min(R - i, P[2 * C - i]);
            }

            // 3. 中心扩展（尝试扩大半径）
            while (t[i + (P[i] + 1)] == t[i - (P[i] + 1)]) {
                P[i]++;
            }

            // 4. 更新 C 和 R
            if (i + P[i] > R) {
                C = i;
                R = i + P[i];
            }
        }

        // 5. 寻找最大半径并截取原串
        int maxLen = 0, centerIndex = 0;
        for (int i = 1; i < n - 1; i++) {
            if (P[i] > maxLen) {
                maxLen = P[i];
                centerIndex = i;
            }
        }
        
        // 映射回原串下标：起始位置 = (中心位置 - 最大长度) / 2
        return s.substr((centerIndex - maxLen) / 2, maxLen);
    }
};
```
