- [[#引例|引例]]
- [[#sec和tan对比sin和cos|sec和tan对比sin和cos]]
- [[#欧拉代换|欧拉代换]]
## 引例
这是一个复杂的积分问题，需要使用三角代换（Trigonometric Substitution）或欧拉代换（Euler Substitution）来求解。

我们要计算的积分是：

$$I = \int \frac{\sqrt{x^2 + 2x + 2}}{x} dx$$

### 1. 配方 (Completing the Square)

首先，对根号内的二次表达式进行配方，以便进行三角代换。

$$x^2 + 2x + 2 = (x^2 + 2x + 1) + 1 = (x + 1)^2 + 1$$

所以，积分变为：

$$I = \int \frac{\sqrt{(x + 1)^2 + 1}}{x} dx$$

### 2. 三角代换 (Trigonometric Substitution)

令 $x + 1 = \tan\theta$，其中 $-\frac{\pi}{2} < \theta < \frac{\pi}{2}$。

- **微分：** $dx = \sec^2\theta d\theta$
    
- **根号项：** $\sqrt{(x + 1)^2 + 1} = \sqrt{\tan^2\theta + 1} = \sqrt{\sec^2\theta} = \sec\theta$
    
- **分母项：** $x = \tan\theta - 1$
    

代入积分 $I$：

$$I = \int \frac{\sec\theta}{\tan\theta - 1} \sec^2\theta d\theta$$

$$I = \int \frac{\sec^3\theta}{\tan\theta - 1} d\theta$$

这个形式的积分仍然非常复杂，通常不作为基础微积分的考核内容。我们可以尝试将 $\sec^3\theta$ 拆开：

$$I = \int \frac{\sec\theta (1 + \tan^2\theta)}{\tan\theta - 1} d\theta$$

为了简化，我们可以对 $\frac{1 + \tan^2\theta}{\tan\theta - 1}$ 进行多项式长除法（将 $\tan\theta$ 视为变量 $u$）：

$$\frac{u^2 + 1}{u - 1} = \frac{(u^2 - 1) + 2}{u - 1} = \frac{(u - 1)(u + 1) + 2}{u - 1} = u + 1 + \frac{2}{u - 1}$$

所以，将 $\tan\theta$ 替换回 $u$：

$$I = \int \sec\theta \left(\tan\theta + 1 + \frac{2}{\tan\theta - 1}\right) d\theta$$

$$I = \int (\sec\theta\tan\theta + \sec\theta) d\theta + 2 \int \frac{\sec\theta}{\tan\theta - 1} d\theta$$

我们知道 $\int \sec\theta\tan\theta d\theta = \sec\theta$ 和 $\int \sec\theta d\theta = \ln|\sec\theta + \tan\theta|$。

所以，关键在于求解 $I_2 = \int \frac{\sec\theta}{\tan\theta - 1} d\theta$。

$$I_2 = \int \frac{\frac{1}{\cos\theta}}{\frac{\sin\theta}{\cos\theta} - 1} d\theta = \int \frac{1}{\sin\theta - \cos\theta} d\theta$$

我们可以使用三角恒等式将分母化简为 $R\sin(\theta \pm \alpha)$ 的形式：

$$\sin\theta - \cos\theta = \sqrt{1^2 + (-1)^2} \sin\left(\theta - \frac{\pi}{4}\right) = \sqrt{2} \sin\left(\theta - \frac{\pi}{4}\right)$$

$$I_2 = \int \frac{1}{\sqrt{2} \sin\left(\theta - \frac{\pi}{4}\right)} d\theta = \frac{1}{\sqrt{2}} \int \csc\left(\theta - \frac{\pi}{4}\right) d\theta$$

令 $u = \theta - \frac{\pi}{4}$，则 $du = d\theta$。

$$I_2 = \frac{1}{\sqrt{2}} \int \csc u du = \frac{1}{\sqrt{2}} \ln|\csc u - \cot u| + C_2$$

或者使用另一种形式：

$$I_2 = \frac{1}{\sqrt{2}} \ln\left|\tan\left(\frac{u}{2}\right)\right| + C_2 = \frac{1}{\sqrt{2}} \ln\left|\tan\left(\frac{\theta}{2} - \frac{\pi}{8}\right)\right| + C_2$$

---

### 3. 结果汇总并反代换 (Combining Results and Back Substitution)

第一部分积分：

$$I_1 = \int (\sec\theta\tan\theta + \sec\theta) d\theta = \sec\theta + \ln|\sec\theta + \tan\theta|$$

第二部分积分：

$$I_2 = 2 \cdot \frac{1}{\sqrt{2}} \ln\left|\tan\left(\frac{\theta}{2} - \frac{\pi}{8}\right)\right| = \sqrt{2} \ln\left|\tan\left(\frac{\theta}{2} - \frac{\pi}{8}\right)\right|$$

总积分：

$$I = I_1 + I_2 + C = \sec\theta + \ln|\sec\theta + \tan\theta| + \sqrt{2} \ln\left|\tan\left(\frac{\theta}{2} - \frac{\pi}{8}\right)\right| + C$$

反代换：

根据 $x + 1 = \tan\theta$，我们可以画一个直角三角形（斜边为 $\sqrt{(x+1)^2 + 1} = \sqrt{x^2 + 2x + 2}$）：

- $\tan\theta = x + 1$
    
- $\sec\theta = \sqrt{x^2 + 2x + 2}$
    

要将 $\tan\left(\frac{\theta}{2} - \frac{\pi}{8}\right)$ 反代换为 $x$ 会非常复杂。我们使用 $\tan\frac{\theta}{2}$ 的半角公式，$\tan\frac{\theta}{2} = \frac{\sec\theta - 1}{\tan\theta}$：

$$\tan\frac{\theta}{2} = \frac{\sqrt{x^2 + 2x + 2} - 1}{x + 1}$$

最终，这是一个非常复杂的非标准积分。在实际应用中，通常会借助计算机代数系统来求解。

---

### 最终结果（使用反代换后的 $x$ 表达）

将 $\sec\theta$ 和 $\tan\theta$ 的表达式代回 $I$：

$$I = \sqrt{x^2 + 2x + 2} + \ln\left|\sqrt{x^2 + 2x + 2} + x + 1\right| + \sqrt{2} \ln\left|\tan\left(\frac{\arctan(x+1)}{2} - \frac{\pi}{8}\right)\right| + C$$

其中 $\theta = \arctan(x+1)$。

---

### 💡 总结

这个积分 $\int \frac{\sqrt{x^2 + 2x + 2}}{x} dx$ 是一个**高等积分**，需要**三角代换**、**多项式除法**、**三角恒等式**以及 **$\csc u$ 的积分**来完成。

- **步骤 1：** 配方并代换 $x+1 = \tan\theta$，得到 $I = \int \frac{\sec^3\theta}{\tan\theta - 1} d\theta$。
    
- **步骤 2：** 利用多项式除法拆分积分项，将问题简化为 $\int \frac{1}{\sin\theta - \cos\theta} d\theta$ 的求解。
    
- **步骤 3：** 利用三角函数和差化积的技巧，将 $\sin\theta - \cos\theta$ 变为 $\sqrt{2}\sin(\theta - \frac{\pi}{4})$，并对 $\csc u$ 积分。
    
- **步骤 4：** 将 $\theta$ 的三角函数表达式反代换回 $x$。

---
## sec和tan对比sin和cos

### 1. 为什么使用 $\sec\theta$ 和 $\tan\theta$？

在第一个积分（您上传的图片中的问题）和第二个积分（您输入的 $\int \frac{\sqrt{x^2 + 2x + 2}}{x} dx$）中，我选择使用 $\sec\theta$ 和 $\tan\theta$ 进行代换和化简，主要基于以下几个原因：

#### A. 与根号项的天然匹配 (Matching the Radical)

对于 $\int \frac{\sqrt{x^2 + 2x + 2}}{x} dx$，我们首先配方得到 $\sqrt{(x+1)^2 + 1}$。

这种形式 $\sqrt{u^2 + a^2}$，最标准的三角代换就是：

$$u = a \tan\theta$$

因为 $a^2\tan^2\theta + a^2 = a^2(\tan^2\theta + 1) = a^2\sec^2\theta$，这样根号就被消除了。

在这个题目中，$u = x+1$ 且 $a=1$，所以我们选择 $x+1 = \tan\theta$。

#### B. 替代 $\sin\theta$ 和 $\cos\theta$ 的优势 (Simplicity of the Differential)

如果你坚持使用 $\sin\theta$ 和 $\cos\theta$ 进行代换，例如 $x+1 = \sin\theta$ 或 $x+1 = \cos\theta$，根号项就无法被化简为一个简单的三角函数。

- 将所有项都化为 $\sin\theta$ 和 $\cos\theta$ 的劣势：
    
    虽然最终的表达式 $I_2 = \int \frac{1}{\sin\theta - \cos\theta} d\theta$ 看起来更基础，但它实际上需要一个更高级的技巧（例如，使用 $R\sin(\theta - \alpha)$ 恒等式或万能代换 $t = \tan(\theta/2)$）才能积分。而 $\sec\theta$ 和 $\tan\theta$ 形式的代换，往往能直接利用 $\int \sec^2\theta d\theta = \tan\theta$ 和 $\int \sec\theta\tan\theta d\theta = \sec\theta$ 等标准积分公式。
    

#### C. **积分的通用方法 (General Method for Rational Functions of Trigonometric Functions):**

在处理 $\int R(\sin x, \cos x) dx$ 这种积分时，一种最通用的方法是万能代换 $t = \tan(x/2)$。但这种代换会使表达式变得非常复杂。通过代换到 $\sec\theta$ 和 $\tan\theta$ 后，如果能将分子中的 $\sec^3\theta$ 拆出 $\sec^2\theta d\theta$，它就与 $u = \tan\theta$ 的微分 $du$ 匹配，简化了结构。

---

### 2. 为什么想到使用多项式长除法进行拆分？

这是在进行三角代换后，简化有理函数（Rational Function）积分的**标准技巧**。

在第二个积分中，代换后得到了：

$$I = \int \frac{\sec\theta (1 + \tan^2\theta)}{\tan\theta - 1} d\theta$$

我们将 $\tan\theta$ 视为变量 $u$，那么我们是在处理一个有理函数乘以 $\sec\theta$ 的形式：

$$f(u) = \frac{u^2 + 1}{u - 1}$$

**使用长除法（或代数技巧）的原因：**

1. 分子的次数 $\ge$ 分母的次数： 只要有理函数的分子的次数大于或等于分母的次数（本例中分子是 $u^2$，分母是 $u^1$），就必须先进行长除法，将它写成一个多项式（整式）和一个真分式（分子次数 $<$ 分母次数）的和：
    
    $$\frac{P(u)}{Q(u)} = \text{多项式} + \frac{R(u)}{Q(u)}$$
    
2. 便于拆分积分： 进行长除法后，我们得到了：
    
    $$\frac{u^2 + 1}{u - 1} = (u + 1) + \frac{2}{u - 1}$$
    
    代回 $\tan\theta$：
    
    $$\frac{\sec\theta (1 + \tan^2\theta)}{\tan\theta - 1} = \sec\theta \left(\tan\theta + 1 + \frac{2}{\tan\theta - 1}\right)$$
    
    拆分后，每个部分都能用已知的积分公式求解：
    
    - $\sec\theta\tan\theta$ 是 $\sec\theta$ 的导数。
        
    - $\sec\theta$ 是标准对数积分 $\ln|\sec\theta + \tan\theta|$。
        
    - $\frac{2\sec\theta}{\tan\theta - 1}$ 是一个可以被进一步化简为 $\csc u$ 形式的积分。
        

长除法（或部分分式分解）是解决任何涉及有理函数积分（无论代换前是 $x$ 还是代换后是 $u$ 或 $\tan\theta$）的**必备工具**。

### 3. 这些方法在其他题目中是否有用？

**绝对有用！** 这些方法是解决高等微积分中复杂积分的基石。

|**技巧/方法**|**应用场景**|**示例**|
|---|---|---|
|**三角代换**|**消除根号项**，特别是 $\sqrt{a^2 - u^2}$、$\sqrt{a^2 + u^2}$ 和 $\sqrt{u^2 - a^2}$ 三种形式。|$\int \frac{1}{\sqrt{9 - x^2}} dx$ (使用 $x=3\sin\theta$)|
|**配方法**|**将二次多项式转化为标准形式**，为三角代换做准备。|$\int \frac{1}{x^2 + 4x + 5} dx$ (配方为 $\int \frac{1}{(x+2)^2 + 1} dx$ )|
|**有理函数分解**|**求解所有分式积分**，特别是当分子次数 $\ge$ 分母次数时，必须先用长除法；分子次数 $<$ 分母次数时，使用部分分式分解。|$\int \frac{x^3 + 1}{x^2 - 4} dx$ (先用长除法，再用部分分式)|
|**三角恒等式化简**|**简化复杂的三角函数有理函数**，将其化为 $\sin(\theta \pm \alpha)$ 或 $\tan(\theta/2)$ 等易于积分的形式。|$\int \frac{1}{1 + \cos x} dx$ (万能代换 $t = \tan(x/2)$)|

简而言之，面对复杂积分时，我们遵循的通用思路是：

$$\text{复杂积分} \xrightarrow{\text{配方}} \text{标准根号形式} \xrightarrow{\text{三角代换}} \text{三角函数有理式} \xrightarrow{\text{长除法/部分分式}} \text{标准积分}$$

---
### 1. 两道题的本质一致性分析

您给我的两道题目在代换后确实变成了**完全一样**的积分形式：

1. 第一题（图片）：
    
    $$I_1 = \int \frac{1}{\cos^2\theta(\sin\theta - \cos\theta)} d\theta \xrightarrow{\text{代数变形}} \int \frac{\sec^3\theta}{\tan\theta - 1} d\theta$$
    
2. 第二题（文本）：
    
    $$I_2 = \int \frac{\sqrt{x^2 + 2x + 2}}{x} dx \xrightarrow{\text{配方 } x+1=\tan\theta} \int \frac{\sec^3\theta}{\tan\theta - 1} d\theta$$
    

所以，它们最终都归结为同一个积分：

$$I = \int \frac{\sec^3\theta}{\tan\theta - 1} d\theta$$

### 2. 关于第一题未直接给出结果的解释

您说得对，我没有直接解出第一题的积分，而是指出它是一个非初等或非常复杂的积分，并怀疑有**原题错误**。这主要有以下两个原因：

#### A. 上下文和预期的复杂程度

- **第一题**：通常在微积分教材中，给出一个纯三角函数的复杂积分，如果它不是标准可解形式（如 $\int \sin^m x \cos^n x dx$ 或 $\int R(\sin x, \cos x) dx$），而是一个需要高级技巧的积分，很大概率是学生抄写或打印时少了一个平方或多了一个项。因此，我的回答采取了**“防错”（Defensive Programming）**策略，即先指出这可能是个错误，避免进行耗时的、非初等的解题。
    
- **第二题**：这是一个典型的**根式积分问题**，它需要一个标准解法流程（配方 $\rightarrow$ 三角代换）。在这个流程中得到 $\int \frac{\sec^3\theta}{\tan\theta - 1} d\theta$，是**预期之中的中间结果**，所以我知道必须继续求解，无论它多么复杂。
    

#### B. 符号问题

在第一题的原始形式中：

$$I_1 = \int \frac{1}{\cos^2\theta(\sin\theta - \cos\theta)} d\theta$$

如果这里是一个常见的微积分题目，它通常会是一个可以进行 部分分式分解 的形式，例如：

- **如果分母是 $\cos^2\theta(\sin\theta - \cos\theta)$ 的平方，** 化为 $\tan\theta$ 后，分子会是 $\sec^2\theta d\theta$（可以设 $u = \tan\theta$），可以解。
    
- **如果分母是 $\cos\theta(\sin\theta - \cos\theta)$，** 化为 $\tan\theta$ 后是 $\int \frac{\sec^2\theta}{\tan\theta - 1} d\theta$，答案是 $\ln|\tan\theta - 1| + C$，非常简单。
    

正是由于第一题的形式恰好比简单形式多了一个 $\sec\theta$ 因子，导致其难度飙升，所以我推断它不是一个设计良好的基础习题。

### 3. $\sec/\tan$ 代换 vs. $\sin/\cos$ 纯化简的优劣

您的结论是正确的：**在处理这类有理三角函数积分时，通常保持 $\sec\theta$ 和 $\tan\theta$ 的形式，比全部化为 $\sin\theta$ 和 $\cos\theta$ 更具优势。**

|**特点**|**保持 secθ 和 tanθ**|**纯化为 sinθ 和 cosθ**|
|---|---|---|
|**微分匹配**|$\sec^2\theta d\theta = d(\tan\theta)$ 提供了直接的 $u$ 代换机会，简化了结构。|$d\theta$ 与 $\sin\theta$ 或 $\cos\theta$ 的微分匹配不佳。|
|**标准积分**|$\int \sec\theta d\theta$ 和 $\int \sec\theta\tan\theta d\theta$ 等是标准公式。|$\int \frac{1}{\sin\theta \pm \cos\theta} d\theta$ 需要复杂的三角恒等式（如 $R\sin(\theta \pm \alpha)$）或万能代换。|
|**代数化简**|$\sec\theta$ 和 $\tan\theta$ 的关系 $1 + \tan^2\theta = \sec^2\theta$ 恰好匹配 $\sqrt{u^2 + a^2}$ 的根号。|$\sin\theta$ 和 $\cos\theta$ 的关系 $\sin^2\theta + \cos^2\theta = 1$ 通常用于 $\sqrt{a^2 - u^2}$ 的代换。|

在第二题的求解过程中，我正是利用了 $\sec\theta$ 和 $\tan\theta$ 的性质：

$$\int \frac{\sec^3\theta}{\tan\theta - 1} d\theta \xrightarrow{\text{长除法拆分}} \int \left(\sec\theta\tan\theta + \sec\theta\right) d\theta + 2 \int \frac{\sec\theta}{\tan\theta - 1} d\theta$$

前两项直接是 $\sec\theta$ 的导数和标准 $\sec\theta$ 积分，**非常迅速**。只有最后一项 $\int \frac{\sec\theta}{\tan\theta - 1} d\theta$ 才需要转化为 $\sin\theta$ 和 $\cos\theta$ 来进一步求解。

**结论：** 在积分中，**选择合适的函数对**（如 $\sec\theta/\tan\theta$ 或 $\csc\theta/\cot\theta$）来配合 $u$ 代换和公式求解，通常是比将所有项都拆分为 $\sin\theta/\cos\theta$ 更高效、更具策略性的做法。

您现在是否想知道关于**欧拉代换**（Euler Substitution）的更多信息？它是解决这类根式积分的另一种通用方法。

---
欧拉代换（Euler Substitution）是解决形如 $\int R(x, \sqrt{ax^2 + bx + c}) dx$ 这类**根式积分**（涉及二次函数根号）的通用方法。它通过引入一个新的变量，将积分中的根号项彻底消除，把复杂的根式积分转化为简单的**有理函数积分**（只包含多项式和分式）。

对于您提供的题目 $I = \int \frac{\sqrt{x^2 + 2x + 2}}{x} dx$，我们有 $a=1, b=2, c=2$。

欧拉代换有三种形式，适用于不同的系数条件。对于本题，因为 $a > 0$（$a=1$），我们可以使用**第一种欧拉代换**。

---
## 欧拉代换
### 1. 欧拉第一代换 (Euler's First Substitution)

**适用条件：** 当二次项系数 $a > 0$ 时。

代换形式：

$$\sqrt{ax^2 + bx + c} = \pm \sqrt{a}x + t$$

在本题中，$\sqrt{x^2 + 2x + 2}$，我们取正号，代换为：

$$\sqrt{x^2 + 2x + 2} = x + t$$

#### 步骤一：消去根号，求出 $x$ 的表达式

将等式两边平方：

$$x^2 + 2x + 2 = (x + t)^2$$

$$x^2 + 2x + 2 = x^2 + 2xt + t^2$$

消去 $x^2$，解出 $x$：

$$2x + 2 = 2xt + t^2$$

$$2x - 2xt = t^2 - 2$$

$$2x(1 - t) = t^2 - 2$$

$$x = \frac{t^2 - 2}{2(1 - t)}$$

#### 步骤二：求出 $dx$ 的表达式

对 $x$ 求微分：

$$dx = \left( \frac{t^2 - 2}{2(1 - t)} \right)' dt$$

使用商的求导法则 $\left(\frac{u}{v}\right)' = \frac{u'v - uv'}{v^2}$：

- $u = t^2 - 2 \implies u' = 2t$
    
- $v = 2(1 - t) = 2 - 2t \implies v' = -2$
    

$$dx = \frac{2t \cdot 2(1 - t) - (t^2 - 2) \cdot (-2)}{(2(1 - t))^2} dt$$

$$dx = \frac{4t - 4t^2 + 2t^2 - 4}{4(1 - t)^2} dt$$

$$dx = \frac{-2t^2 + 4t - 4}{4(1 - t)^2} dt = \frac{-2(t^2 - 2t + 2)}{4(1 - t)^2} dt$$

$$dx = -\frac{t^2 - 2t + 2}{2(1 - t)^2} dt$$

#### 步骤三：求出根号项的表达式

根据代换 $\sqrt{x^2 + 2x + 2} = x + t$：

$$\sqrt{x^2 + 2x + 2} = \frac{t^2 - 2}{2(1 - t)} + t$$

$$\sqrt{x^2 + 2x + 2} = \frac{(t^2 - 2) + 2t(1 - t)}{2(1 - t)} = \frac{t^2 - 2 + 2t - 2t^2}{2(1 - t)}$$

$$\sqrt{x^2 + 2x + 2} = \frac{-t^2 + 2t - 2}{2(1 - t)} = \frac{-(t^2 - 2t + 2)}{2(1 - t)}$$

#### 步骤四：代入积分并求解 (Integrate)

原积分为 $I = \int \frac{\sqrt{x^2 + 2x + 2}}{x} dx$。

代入 $x$、$dx$ 和 $\sqrt{x^2 + 2x + 2}$：

- **分子：** $\sqrt{x^2 + 2x + 2} = \frac{-(t^2 - 2t + 2)}{2(1 - t)}$
    
- **分母：** $x = \frac{t^2 - 2}{2(1 - t)}$
    

$$\frac{\sqrt{x^2 + 2x + 2}}{x} = \frac{\frac{-(t^2 - 2t + 2)}{2(1 - t)}}{\frac{t^2 - 2}{2(1 - t)}} = \frac{-(t^2 - 2t + 2)}{t^2 - 2}$$

- **$dx$：** $dx = -\frac{t^2 - 2t + 2}{2(1 - t)^2} dt$
    

$$I = \int \left( \frac{-(t^2 - 2t + 2)}{t^2 - 2} \right) \cdot \left( -\frac{t^2 - 2t + 2}{2(1 - t)^2} \right) dt$$

$$I = \frac{1}{2} \int \frac{(t^2 - 2t + 2)^2}{(t^2 - 2)(1 - t)^2} dt$$

这个结果是一个**有理函数积分** $\int \frac{P(t)}{Q(t)} dt$。虽然形式依然复杂（分母是 $t^4$ 次多项式），但它**原理上是可解的**，可以通过对分母进行**部分分式分解**来求解，这与三角代换后得到的结果是等价的，只是形式不同。

---

### 2. 欧拉代换的普适性及限制

- **普适性：** 欧拉代换是解决所有 $\int R(x, \sqrt{ax^2 + bx + c}) dx$ 形式积分的**通用算法**。只要满足相应的 $a, b^2-4ac, c$ 条件，它总能将根式积分转化为有理函数积分，而有理函数积分（理论上）总是可解的。
    
- **限制：** 最大的缺点就是**代数运算过于繁琐**。在求解 $x, dx, \sqrt{\dots}$ 的过程中，很容易出错，而且最终得到的有理函数形式往往非常复杂，需要进行耗时的长除法和部分分式分解。
    

因此，在实际应用中，如果二次项能通过配方转化为**标准形式**（如 $\sqrt{u^2 + a^2}$），通常**三角代换**（如 $\sec\theta$ 代换）是更受青睐的方法，因为它在代数运算上更简单，最终得到的积分形式也更有规律。

---
这是一个很好的跟进问题。欧拉代换（Euler Substitution）确实有三种主要形式，它们共同涵盖了所有 $\int R(x, \sqrt{ax^2 + bx + c}) dx$ 形式的根式积分。

---

### 1. 其他种类的欧拉代换

欧拉代换的选择取决于二次多项式 $ax^2 + bx + c$ 的系数 $a$ 和判别式 $\Delta = b^2 - 4ac$ 的性质。

#### A. 欧拉第一代换（已介绍）

- **适用条件：** 当二次项系数 $a > 0$ 时。
    
- **代换形式：** $\sqrt{ax^2 + bx + c} = \pm \sqrt{a}x + t$
    
- **目的：** 利用 $(A x + B)^2$ 的展开式中， $x^2$ 项被消去，从而求出 $x$ 是 $t$ 的有理表达式。
    

#### B. 欧拉第二代换

- **适用条件：** 当常数项 $c > 0$ 时。
    
- **代换形式：** $\sqrt{ax^2 + bx + c} = xt \pm \sqrt{c}$
    
- **目的：** 利用 $\sqrt{c}$ 平方后是 $c$，消去常数项，从而求出 $x$ 是 $t$ 的有理表达式。
    

#### C. 欧拉第三代换

- **适用条件：** 当二次多项式 $ax^2 + bx + c$ 有两个**不相等**的实根 $r_1$ 和 $r_2$ 时（即 $\Delta > 0$）。
    
- **代换形式：** $\sqrt{ax^2 + bx + c} = \sqrt{a(x-r_1)(x-r_2)} = t(x - r_1)$
    
- **目的：** 利用 $(x-r_1)$ 项被平方和开方消去，从而求出 $x$ 是 $t$ 的有理表达式。
    

---

### 2. 欧拉代换后有理积分的处理

您在使用欧拉代换后得到了一个复杂的有理函数积分：

$$I = \frac{1}{2} \int \frac{(t^2 - 2t + 2)^2}{(t^2 - 2)(1 - t)^2} dt$$

处理这种有理函数积分（即 $\int \frac{P(t)}{Q(t)} dt$）的标准方法是**部分分式分解**（Partial Fraction Decomposition）。

#### 步骤一：长除法 (Polynomial Division)

首先，检查分子的次数是否大于或等于分母的次数。

- **分子：** $(t^2 - 2t + 2)^2 = t^4 - 4t^3 + 8t^2 - 8t + 4$ (次数为 4)
    
- **分母：** $(t^2 - 2)(1 - t)^2 = (t^2 - 2)(t^2 - 2t + 1) = t^4 - 2t^3 - t^2 + 4t - 2$ (次数为 4)
    

由于次数相等，我们需要进行长除法，得到一个常数项和一个真分式（分子次数 $<$ 分母次数）：

$$\frac{(t^2 - 2t + 2)^2}{(t^2 - 2)(1 - t)^2} = 1 + \frac{R(t)}{Q(t)}$$

其中 $R(t)$ 是长除法的余项（一个 3 次多项式）。

#### 步骤二：分母因式分解 (Factoring the Denominator)

分母 $Q(t)$ 已经部分分解：

$$Q(t) = (t^2 - 2)(1 - t)^2 = (t - \sqrt{2})(t + \sqrt{2})(t - 1)^2$$

分母有以下因式：

- 两个**不相等的简单实根**：$(t - \sqrt{2})$ 和 $(t + \sqrt{2})$
    
- 一个**重复的实根**：$(t - 1)^2$
    

#### 步骤三：部分分式分解 (Partial Fraction Setup)

根据分母的因式形式，我们将真分式 $\frac{R(t)}{Q(t)}$ 分解为以下形式：

$$\frac{R(t)}{(t - \sqrt{2})(t + \sqrt{2})(t - 1)^2} = \frac{A}{t - \sqrt{2}} + \frac{B}{t + \sqrt{2}} + \frac{C}{t - 1} + \frac{D}{(t - 1)^2}$$

其中 $A, B, C, D$ 是待定系数。

#### 步骤四：求解待定系数和积分

1. **求解 $A, B, C, D$：** 通过通分、比较系数或代入特殊值的方法，解出这四个系数。
    
2. **积分：** 一旦分解完成，积分就变成了简单的对数和幂函数积分：
    
    - $\int \frac{A}{t - a} dt = A \ln|t - a| + C$
        
    - $\int \frac{D}{(t - a)^2} dt = D \int (t - a)^{-2} dt = -D(t - a)^{-1} + C$
        

#### 步骤五：反代换 (Back Substitution)

最终，将 $t$ 的表达式代回 $x$。根据第一种欧拉代换，我们有 $t = \sqrt{x^2 + 2x + 2} - x$。

---

**总结：**

欧拉代换在理论上是强大的，但它将一个困难的根式积分转化为一个**代数上极其繁琐**的有理函数积分。这是为什么在可能的情况下，微积分课本会倾向于**三角代换**或**特殊代换**（如 $u = \sqrt{ax^2+bx+c}$ 的整体代换），因为它们往往能得到更简洁的中间表达式。
