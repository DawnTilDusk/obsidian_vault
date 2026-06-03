# 格点积分的并行优化与数据局部性提升

## 大作业说明

---

## 一、背景介绍

### 0.1 格点积分基础

#### 0.1.1 什么是格点积分？

在密度泛函理论（DFT）计算中，格点积分是指在实空间网格上进行的数值积分，主要用于：

- **电荷密度计算**：$ho(\mathbf{r}) = \sum_{n,k} f_{nk} |\psi_{nk}(\mathbf{r})|^2$
- **势能计算**：交换关联势、哈特利势等的网格积分
- **矩阵元计算**：原子轨道与平面波之间的重叠积分
- **物理量提取**：能带、态密度等物理量的计算

格点积分的计算精度和效率直接影响整个DFT计算的性能和准确性。

#### 0.1.2 格点积分的计算挑战

格点积分面临以下挑战：

1. **计算复杂度高**：三维网格上的积分通常涉及 $O(N^3)$ 的计算量，其中 $N$ 是网格点数
2. **内存访问模式差**：不同的积分算法可能导致不连续的内存访问
3. **数据依赖性**：某些积分操作存在数据依赖，难以并行化
4. **并行扩展性差**：在大规模并行计算中，通信开销可能成为瓶颈

#### 0.1.3 格点积分在ABACUS中的实现

ABACUS中格点积分主要通过以下模块实现：

- `module_gint`：格点积分核心模块
- `module_pw`：平面波基组相关的积分计算
- `module_charge`：电荷密度相关的积分计算
- `module_hamilt_gint`：哈密顿量格点积分

### 1.1 问题由来

在ABACUS的格点积分计算中，存在以下性能瓶颈：

1. **数据重排开销**：格点积分需要在不同的网格布局之间进行数据重排
2. **内存访问模式**：三维网格的内存访问模式不连续，导致cache命中率低
3. **并行效率**：现有并行实现存在通信开销大、负载不均衡等问题
4. **计算与通信重叠**：未充分利用非阻塞通信来重叠计算与通信

### 1.2 现有代码结构

#### 1.2.1 格点积分核心类

```cpp
// 格点积分核心类
template <typename FPTYPE, typename Device>
class GInt {
public:
    // 格点积分计算
    void cal_ints(const int& nks, const int& npwx, const int& nbands,
                 const std::complex<FPTYPE>* psi, const double* occ, double* rho);
    
    // 数据重排
    void rearrange_data(const FPTYPE* input, FPTYPE* output, const int& grid_size);
    
private:
    // 网格信息
    int nx, ny, nz;     // 网格大小
    int nrxx;           // 总网格点数
    
    // 并行信息
    int nproc_in_pool;  // pool内进程数
    int rank_in_pool;   // pool内rank
    
    // 设备相关
    Device* device;     // 计算设备（CPU/GPU）
};
```

#### 1.2.2 格点积分计算流程

```
GInt::cal_ints()
    ↓
数据准备与分发
    ↓
循环k点和能带
    ↓
计算波函数模平方 |ψ|²
    ↓
按权重累加得到电荷密度 ρ
    ↓
数据重排与归约
    ↓
输出结果
```

#### 1.2.3 现有实现的性能瓶颈

| 瓶颈类型 | 具体位置 | 问题描述 |
|---------|---------|----------|
| **数据重排** | `rearrange_data` | 三维网格数据重排，内存访问不连续 |
| **并行计算** | `cal_ints` | 负载不均衡，通信开销大 |
| **内存访问** | 波函数访问 | 波函数存储格式导致缓存不友好 |
| **计算密度** | 浮点计算 | 未充分利用SIMD指令 |

---

## 二、题目：格点积分的并行优化与数据局部性提升

**难度**：⭐⭐⭐

### 2.1 题目描述

优化ABACUS中格点积分的计算效率，重点解决数据重排和内存访问模式问题，同时提升并行计算效率。

### 2.2 现有代码位置

- `source/source_lcao/module_gint/gint.cpp` - 格点积分核心实现
- `source/source_lcao/module_gint/gint_tools.h` - 工具函数
- `source/source_lcao/module_gint/gint_kernels.h` - 计算内核

### 2.3 具体要求

#### 2.3.1 数据重排优化

1. **内存访问模式分析**
   - 分析 `rearrange_data` 函数中的内存访问模式
   - 识别缓存不命中的原因
   - 设计更优的数据布局

2. **数据重排算法优化**
   - 实现分块（tiling）策略减少cache miss
   - 使用OpenMP并行化数据重排
   - 优化循环顺序提高数据局部性

3. **性能测试**
   - 测试不同分块大小的性能
   - 分析cache命中率提升
   - 对比优化前后的性能

#### 2.3.2 并行计算优化

1. **负载均衡分析**
   - 分析现有并行实现的负载分布
   - 识别负载不均衡的原因

2. **MPI通信优化**
   - 使用非阻塞通信（`MPI_Isend`/`MPI_Irecv`）
   - 实现计算与通信重叠
   - 优化数据分发策略

3. **OpenMP并行化**
   - 优化OpenMP线程分配
   - 减少线程同步开销
   - 实现多级并行（MPI+OpenMP）

#### 2.3.3 SIMD向量化

1. **向量化分析**
   - 识别可向量化的计算区域
   - 分析数据对齐情况

2. **SIMD实现**
   - 使用 `#pragma omp simd` 指导编译器向量化
   - 或使用Intrinsics直接编写向量化代码
   - 确保数据对齐以提高向量化效率

3. **性能评估**
   - 测试不同SIMD宽度的性能（AVX2/AVX512）
   - 分析向量化带来的加速比

#### 2.3.4 数据布局优化

1. **存储格式设计**
   - 设计更有利于缓存的数据存储格式
   - 考虑不同计算模式的存储需求

2. **实现方案**
   - 修改数据结构支持新的存储格式
   - 实现格式转换函数
   - 确保与现有接口兼容

3. **测试验证**
   - 验证新存储格式的正确性
   - 测试内存访问性能提升

### 2.4 单元测试要求

1. **正确性验证**
   - 编写测试验证优化后结果与原实现一致
   - 测试不同网格大小的边界情况
   - 验证并行计算的数值一致性

2. **性能测试**
   - 编写性能测试脚本
   - 测试不同进程数和线程数的性能
   - 生成性能报告

3. **代码质量**
   - 遵循项目代码规范
   - 添加必要的注释和文档
   - 确保代码可维护性

### 2.5 代码重构（加分项）

1. **模块化设计**
   - 将格点积分算法抽象为独立模块
   - 设计可插拔的计算内核接口

2. **设备抽象**
   - 实现设备无关的接口
   - 支持CPU和GPU计算

3. **性能预测**
   - 设计性能预测模型
   - 实现自适应优化策略

---

## 三、测试环境与基准数据

### 3.1 推荐测试体系

| 体系 | 原子数 | 网格大小 | 预计计算量 | 推荐测试规模 |
|------|--------|---------|-----------|-------------|
| Al fcc | 32 | 64×64×64 | ~260k 网格点 | 初级测试 |
| Si diamond | 64 | 80×80×80 | ~512k 网格点 | 基准测试 |
| Fe bcc | 128 | 96×96×96 | ~884k 网格点 | 性能测试 |
| TiO₂ rutile | 192 | 112×112×112 | ~1.4M 网格点 | 大规模测试 |

### 3.2 性能基准

| 优化项 | 当前时间 | 目标时间 | 最低加速比 |
|--------|---------|---------|-----------|
| 数据重排 | T₁ | T₁/3 | 3x |
| 并行计算 | T₂ | T₂/4 | 4x |
| SIMD向量化 | T₃ | T₃/2 | 2x |
| 端到端性能 | T_total | T_total/5 | 5x |

### 3.3 测试脚本参考

```bash
#!/bin/bash
# benchmark_grid_integration.sh - 格点积分性能测试

export OMP_NUM_THREADS=8
export MKL_NUM_THREADS=8

for nproc in 1 2 4 8 16; do
    for nthread in 1 2 4 8; do
        echo "Testing: nproc=$nproc, nthread=$nthread"
        export OMP_NUM_THREADS=$nthread
        mpirun -np $nproc ./abacus INPUT > log_p${nproc}_t${nthread}.out 2>&1
        grep "grid integration" log_p${nproc}_t${nthread}.out | tail -1
    done
done
```

---

## 四、代码规范与提交流程

### 4.1 代码规范

1. **命名规范**
   - 遵循项目现有的命名风格
   - 新增函数需添加文档注释

2. **模块化设计**
   - 独立功能封装为独立函数/类
   - 便于单元测试

3. **错误处理**
   - 检查所有 MPI 调用返回值
   - 妥善处理异常情况

### 4.2 提交流程

1. **Fork 仓库**
   - Fork ABACUS 仓库到你自己的 GitHub 账户

2. **创建分支**
   ```bash
   git checkout -b feature/grid-integration-optimization
   ```

3. **少量多次提交**
   ```bash
   git add source/source_lcao/module_gint/
   git commit -m "Optimize data rearrangement for grid integration"
   git push origin feature/grid-integration-optimization
   ```

4. **提交 Pull Request**
   - 在 GitHub 上创建 Pull Request
   - 描述你的优化内容

---

## 五、参考资料

### 5.1 代码位置索引

| 文件 | 路径 | 说明 |
|------|------|------|
| GInt 类 | `source/source_lcao/module_gint/gint.h` | 格点积分核心类 |
| GInt 实现 | `source/source_lcao/module_gint/gint.cpp` | 格点积分实现 |
| 工具函数 | `source/source_lcao/module_gint/gint_tools.h` | 辅助工具 |
| 计算内核 | `source/source_lcao/module_gint/gint_kernels.h` | 计算内核 |

### 5.2 推荐阅读

1. **缓存优化**：《计算机体系结构：量化研究方法》- 内存层次结构章节
2. **SIMD编程**：Intel AVX 编程指南
3. **并行计算**：《并行程序设计原理》
4. **数据局部性**：《高性能计算导论》- 数据局部性优化

---

**最后更新**：2026-04-21

**版本**：v1.0