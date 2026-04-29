## Pytorch使用指南
### 0. 一切的开始：import torch
```python
import torch.nn as nn
import torch.Functional as F
```
torch.nn 里面是神经网络的各种调用方法，能够处理计算图，实现forward_prop和backward_prop
### 1. 张量（Tensor）：RL 的数据载体

所有 RL 中的数据（状态 s、动作 a、奖励 r、价值 V 等）都要转换成 `Tensor`，才能用 GPU 加速和自动求导。
##### *从numpy/列表转Tensor（RL中最常用：状态是numpy数组，转Tensor输入网络）*
```python
# 1. 从numpy/列表转Tensor（RL中最常用：状态是numpy数组，转Tensor输入网络）
state = [1.2, 3.4, -0.5]  # 单个状态（比如CartPole的4维状态，这里简化为3维）
state_tensor = torch.tensor(state, dtype=torch.float32)  # 必须用float32（RL默认精度，省显存）
state_batch = torch.tensor([[1.2,3.4,-0.5], [2.1,0.8,-1.0]], dtype=torch.float32)  # 批量状态（batch_size=2）
```

>[!NOTE]
>可以发现：
>`torch.tensor()`里面一共有4个参数
>1. `data`参数：接受 Python 列表（以及多维列表、元组、NumPy 数组、标量、其他 Tensor
>2. `dtype`数据类型
>3. `device`
>4. `requires_grad` (bool) 自动求导开关


##### RL 场景常用 dtype

| dtype           | 用途                | 原始数据示例             |
| --------------- | ----------------- | ------------------ |
| `torch.float32` | 状态（s）、奖励（r）、价值（V） | [1.2, 3.4]、1.0、2.5 |
| `torch.long`    | 离散动作（a）（索引型）      | [0, 1, 0]、2        |
| `torch.bool`    | 结束标记（done）（可选）    | [True, False]      |

##### *设备迁移（GPU/CPU，RL加速关键）
```python
# 2. 设备迁移（GPU/CPU，RL加速关键）
device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
state_tensor = state_tensor.to(device)  # 把数据移到GPU（训练大网络必用）
```

##### *维度调整（RL中网络输入必须是「批量维度在前」，即(batch_size, dim)*
```python
# 3. 维度调整（RL中网络输入必须是「批量维度在前」，即(batch_size, dim)）
state_tensor = state_tensor.unsqueeze(0)  # 单个状态加batch维度：(3,) → (1, 3)
state_batch = state_batch.squeeze(0)  # 批量维度多余时删除：(1, 2, 3) → (2, 3)
```
squeeze, unsqueeze使用规范：
(`Tensor`).squeeze(`int`) 括号里面是维度索引，删去那个维度
(`Tensor`).unsqueeze(`int`) 括号里面是维度索引，0最前，-1最后，插入一个size=1的维度

| 张量 shape   | 维度含义                  | 是否多余？               | 处理方式                          |
| ---------- | --------------------- | ------------------- | ----------------------------- |
| (1, 4)     | 1 个样本，4 维状态           | 可选（看需求）             | 要单个样本→squeeze (0)→(4,)；要批量→保留 |
| (1, 32, 4) | 1 个 “批量”，32 个样本，4 维状态 | 是（size=1 的维度不代表样本数） | squeeze(0)→(32,4)             |
| (32, 1, 4) | 32 个样本，每个样本 1×4 维     | 是（中间的 size=1 维度无用）  | squeeze(1)→(32,4)             |
| (32, 4)    | 32 个样本，4 维状态          | 否（有效批量维度）           | 保留                            |
##### *并批操作（把多个状态向量拼起来输入）*
```python
# 并批操作（把多个状态向量拼起来输入）
def sample(self, batch_size):
        """随机采样并批处理"""
        batch = np.random.choice(self.buffer, batch_size, replace=False)
        # 按类型拆分数据（每个元素是批量列表）
        states, actions, rewards, next_states, dones = zip(*batch)
        
        # 转换为张量（自动增加批量维度）
        states = torch.tensor(states, dtype=torch.float32)  
        # shape: [batch_size, state_dim]
        actions = torch.tensor(actions, dtype=torch.long)   
        # shape: [batch_size, action_dim]
        rewards = torch.tensor(rewards, dtype=torch.float32)
        # shape: [batch_size]
        next_states = torch.tensor(next_states, dtype=torch.float32)  
        # [batch_size, state_dim]
        dones = torch.tensor(dones, dtype=torch.float32)    
        # [batch_size]
        return states, actions, rewards, next_states, dones

```
>[!NOTES] 上述并批操作是DQN中Replay_Buffer的经典并批操作
>按行采样batch_size个样本
>然后`zip(*batch)`按列解包
>最后全部转化为s, a, s', r张量 `[batch_size, var_size]`

>[!INFO] Key：矩阵运算的过程完全一致！
>##### 1. 单个样本输入（`x.shape = [input_dim]`）
>计算过程：
>```plaintext
>y = W × x + b  
># 矩阵乘法：[output_dim, input_dim] × [input_dim] → [output_dim]  
># 加偏置：[output_dim] + [output_dim] → [output_dim]
>```
>#### 2. 批量样本输入（`x.shape = [batch_size, input_dim]`）
>计算过程完全相同，只是通过矩阵运算并行处理所有样本：
>```plaintext
>y = (W × x^T)^T + b  
># 步骤1：x 转置为 [input_dim, batch_size]，与 W 相乘 → [output_dim, batch_size]  
># 步骤2：结果转置为 [batch_size, output_dim]  
># 步骤3：加偏置（广播机制自动扩展 b 为 [batch_size, output_dim]）
>```


>[!WAYS]
>1. 同一个智能体连续收集 32(batch_size) 步
>2. 32(batch_size) 个并行智能体各收集 1 步 （*这里是通过多线程实现的*）

>[!Advantages]
>1. 降低优势值 $A_t$ 的估计方差，让训练更稳定
>	- TD 优势函数 $A_t = r_t + γV (s_{t+1}) - V (s_t)$ 中，$V (s_t)$ 和 $V (s_{t+1})$是网络的估计值（存在误差），$r_t$ 是环境的随机奖励（存在噪声）。
>	- 单样本 $A_t$：噪声和估计误差完全由这一步承担，导致 $A_t$ 波动极大（比如某一步随机得到 -1 奖励，$A_t$ 瞬间变负，误导策略更新）；
>	- 32 个 $A_t$ 并批：通过「批量平均梯度」抵消单样本的随机噪声和估计误差 —— 有的样本 $A_t$ 偏高，有的偏低，平均后梯度方向更接近真实值，策略更新不会 “因单个异常样本跑偏”
>2. 充分利用硬件算力，提升训练效率（GPU擅长同时并行计算，单批次反而浪费算力）

##### *禁止求导（RL中目标网络、旧策略网络以及临时权重网络的参数不能被梯度更新）*
```python
# 5. 禁止求导（RL中目标网络、旧策略网络的参数不能被梯度更新）
with torch.no_grad():
    q_target = target_net(state_batch)  # 目标网络的输出，不保存计算图
```

实际上，`with torch.no_grad()`阻止了计算图随前向传播而记录，这里好处有二：
1.  不记录计算图使得运算加快
2. 计算loss的时候，有一部分网络是只作为权重出现的，不能全部一起backward

>[!EXAMPLE]
> A2C算法里面Actor 的损失是$$loss_{Actor} = -\log p(a_t|s_t) \cdot A_t$$
>其中优势函数（TD 误差） $$A_t = TD_{target} - V_{s_t}$$
>这里的$A_t$ 是**固定权重**，不能让梯度通过 $A_t$ 回流到 Critic 网络 
>—— 否则 Actor 的梯度会受到 Critic 网络的干扰（Actor 只需要关注 “动作概率” 和 “优势” 的相关性，不需要优化优势本身）。
>所以在执行`loss_actor.backward()`的时候 $A_t$ 必须阻断！

**更加具体的，我们这样理解：**

>[!Example]
>###### 1. Actor 损失的数学目标（A2C 原本要优化的）
>$$\text{loss\_actor} = -\mathbb{E}[\log p(a_t|s_t) \cdot A_t]$$
>这里的 $A_t$ 是「固定权重」—— 它的作用是 “告诉 Actor：这个动作好不好（优势为正 / 负）”，Actor 只需要调整自己的参数 $W_a$，让 “优势正的动作概率变大，优势负的动作概率变小”。此时 Actor 损失对自身参数的梯度应该是：$$\frac{\partial \text{loss\_actor}}{\partial W_a} = -\mathbb{E}\left[ A_t \cdot \frac{\partial \log p(a_t|s_t)}{\partial W_a} \right] $$
>梯度只和 Actor 的参数 $W_a$ 有关，和 Critic 的参数 $W_c$ 无关。
>###### 2. 不阻断 $A_t$ 梯度时，实际的梯度流向
>当 $A_t$ 带有梯度（即 $V(s_t)$ 的梯度），Actor 损失对参数的梯度会变成两部分（链式法则）：
>正常项：A2C 想要的：$$\frac{\partial W_a}{\partial loss\_actor}​=−\mathbb{E} \left[A_t\cdot\frac{\partial W_a}{\partial \log p(a_t|s_t)}\right]$$
>干扰项：A2C 不想要的：$$\frac{\partial W_a}{\partial loss\_actor}​=−\mathbb{E} \left[A_t\cdot\frac{\partial W_a}{\partial \log p(a_t|s_t)}\right]$$
>###### 关键问题：这个 “干扰项” 会做什么？
>- 因为 $TD_{\text{target}}$ 无梯度，则：$$\frac{\partial A_t}{\partial W_c} = \frac{\partial (TD_{\text{target}} - V(s_t))}{\partial W_c} = -\frac{\partial V(s_t)}{\partial W_c}$$
>- 所以干扰项本质是：$$\mathbb{E}\left[ \log p(a_t|s_t) \cdot \frac{\partial V(s_t)}{\partial W_c} \right]$$
>- 当你执行 `total_loss.backward()` 并调用 `optimizer.step()`（优化器包含 Actor 和 Critic 的参数）时，这个干扰项会**让 Critic 的参数 $W_c$ 朝着 “最大化 $\log p(a_t|s_t) \cdot V(s_t)$” 的方向更新**—— 这完全违背了 Critic 的设计目标！
>
>Critic 的原本目标是：**最小化价值预测误差($V(s_t)$ 接近 $TD_{\text{target}}$）**，而不是 “迎合 Actor 的动作概率”。现在 Critic 不仅要学习价值预测，还要被 Actor 的动作概率 “绑架”，导致两个网络互相干扰：
>- Actor 为了降低自己的损失，可能会让 Critic 预测的 $V(s_t)$ 变大（从而让 $A_t$ 变大），而不是优化动作策略；
>- Critic 为了迎合 Actor 的 $\log p(a_t|s_t)$，可能会扭曲价值预测（比如明明是坏动作，却预测高价值），导致优势函数 $A_t$失去意义。


### 2. 神经网络（nn.Module）：RL 的核心模型

DQN 的 Q 网络、A2C 的策略网络 / 价值网络、PPO 的 Actor-Critic 网络，都用`nn.Module`构建。

#### 模板（RL 通用网络结构）：

```python
class RLNetwork(nn.Module):
    def __init__(self, state_dim, action_dim, hidden_dim=64):
        super(RLNetwork, self).__init__()
        # 1. 定义网络层（RL常用：全连接层nn.Linear + 激活函数ReLU）
        self.fc1 = nn.Linear(state_dim, hidden_dim)  # 输入层→隐藏层
        self.fc2 = nn.Linear(hidden_dim, hidden_dim) # 隐藏层→隐藏层
        # 2. 输出层（根据算法调整）
        self.output_layer = nn.Linear(hidden_dim, action_dim)  
        # 离散动作（DQN/A2C Actor）：输出动作价值/概率
    
    def forward(self, x):
        # 前向传播（输入x是(batch_size, state_dim)的Tensor）
        x = torch.relu(self.fc1(x))  # 隐藏层激活（RL几乎不用sigmoid，避免梯度消失）
        x = torch.relu(self.fc2(x))
        out = self.output_layer(x)
        return out

# 实例化（以CartPole为例：state_dim=4，action_dim=2）
net = RLNetwork(state_dim=4, action_dim=2).to(device)  # 移到GPU
```

>[!KEY]
>可以看见，创建实例的时候传入的参数是`__init__`函数里面的参数
>该网络子类继承了`nn.Module`，所以自然继承了父类中的属性和方法
>最常用的莫过于`net.parameters()`，用于优化器的声明：
>```python
optimizer = torch.optim.Adam(net.parameters(), lr=1e-3)
>```


### 3. 自动求导（autograd）：RL 的梯度更新核心

RL 算法的本质是「最小化损失函数」，PyTorch 的`autograd`会自动计算梯度，无需手动推导。

#### 核心流程（RL 参数更新通用模板）：
```python
# 1. 定义优化器（RL常用Adam，学习率1e-3~1e-4）
optimizer = torch.optim.Adam(net.parameters(), lr=1e-3)

# 2. 前向传播：计算预测值（比如DQN的Q预测）
state_batch = torch.randn(32, 4).to(device)  # 批量状态（batch_size=32）
q_pred = net(state_batch)  # (32, 2)：每个样本的2个动作Q值

# 3. 定义损失函数（根据算法调整）
q_target = torch.randn(32, 2).to(device)  # DQN的目标Q值（由目标网络计算）
loss_fn = nn.MSELoss()  # 均方误差（DQN/Critic网络常用）
loss = loss_fn(q_pred, q_target)  # 计算损失

# 4. 反向传播+参数更新（RL的核心步骤，必须严格顺序）
optimizer.zero_grad()  # 清空上一轮梯度（必写！否则梯度累积）
loss.backward()  # 自动计算所有可训练参数的梯度
optimizer.step()  # 用梯度更新参数
```
>[!NOTE]
>这里有一个很奇怪的操作：
>```python
>optimizer.zero_grad()  # 清空上一轮梯度（必写！否则梯度累积）
>```
>梯度累积的本质是：**PyTorch 中，参数的梯度（`param.grad`）会「自动叠加」，而非每次前向传播后重置**。如果不手动清空，下一轮的梯度会和上一轮的梯度相加，这就是「梯度累积」。
>
>我们可以回忆一下整个流程：
>第一步：$\text{net(batch)}$`__call__`函数调用了`forward`前向传播同时保存了计算图 
>第二步：计算 $\text{Loss: Tensor类型}$ 
>第三步：$\text{Loss.backward()}$
>
>首先，Adam优化器的历史与计算图的梯度历史不相关！
>其次，数学推导可知，**小批次梯度累加和 = 大批次梯度**
>
>这意味着如果不清空梯度，相当于**每次使用了一个巨型批次优化模型**
>更可怕的是，**早期批次的梯度会一直 “污染” 当前轮的梯度**，这显然不是我们想要的
>而清空梯度并不会影响Adam利用之前的动量和速度跳出局部最优
>所以第三步会有三个操作顺流而下
>```python
>optimizer.zero_grad()  # 清空上一批次梯度
>loss.backward()  # 反向传播更新梯度和权重
>optimizer.step()  # 用梯度更新Adam的动量,速度,以及网络的所有权重
>```

###### 从初始状态开始进行一批次学习时各步骤的状态变化：

| 操作             | param.grad（g_t） | Adam 动量（m_t） | Adam 二阶矩（v_t） | 网络权重（w_t）  |
| -------------- | --------------- | ------------ | ------------- | ---------- |
| 初始化后           | None（无梯度）       | 0（初始状态）      | 0（初始状态）       | 随机初始值      |
| zero_grad () 后 | 0（清空）           | 0（未变）        | 0（未变）         | 初始值（未变）    |
| backward () 后  | g_t（当前梯度）       | 0（未变）        | 0（未变）         | 初始值（未变）    |
| step () 后      | g_t（仍保留）        | 新 m_t（更新后）   | 新 v_t（更新后）    | 新 w_t（更新后） |
### 4. 模型保存与加载（RL 断点续训）

训练 RL 算法通常需要很久，必须会保存 / 加载模型，避免中途中断前功尽弃。
```python
# 保存模型（推荐保存.state_dict()，灵活度高）
torch.save({
    'model_state_dict': net.state_dict(),  # 网络参数
    'optimizer_state_dict': optimizer.state_dict(),  # 优化器参数（续训必备）
    'epoch': 100,  # 当前迭代次数
}, 'rl_model.pth')

# 加载模型
checkpoint = torch.load('rl_model.pth', map_location=device)  # map_location自动适配GPU/CPU
net.load_state_dict(checkpoint['model_state_dict'])
optimizer.load_state_dict(checkpoint['optimizer_state_dict'])
start_epoch = checkpoint['epoch']

# 加载后继续训练/测试
net.train()  # 训练模式（启用Dropout/BatchNorm）
# net.eval()  # 测试模式（禁用Dropout/BatchNorm，必用！）
```


### 5. 一些功能函数的使用

#### (1)怎么实现Tensor类型转其他非Tensor类型？
##### `Tensor(single)`.item()
这个函数的应用对象为单元素标量张量
用途：把Tensor转换为python原生类型

|张量输入|`item()` 输出|Python 类型|
|---|---|---|
|`tensor(0)`|`0`|`int`|
|`tensor(1.2)`|`1.2`|`float`|
|`tensor([3])`|`3`|`int`|
>[!WARNING]
>`.item()`不支持非标量张量的转换！

##### `Tensor(Any)`.tolist()
任意Tensor类型的变量都可以转换为相应列表，
转换前后维度保持一致

##### `Tensor(Any)`.detach().cpu().numpy()
任意Tensor类型变量转换为相应numpy类型数据
>[!Warning]
>detach截断梯度，cpu把数据移动回cpu，numpy进行数据转换
>因为numpy类型只能在cpu上处理，如果Tensor在GPU上会报错
>如果想要进一步转换的话numpy类型也有.item() .tolist()方法

### (2)`torch`, `nn`, `F`之下都有激活函数的实现，怎么选用？
#### 实战建议：可以混用，但要 “有规则地混”

混用本身没问题，但建议遵循以下规则，避免问题：

1. **同一网络层，统一风格**：
    - 用 `nn.Sequential` 搭建的网络，全程用 `nn` 类形式（整洁统一）；
    - 自定义 `forward` 传播时，全程用 `F` 函数形式（灵活高效）；
    - 非网络场景的张量运算，用 `torch` 底层函数（明确区分 “网络激活” 和 “张量运算”）。
2. **带参数的激活函数，只用 `nn` 类形式**：
    如 `PReLU`、`GELU(approximate='tanh')` 等带参数（可训练或配置参数）的激活函数，一律用 `nn` 类形式，避免用函数形式手动传参。
3. **统一 `inplace` 参数**：
    要么全程不用 `inplace`（默认 `False`），要么统一设置为 `True`，不要部分激活函数用 `inplace=True`，部分用 `False`。
4. **避免冗余代码**：
    不在 `__init__` 中实例化激活函数后又不用，也不重复写相同的激活逻辑（比如 `nn.LeakyReLU` 和 `F.leaky_relu` 选一个即可）。

## A2C算法笔记
一般来说`logits`是指模型还未归一化(softmax)的时候输出的原始分数

##### 预测动作
```python
def take_action(self, state):
	state = torch.tensor([state], dtype=torch.float).to(self.device) # 取状态
    probs = self.actor(state) # 得到概率分布
    action_dist = torch.distributions.Categorical(probs) # 创建分类分布实例
    action = action_dist.sample() # 采样
    return action.item() # 返回选择的动作
```
这里的`torch.distributions.Categorical(probs)`创建的实例是一个工具
里面的常用成员函数有`.log_prob()`（支持批量取对数，保留计算图）以及`.sample()`（支持随机采样，这个功能与rand.choice类似）

##### 更新两个网络
```python
def update(self):
    # 时序差分目标
    td_target = rewards + self.gamma * self.critic(next_states) * (1 -dones)
    td_delta = td_target - self.critic(states)  # 时序差分误差
    log_probs = torch.log(self.actor(states).gather(1, actions))
    actor_loss = torch.mean(-log_probs * td_delta.detach())
    # 均方误差损失函数
    critic_loss = torch.mean(
        F.mse_loss(self.critic(states), td_target.detach()))
    self.actor_optimizer.zero_grad()
    self.critic_optimizer.zero_grad()
    actor_loss.backward()  # 计算策略网络的梯度
    critic_loss.backward()  # 计算价值网络的梯度
    self.actor_optimizer.step()  # 更新策略网络的参数
    self.critic_optimizer.step()  # 更新价值网络的参数
```