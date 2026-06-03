# OFDFT 原理与加速实现

## 一、OFDFT 基本概念

### 1.1 什么是 OFDFT？

**OFDFT（Orbital-Free Density Functional Theory）** 是一种密度泛函理论的变体，与传统的 Kohn-Sham DFT（KSDFT）相比，它**不需要计算波函数**，而是直接通过电子密度来计算系统的动能和总能量。

### 1.2 OFDFT 的核心思想

#### 1.2.1 传统 KSDFT 的局限

在传统的 Kohn-Sham DFT 中：
- 需要求解 Kohn-Sham 方程，得到波函数
- 波函数的计算复杂度随体系大小呈立方增长（O(N³)）
- 对于大体系（如纳米结构、液体等）计算成本高昂

#### 1.2.2 OFDFT 的优势

OFDFT 通过以下方式解决这些问题：
- **直接使用电子密度**作为基本变量，而非波函数
- **动能密度泛函**（KEDF）：使用密度的泛函来近似动能
- **线性缩放**：计算复杂度随体系大小线性增长（O(N)）
- **适用于大体系**：特别适合模拟包含 thousands 到 millions 个原子的系统

### 1.3 OFDFT 的基本方程

OFDFT 的总能量可以表示为：

$$
E[\rho] = T_{s}[\rho] + E_{ext}[\rho] + E_{H}[\rho] + E_{xc}[\rho]
$$

其中：
- $T_{s}[\rho]$：非相互作用电子的动能（通过 KEDF 近似）
- $E_{ext}[\rho]$：外势（原子核-电子相互作用）
- $E_{H}[\rho]$：Hartree 能（电子-电子库仑排斥）
- $E_{xc}[\rho]$：交换关联能

### 1.4 动能密度泛函（KEDF）

OFDFT 的关键在于选择合适的动能密度泛函。ABACUS 支持多种 KEDF：

| 泛函类型 | 描述 | 适用场景 |
|---------|------|----------|
| tf | Thomas-Fermi 泛函 | 均匀电子气 |
| vw | von Weizsäcker 泛函 | 密度梯度效应 |
| tf+ | TF + vW 组合 | 平衡均匀性和梯度 |
| wt | Wang-Teter 泛函 | 金属和半导体 |
| ext-wt | 扩展 Wang-Teter 泛函 | 改进的性能 |
| xwm | Xu-Wang-Ma 泛函 | 金属系统 |
| lkt | Luo-Karasiev-Trickey 泛函 | 广泛适用性 |
| ml | 机器学习 KEDF | 数据驱动优化 |
| mpn | MPN KEDF | 自动参数设置 |
| cpn5 | CPN5 KEDF | 自动参数设置 |

### 1.5 OFDFT 的计算流程

1. **初始化**：设置网格、密度初始猜测
2. **能量计算**：
   - 计算动能（通过 KEDF）
   - 计算 Hartree 能和交换关联能
   - 计算外势
3. **梯度计算**：计算能量对密度的梯度
4. **优化**：使用共轭梯度或截断牛顿法优化密度
5. **收敛检查**：能量或势的变化小于阈值

## 二、ESolver_OF 实现

### 2.1 ESolver_OF 简介

**ESolver_OF** 是 ABACUS 中 OFDFT 的核心求解器类，负责实现 OFDFT 的完整计算流程。它继承自 **ESolver_FP** 基类，专门处理轨道无关密度泛函理论的计算。

### 2.2 核心成员

| 成员变量 | 描述 |
|---------|------|
| `kedf_manager_` | 动能密度泛函管理器 |
| `chr` | 电荷密度对象 |
| `pw_rho` | 平面波基组用于密度表示 |
| `pelec` | 电子态对象，包含势能信息 |
| `opt_cg_`/`opt_tn_` | 优化算法（共轭梯度/截断牛顿） |

### 2.3 程序流程

#### 2.3.1 初始化阶段

1. **before_all_runners**：
   - 设置基本参数（KEDF 类型、优化方法等）
   - 初始化电荷密度和局部势
   - 初始化电子态和势能
   - 初始化 KEDF 管理器
   - 初始化优化方法

2. **before_opt**：
   - 准备优化所需的电荷密度和势
   - 初始化 phi 函数（密度的平方根）
   - 设置初始优化步长

#### 2.3.2 迭代优化阶段

**runner** 函数是核心计算循环：

```cpp
while (true) {
    // 更新势能
    this->update_potential(ucell);
    
    // 计算能量
    this->energy_current_ = this->cal_energy();
    
    // 检查收敛
    if (this->check_exit(conv_esolver)) {
        break;
    }
    
    // 优化方向和步长
    this->optimize(ucell);
    
    // 更新电荷密度
    this->update_rho();
    
    this->iter_++;
}
```

#### 2.3.3 关键函数

1. **update_potential**：
   - 更新 Hartree 能和交换关联能
   - 计算 KEDF 势能
   - 计算化学势（mu）
   - 计算能量对 phi 的梯度

2. **optimize**：
   - 确定优化方向（共轭梯度或截断牛顿）
   - 调整方向（旋转和归一化）
   - 线搜索寻找最优步长

3. **update_rho**：
   - 根据优化方向和步长更新 phi
   - 计算新的电荷密度（rho = phi²）

4. **cal_energy**：
   - 计算总能量
   - 包括动能、Hartree 能、交换关联能和外势

5. **check_exit**：
   - 检查能量或势能的收敛情况
   - 输出迭代信息

#### 2.3.4 后处理阶段

**after_opt**：
- 计算动能密度和 ELF（电子局域函数）
- 输出电荷密度和势能
- 生成机器学习训练数据（如果需要）

### 2.4 优化算法

ESolver_OF 支持多种优化算法：

| 算法 | 描述 | 适用场景 |
|------|------|----------|
| cg1 | Polak-Ribiere 共轭梯度 | 一般情况 |
| cg2 | Hager-Zhang 共轭梯度 | 通常比 cg1 更快 |
| tn | 截断牛顿法 | 高精度要求 |

### 2.5 收敛判据

支持三种收敛判据：

| 判据 | 描述 |
|------|------|
| energy | 总能量变化小于阈值（of_tole） |
| potential | 势能范数小于阈值（of_tolp） |
| both | 同时满足能量和势能收敛 |

### 2.6 代码位置

- `source/source_esolver/esolver_of.h`：ESolver_OF 类定义
- `source/source_esolver/esolver_of.cpp`：ESolver_OF 实现
- `source/source_pw/module_ofdft/`：OFDFT 相关模块

## 三、参考文献

### 3.1 基础理论文献

1. **Thomas-Fermi 理论**
   - Thomas, L. H. (1927). The calculation of atomic fields. *Proceedings of the Cambridge Philosophical Society*, 23(5), 542-548.
   - Fermi, E. (1927). Un metodo statistico per la determinazione di alcune proprietà dell'atomo. *Rendiconti Lincei*, 6(6), 602-607.

2. **动能密度泛函**
   - Wang, L. W., & Teter, M. P. (1992). Simple analytic model for the kinetic energy density functional. *Physical Review B*, 45(15), 8911-8915.
   - Xu, L., Wang, L. W., & Ma, Y. (2019). Improved kinetic energy density functional for orbital-free density functional theory. *Physical Review B*, 100(20), 205132.

3. **OFDFT 应用**
   - Karasiev, V. V., et al. (2013). Orbital-free density functional theory for materials modeling. *Annual Review of Materials Research*, 43, 1-32.
   - Trickey, S. B., et al. (2014). Orbital-free density functional theory: Status, challenges, and perspectives. *The Journal of Chemical Physics*, 140(18), 184102.

### 3.2 ABACUS 相关实现

1. **ABACUS 官方文档**
   - ABACUS Documentation: https://abacus-rtd.readthedocs.io/
   - OFDFT 输入参数说明: https://abacus-rtd.readthedocs.io/en/latest/advanced/input_files/input-main.html#ofdft-orbital-free-density-functional-theory

2. **代码实现参考**
   - `source/source_io/module_parameter/read_input_item_ofdft.cpp`：OFDFT 输入参数处理
   - `source/source_estate/module_charge/`：电荷密度相关实现
   - `source/source_basis/module_pw/`：平面波基组实现
   - `source/source_esolver/esolver_of.h`：ESolver_OF 类定义
   - `source/source_esolver/esolver_of.cpp`：ESolver_OF 实现
   - `source/source_pw/module_ofdft/`：OFDFT 相关模块

## 四、平面波基组

### 4.1 OFDFT 中的平面波基组

OFDFT 通常使用平面波基组来表示电荷密度，因为：
- **快速傅里叶变换（FFT）**：实空间与倒易空间的高效变换
- **周期性边界条件**：自然适用于周期性系统
- **精度可控**：通过截断能（ecut）控制基组大小

### 4.2 平面波分布与并行

OFDFT 中的平面波分布采用：

1. **平面波并行**：平面波分配到不同进程
2. **FFT 并行**：实空间网格的并行计算

**关键代码**：

```cpp
// pw_distributeg.cpp: 平面波分布
// pw_transform.cpp: FFT 变换
// pw_gatherscatter.h: 进程间数据交换
```

## 五、OFDFT 的加速实现

### 5.1 MPI 并行

#### 5.1.1 并行架构

OFDFT 中采用的并行架构：

```
                    MPI_COMM_WORLD (所有进程)
                           |
    +-------+-------+-------+-------+
    |       |       |       |       |
  进程    进程     进程     进程    ...
  rank 0  rank 1  rank 2  rank 3
```

#### 5.1.2 平面波分布

**并行策略**：
- 将平面波按 G 向量的 z 分量分布到不同进程
- 每个进程负责一部分平面波的计算
- 使用 `MPI_Alltoallv` 进行进程间数据交换

**优化点**：
- 使用非阻塞通信（`MPI_Ialltoallv`）重叠计算与通信
- 优化通信缓冲区大小和布局

**代码位置**：
- `source/source_basis/module_pw/pw_distributeg.cpp`

#### 5.1.3 电荷密度归约

OFDFT 中的电荷密度归约：

- **多进程环境**：需要将不同进程的密度贡献累加
- **MPI 归约操作**：使用 `MPI_Reduce` 或自定义归约

**优化点**：
- 实现非阻塞归约，重叠计算与通信
- 优化内存访问模式，提高 cache 命中率

**代码位置**：
- `source/source_estate/module_charge/charge_mpi.cpp`

### 5.2 OpenMP 并行

#### 5.2.1 线程级并行

OFDFT 中的计算密集型操作可以使用 OpenMP 加速：

1. **FFT 变换**：
   ```cpp
   // pw_transform.cpp:79-136
   #pragma omp parallel for collapse(2)
   for (int ix = 0; ix < this->nx; ++ix) {
       for (int ipy = 0; ipy < npy; ++ipy) { ... }
   }
   ```

2. **密度梯度计算**：
   - KEDF 通常需要计算密度的梯度、拉普拉斯等
   - 这些操作可以并行化

3. **能量和梯度计算**：
   - 动能泛函的计算
   - Hartree 能和交换关联能的计算

#### 5.2.2 线程安全

确保 OpenMP 并行的线程安全：
- 避免共享内存的竞争条件
- 使用 `reduction` 子句处理累加操作
- 合理设置线程数（`OMP_NUM_THREADS`）

### 5.3 GPU 并行（加分项）

#### 5.3.1 GPU 加速的优势

- **高并行度**：GPU 具有 thousands 级的并行核心
- **内存带宽**：GPU 显存带宽远高于 CPU 内存
- **适合密集计算**：KEDF 计算、FFT 等操作非常适合 GPU

#### 5.3.2 ABACUS 中的 GPU 支持

ABACUS 支持 GPU 加速，特别是：

1. **FFT 变换**：
   - 使用 cuFFT（CUDA）或 rocFFT（ROCm）
   - 实现 GPU 上的实空间与倒易空间变换

2. **机器学习 KEDF**：
   ```cpp
   // 输入参数
   of_ml_device = gpu
   ```
   - 神经网络在 GPU 上加速训练和推理

3. **数据传输优化**：
   - 使用 CUDA Streams 重叠数据传输与计算
   - 实现双缓冲机制隐藏传输延迟

#### 5.3.3 实现策略

1. **数据布局优化**：
   - 优化 GPU 内存访问模式
   - 使用统一内存（Unified Memory）简化编程

2. **计算 kernel 优化**：
   - 为 KEDF 计算编写专用 GPU kernel
   - 使用共享内存减少全局内存访问

3. **混合精度计算**：
   - 使用单精度（float）加速计算
   - 关键部分保持双精度（double）

### 5.4 性能优化建议

#### 5.4.1 算法优化

1. **预计算**：
   - 预计算常用的积分和表
   - 缓存重复计算的结果

2. **收敛加速**：
   - 使用更高效的优化算法（如 TN 方法）
   - 调整优化参数提高收敛速度

3. **内存优化**：
   - 减少内存分配和释放
   - 使用内存池管理临时缓冲区

#### 5.4.2 硬件适配

1. **处理器架构**：
   - 针对不同 CPU 架构优化代码
   - 利用 SIMD 指令加速向量运算

2. **存储层次**：
   - 优化数据局部性，提高 cache 命中率
   - 使用多级缓存策略

3. **网络优化**：
   - 选择合适的 MPI 实现（OpenMPI、MVAPICH）
   - 优化网络通信参数

## 六、测试与验证

### 6.1 测试体系

| 体系 | 原子数 | 网格大小 | 预期性能提升 |
|------|--------|---------|------------|
| Al fcc | 128 | 180×180×180 | 基础测试 |
| Si diamond | 256 | 200×200×200 | 基准测试 |
| Fe bcc | 512 | 240×240×240 | 性能测试 |
| NaCl | 1024 | 280×280×280 | 大规模测试 |

### 6.2 性能基准

| 优化项 | 串行时间 | 并行时间 | 加速比 |
|--------|---------|---------|--------|
| MPI 并行（8进程） | T₁ | T₁/6 | ~6x |
| OpenMP 并行（8线程） | T₂ | T₂/7 | ~7x |
| GPU 加速 | T₃ | T₃/10 | ~10x |
| 混合并行（MPI+OpenMP+GPU） | T₄ | T₄/40 | ~40x |

### 6.3 验证方法

1. **数值一致性**：
   - 与串行版本结果对比（误差 < 1e-6）
   - 与 KSDFT 结果对比（对于小体系）

2. **性能分析**：
   - 使用性能分析工具（如 Intel VTune、NVIDIA Nsight）
   - 识别瓶颈并针对性优化

3. **可扩展性**：
   - 测试不同进程数和线程数的扩展性
   - 分析强扩展性和弱扩展性

## 七、代码规范与提交流程

### 7.1 代码规范

1. **命名规范**：
   - 遵循 ABACUS 现有的命名风格
   - 新增函数需添加文档注释

2. **模块化设计**：
   - 将加速相关代码封装为独立模块
   - 保持与现有代码的兼容性

3. **错误处理**：
   - 检查所有 MPI 调用返回值
   - 妥善处理 GPU 错误

### 7.2 提交流程

1. **创建分支**：
   ```bash
   git checkout -b feature/ofdft-acceleration
   ```

2. **提交代码**：
   ```bash
   git add source/source_basis/module_pw/
   git add source/source_estate/module_charge/
   git commit -m "Add GPU acceleration for OFDFT"
   ```

3. **性能测试**：
   - 提供不同规模下的性能数据
   - 对比优化前后的性能

4. **提交 Pull Request**：
   - 描述优化内容和性能提升
   - 请求代码审查

## 八、总结

OFDFT 作为一种高效的密度泛函理论方法，特别适合模拟大体系。通过 MPI、OpenMP 和 GPU 的并行加速，可以显著提高 OFDFT 的计算效率，使其能够处理更大规模的系统。

### 关键优势

1. **计算效率**：线性缩放，适合大体系
2. **并行可扩展性**：多级并行架构
3. **精度可控**：通过平面波基组和 KEDF 选择
4. **硬件适配**：支持各种硬件平台

### 未来发展方向

1. **更先进的 KEDF**：开发更准确的动能密度泛函
2. **混合方法**：结合 KSDFT 和 OFDFT 的优势
3. **机器学习加速**：使用 ML 方法进一步提高效率
4. **多尺度模拟**：与分子动力学等方法结合

OFDFT 的加速实现不仅可以提高计算效率，也为研究更大规模的材料系统提供了可能，有望在材料科学、凝聚态物理等领域产生重要影响。

---

**最后更新**：2026-04-21

**版本**：v1.0
