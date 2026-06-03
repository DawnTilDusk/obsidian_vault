### 方法 1：等价标准形法（“打回原形”法）

**核心逻辑：** 任何复杂的矩阵，只要在两边乘上可逆矩阵，就可以变成最简单的“左上角全是 1，其他全是 0”的块对角矩阵（即等价标准形）。可逆变换不改变矩阵的秩。

**具体推导：**

1. **化简 $A$：** 存在可逆矩阵 $P$ 和 $Q$，使得 $PAQ = \begin{bmatrix} E_r & 0 \\ 0 & 0 \end{bmatrix}$，其中 $r = \text{rank}(A)$。
    
2. **构造恒等变形：** $\text{rank}(AB) = \text{rank}(PAB)$ （左乘可逆矩阵秩不变）。
    
    为了把化简后的 $A$ 用上，我们在中间强行插入一个 $QQ^{-1} = E$：
    
    $$\text{rank}(PAB) = \text{rank}(PAQQ^{-1}B) = \text{rank}\left( \begin{bmatrix} E_r & 0 \\ 0 & 0 \end{bmatrix} Q^{-1}B \right)$$
    
3. **分块计算：** 把后面的 $Q^{-1}B$ 按照对应的行数切开，设为 $\begin{bmatrix} B_1 \\ B_2 \end{bmatrix}$。乘起来就会发现：
    
    $$\begin{bmatrix} E_r & 0 \\ 0 & 0 \end{bmatrix} \begin{bmatrix} B_1 \\ B_2 \end{bmatrix} = \begin{bmatrix} B_1 \\ 0 \end{bmatrix}$$
    
    所以 $\text{rank}(AB) = \text{rank}(B_1)$。
    
4. **放缩：** 整个矩阵 $Q^{-1}B$ 的秩不变，依然是 $\text{rank}(B)$。而 $\begin{bmatrix} B_1 \\ B_2 \end{bmatrix}$ 的秩最多等于 $B_1$ 的秩加上 $B_2$ 的秩。
    
    因为 $B_2$ 只有 $n-r$ 行，所以 $\text{rank}(B_2) \le n - \text{rank}(A)$。
    
    根据 $\text{rank}(B) \le \text{rank}(B_1) + \text{rank}(B_2)$，稍微移项就能得出 Sylvester 不等式。
    

**评价：** 这是最踏实、最传统的代数证法。它完全基于矩阵的坐标变换，极其严谨，但也略显繁琐。

---

### 方法 2：分块矩阵初等变换法（“降维打击”的矩阵版）

**核心逻辑：** 如果我们想研究 $A, B$ 和 $AB$ 的关系，不如把它们全部塞进一个更高维度的“超级大矩阵”里，然后像玩魔方一样，用分块初等变换把 $AB$ “转”出来。这是极其常用且优雅的技巧。

**具体推导：**

1. **构造大矩阵：** 构造一个分块矩阵 $M = \begin{bmatrix} E_n & B \\ A & 0 \end{bmatrix}$。
    
2. **用行变换“消元”：** 我们把第一行块左乘 $A$ 然后减到第二行块（相当于矩阵左乘一个分块初等矩阵 $\begin{bmatrix} E & 0 \\ -A & E \end{bmatrix}$）：
    
    $$\begin{bmatrix} E & 0 \\ -A & E \end{bmatrix} \begin{bmatrix} E_n & B \\ A & 0 \end{bmatrix} = \begin{bmatrix} E_n & B \\ 0 & -AB \end{bmatrix}$$
    
    因为分块初等变换不改变秩，所以 $\text{rank}(M) = n + \text{rank}(-AB) = n + \text{rank}(AB)$。
    
3. **从另一个角度看秩（抽屉原理）：** 我们再回头看原矩阵 $\begin{bmatrix} E_n & B \\ A & 0 \end{bmatrix}$。
    
    它左下角有一个 $A$，右上角有一个 $B$。因为右下角是 $0$，这意味着 $A$ 所在的非零行与 $B$ 所在的非零列，在“空间”上是完全错开的、没有重叠的。
    
    所以，这个大矩阵的秩，必定**大于或等于**这两个部分各自秩的和：$\text{rank}(M) \ge \text{rank}(A) + \text{rank}(B)$。
    
4. **得出结论：** 把两步结合起来：$n + \text{rank}(AB) \ge \text{rank}(A) + \text{rank}(B)$，移项即得。
    

**评价：** 极其精妙！它把复杂的代数不等式，转化成了更高维度下矩阵排布的“几何容积”问题。

---

### 方法 3：零空间维度放缩法（与我们刚才推导的“对偶”）

**核心逻辑：** 既然我们刚才可以用像空间（Image）的维度来推导，那自然也可以用核空间（Null space）的维度来推导。这是最纯粹的“空间映射”视角。

**具体推导：**

1. **寻找零空间的包含关系：** 考虑 $AB\mathbf{x} = \mathbf{0}$ 这个方程。
    
    如果向量 $\mathbf{x} \in \text{Nul}(AB)$，这意味着 $A(B\mathbf{x}) = \mathbf{0}$。
    
    换句话说，$B\mathbf{x}$ 这个向量，必须落在 $A$ 的零空间 $\text{Nul}(A)$ 里。
    
2. **构造限制映射：** 我们把映射 $B$ 的定义域，**限制**在 $\text{Nul}(AB)$ 这个子空间上。
    
    根据我们刚才的分析，这个限制映射 $B|_{\text{Nul}(AB)}$ 的值域必定是 $\text{Nul}(A)$ 的一个子空间。
    
    所以它的像空间的维度：$\dim(\text{im}(B|_{\text{Nul}(AB)})) \le \dim(\text{Nul}(A))$。
    
3. **对这个限制映射使用秩零定理：**
    
    （定义域维度） = （核空间维度） + （像空间维度）
    
    $$\dim(\text{Nul}(AB)) = \dim(\text{ker}(B|_{\text{Nul}(AB)})) + \dim(\text{im}(B|_{\text{Nul}(AB)}))$$
    
    - 它的定义域维度就是 $\dim(\text{Nul}(AB))$。
        
    - 它的核空间，就是要求 $\mathbf{x} \in \text{Nul}(AB)$ 且 $B\mathbf{x} = \mathbf{0}$。但这其实就是整个 $\text{Nul}(B)$，所以核维度是 $\dim(\text{Nul}(B))$。
        
4. **放缩并代入公式：**
    
    $$\dim(\text{Nul}(AB)) \le \dim(\text{Nul}(B)) + \dim(\text{Nul}(A))$$
    
    把 $\dim(\text{Nul}(X)) = \text{列数} - \text{rank}(X)$ 代入上式：
    
    $$p - \text{rank}(AB) \le (p - \text{rank}(B)) + (n - \text{rank}(A))$$
    
    化简后，完美得到 $\text{rank}(AB) \ge \text{rank}(A) + \text{rank}(B) - n$。