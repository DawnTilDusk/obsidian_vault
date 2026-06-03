# ABACUS NEB/DFPT 并行策略重构

## 一、背景与需求

### 1.1 NEB 计算概述

**NEB（Nudged Elastic Band）** 是一种计算反应路径和能垒的标准方法，用于研究化学反应的过渡态。在NEB计算中：

- **多个图像（Image）**：在初始态和最终态之间插入多个中间结构
- **并行图像**：不同图像可以并行计算，每个图像独立进行SCF计算
- **弹簧力连接**：相邻图像之间通过弹簧力保持等间距
- **最终态识别**：找到能量最高的鞍点

### 1.2 DFPT 概述

**DFPT（Density Functional Perturbation Theory）** 用于计算线性响应性质，如：

- **声子谱**：通过原子位移的响应计算动力学矩阵
- **介电性质**：通过外加电场的响应计算介电常数
- **弹性常数**：通过应变扰动计算弹性张量

### 1.3 当前架构的局限性

ABACUS现有架构主要针对：
- 单点能计算
- 几何优化
- 分子动力学

**不支持NEB/DFPT的直接原因**：

| 问题 | 描述 |
|------|------|
| Cell 耦合 | 当前Cell与单个计算绑定，NEB需要多个独立的Cell |
| MPI通讯模式 | 当前MPI通讯主要在pool内，NEB需要跨图像通讯 |
| 数据结构 | 电荷密度、势能等按单一体系设计 |
| 内存管理 | 无法有效管理多图像的并行存储 |

### 1.4 重构目标

1. **支持多图像并行**：每个图像独立计算，图像间仅传递少量数据
2. **Cell模块化**：将Cell从单一绑定改为可复制/共享模式
3. **优化通讯**：最小化图像间通讯开销
4. **内存效率**：支持多图像的内存共享和复用

## 二、Cell 模块重构

### 2.1 现有 Cell 结构

当前 ABACUS 中 Cell 的使用方式：

```cpp
// source/source_esolver/esolver_fp.cpp
class ESolver_FP {
protected:
    UnitCell* p_ucell;  // 单一Cell指针
    // ...
};
```

**问题**：
- Cell 与 ESolver 紧密耦合
- 无法支持多图像的独立 Cell
- Cell 的创建、销毁缺乏灵活机制

### 2.2 新架构设计

#### 2.2.1 Cell 容器抽象

```cpp
// 新设计：Cell容器管理多个独立的Cell实例
class CellContainer {
public:
    // 创建新图像的Cell
    UnitCell* create_cell();
    
    // 获取图像i的Cell
    UnitCell* get_cell(int image_id);
    
    // 删除图像i的Cell
    void delete_cell(int image_id);
    
    // 图像间Cell数据同步
    void sync_cells(const std::vector<int>& image_ids);
    
    // 共享只读数据（原子类型、势函数等）
    void share_static_data(CellContainer& other);
    
private:
    std::vector<std::unique_ptr<UnitCell>> cells_;
    std::shared_ptr<CellStaticData> shared_static_data_;
};
```

#### 2.2.2 Cell 静态数据分离

将 Cell 中不变的数据分离出来，供多图像共享：

```cpp
// 静态数据（只读，可共享）
struct CellStaticData {
    std::vector<AtomType> atom_types;      // 原子类型信息
    std::vector<Pseudopotential> pseudos;  // 赝势数据
    LCAO_Orbital_set orbitals;              // 原子轨道基组
    SymmetryData sym_data;                 // 对称性数据
};

// 动态数据（每个图像独立）
struct CellDynamicData {
    std::vector<AtomPosition> positions;   // 原子位置
    ModuleBase::Matrix3 lattice;            // 晶格矢量
    double energy;                          // 当前能量
    ModuleBase::Matrix3 stress;             // 应力张量
};
```

**优势**：
- 减少内存占用：静态数据只存储一份
- 加速图像创建：只需复制动态数据
- 便于并行：静态数据可设为只读共享

### 2.3 Image 并行管理

```cpp
class ImageManager {
public:
    // 初始化N图像系统
    void init_images(int n_images, const UnitCell& reference);
    
    // 获取指定图像的数据
    ImageData& get_image(int image_id);
    
    // 图像间插值
    void interpolate_images(const std::vector<double>& energies);
    
    // 并行执行所有图像的SCF
    void solve_all_images(ESolver_FP& esolver);
    
    // 计算弹簧力和垂直力
    void calculate_forces();
    
private:
    int n_images_;
    std::vector<CellDynamicData> image_data_;
    std::vector<ParallelConfig> parallel_config_;
};
```

## 三、MPI 通讯优化

### 3.1 NEB 中的通讯模式

NEB 计算中的通讯模式：

```
Image 0  Image 1  Image 2  Image 3  Image N-1
   |         |         |         |         |
   |         |         |         |         |
   +---------+---------+---------+---------+
                图像间通讯
            (能量、力、位置)
```

**通讯需求**：
- 每轮迭代：图像间交换能量和力
- 同步点：所有图像到达同一迭代后继续
- 数据量小：主要是标量（能量）和短向量（力）

### 3.2 通讯域设计

#### 3.2.1 二维进程网格

```cpp
// NEB并行：二维进程网格
// 维度1：图像并行 (NPROC_IMAGE)
// 维度2：单图像内并行 (NPROC_SCF)

class NEBParallelConfig {
public:
    int nproc_image;   // 图像数
    int nproc_scf;    // 每图像SCF并行进程数
    int total_procs;  // nproc_image * nproc_scf
    
    // 通讯子
    MPI_Comm comm_image;  // 同图像内进程
    MPI_Comm comm_scf;    // 同图像SCF进程
    
    // 创建通讯域
    void create_mpi_comms(MPI_Comm world);
    
    // 进程定位
    int get_image_rank() const;
    int get_scf_rank() const;
};
```

#### 3.2.2 通讯模式对比

| 模式 | 优点 | 缺点 | 适用场景 |
|------|------|------|----------|
| 纯图像并行 | 简单，通讯少 | 单图像效率低 | 小体系 |
| 二维网格 | 平衡效率 | 实现复杂 | 大体系 |
| 动态负载均衡 | 负载最优化 | 开销大 | 图像计算量差异大 |

### 3.3 非阻塞通讯优化

#### 3.3.1 图像间非阻塞同步

```cpp
// 非阻塞同步图像数据
class ImageSync {
private:
    std::vector<MPI_Request> requests_;
    std::vector<double> send_energies_;
    std::vector<double> recv_energies_;
    
public:
    // 异步发送能量数据
    void isend_energies(int dest_image, double energy) {
        MPI_Issend(&energy, 1, MPI_DOUBLE, dest_image, 
                   TAG_ENERGY, MPI_COMM_NEB, &requests_.back());
    }
    
    // 异步接收能量数据
    void irecv_energies(int src_image, double& energy) {
        MPI_Irecv(&energy, 1, MPI_DOUBLE, src_image,
                  TAG_ENERGY, MPI_COMM_NEB, &requests_.back());
    }
    
    // 等待所有通讯完成
    void wait_all() {
        MPI_Waitall(requests_.size(), requests_.data(), MPI_STATUSES_IGNORE);
    }
};
```

#### 3.3.2 计算与通讯重叠

```cpp
// 重叠计算与通讯
void NEBOptimizer::optimize() {
    for (int iter = 0; iter < max_iter; ++iter) {
        // 启动异步发送
        for (int i = 0; i < n_images_; ++i) {
            sync_.isend_forces(i, forces_[i]);
        }
        
        // 本地图像计算（不依赖远端数据）
        for (int i = 0; i < n_images_; ++i) {
            if (!images_[i].is_master()) continue;
            calculate_spring_forces(i);
        }
        
        // 等待远端数据
        sync_.wait_all();
        
        // 计算垂直力
        calculate_perpendicular_forces();
        
        // 更新位置
        update_positions();
    }
}
```

### 3.4 集合通讯优化

#### 3.4.1 NEB 专用归约操作

```cpp
// 定义NEB专用的归约操作
void neb_allreduce_forces(double* local_forces, double* global_forces, int n) {
    // 图像内归约
    MPI_Reduce(local_forces, global_forces, n, MPI_DOUBLE,
               MPI_SUM, 0, comm_image_);
    
    // 广播给同图像所有进程
    MPI_Bcast(global_forces, n, MPI_DOUBLE, 0, comm_image_);
}

// 计算所有图像的能量差异
void neb_allgather_energies(std::vector<double>& energies) {
    MPI_Allgather(MPI_IN_PLACE, 1, MPI_DOUBLE,
                  energies.data(), 1, MPI_DOUBLE,
                  comm_image_all_);
}
```

## 四、代码框架调整

### 4.1 ESolver 接口扩展

#### 4.1.1 多图像 ESolver 抽象

```cpp
// 支持多图像计算的ESolver接口
class IESolverMultiImage {
public:
    virtual ~IESolverMultiImage() = default;
    
    // 初始化多图像计算
    virtual void init_multi_images(int n_images) = 0;
    
    // 为指定图像设置Cell
    virtual void set_cell(int image_id, UnitCell* cell) = 0;
    
    // 运行指定图像的SCF
    virtual void solve_one_image(int image_id) = 0;
    
    // 运行所有图像
    virtual void solve_all_images() = 0;
    
    // 获取指定图像的能量和力
    virtual double get_image_energy(int image_id) = 0;
    virtual const double* get_image_forces(int image_id) = 0;
};
```

#### 4.1.2 现有 ESolver 适配

```cpp
// 适配现有ESolver到多图像接口
template <typename ESolverType>
class MultiImageESolver : public IESolverMultiImage {
private:
    std::unique_ptr<ESolverType> esolver_;
    std::vector<std::unique_ptr<UnitCell>> cells_;
    
public:
    void init_multi_images(int n_images) override {
        cells_.resize(n_images);
        for (int i = 0; i < n_images; ++i) {
            cells_[i] = std::make_unique<UnitCell>();
        }
    }
    
    void set_cell(int image_id, UnitCell* cell) override {
        // 从reference cell复制配置
        *cells_[image_id] = *cell;
        esolver_->set_cell(cells_[image_id].get());
    }
    
    void solve_one_image(int image_id) override {
        esolver_->set_cell(cells_[image_id].get());
        esolver_->solve(SCF);
    }
    
    // ... 其他接口实现
};
```

### 4.2 工厂模式创建

```cpp
// ESolver工厂，支持多图像计算
class ESolverFactory {
public:
    enum class Type {
        KSDFT,      // 传统KS-DFT
        OFDFT,      // 轨道无关DFT
        NEB,        // NEB计算
        DFPT        // 线性响应
    };
    
    static std::unique_ptr<IESolverMultiImage> create(Type type);
};
```

### 4.3 数据管理重构

#### 4.3.1 多图像数据结构

```cpp
// 多图像计算的数据容器
class MultiImageData {
public:
    // 电荷密度管理
    std::vector<std::unique_ptr<Charge>> rho;      // 每个图像的电荷密度
    std::unique_ptr<Charge> rho_avg;                // 平均电荷密度（可选）
    
    // 势能管理
    std::vector<std::unique_ptr<Potential>> V_eff;  // 每个图像的有效势
    
    // 力和能量
    std::vector<double> energies;
    std::vector<ModuleBase::Matrix> forces;
    
    // 共享数据（只读）
    std::shared_ptr<XCFunctional> xc_func;
    std::shared_ptr<PseudopotentialCell> psp_cell;
};
```

#### 4.3.2 内存优化策略

```cpp
// 内存池管理，减少分配开销
class NEBMemoryPool {
private:
    // 预分配的内存块
    std::vector<double> rho_buffer_;
    std::vector<double> V_buffer_;
    std::vector<double> force_buffer_;
    
public:
    // 获取可用内存块
    double* allocate_rho(int nrxx);
    
    // 归还内存块
    void deallocate_rho(double* ptr);
    
    // 清空所有缓冲区
    void clear();
    
    // 调整缓冲区大小
    void resize(int new_nrxx);
};
```

## 五、DFPT 扩展设计

### 5.1 DFPT 特殊性

DFPT 与 NEB 的关键区别：

| 特性 | NEB | DFPT |
|------|-----|------|
| 图像关系 | 独立，平滑连接 | 扰动态依赖原始态 |
| 求解方式 | 并行优化 | 耦合求解 |
| 内存需求 | 多份独立数据 | 原始+扰动数据 |
| 收敛控制 | 各图像独立 | 全系统同步 |

### 5.2 DFPT 并行策略

#### 5.2.1 扰动模式并行

```cpp
class DFPT_Perturbation {
public:
    // 扰动类型
    enum class PerturbationType {
        AtomicDisplacement,  // 原子位移
        ElectricField,       // 电场
        Strain              // 应变
    };
    
    // 并行处理多个扰动
    void solve_all_perturbations(
        const std::vector<PerturbationType>& perturbations,
        ParallelConfig& parallel
    );
};
```

#### 5.2.2 线性响应求解器

```cpp
class LinearResponseSolver {
private:
    // 原始态数据
    std::unique_ptr<ScfResult> ground_state_;
    
    // 扰动态数据
    std::vector<std::unique_ptr<ScfResult>> perturbed_states_;
    
public:
    // 设置原始态
    void set_ground_state(const ScfResult& gs);
    
    // 计算对扰动的响应
    void compute_response(PerturbationType type, int perturbation_id);
    
    // 构建线性响应矩阵
    void build_perturbation_matrix();
    
    // 求解响应
    void solve();
};
```

## 六、实施计划

### 6.1 阶段一：Cell 重构（基础）

**目标**：实现 Cell 模块的分离和共享

**任务**：
1. 分离 CellStaticData
2. 实现 CellContainer
3. 修改 ESolver 接口
4. 编写单元测试

**验收标准**：
- Cell 可以独立创建和销毁
- 静态数据正确共享
- 内存占用减少 > 30%

### 6.2 阶段二：NEB 支持

**目标**：实现基本的 NEB 计算

**任务**：
1. 实现 ImageManager
2. 设计 NEB 通讯域
3. 实现 NEB 优化器
4. 集成到 ABACUS 主程序

**验收标准**：
- NEB 计算正确性验证
- 并行效率 > 70%
- 支持 >= 16 图像

### 6.3 阶段三：DFPT 扩展

**目标**：扩展支持 DFPT 计算

**任务**：
1. 实现扰动管理
2. 线性响应求解器
3. DFPT 并行优化
4. 声子谱计算接口

**验收标准**：
- 声子谱计算正确性验证
- 与 Phonopy 接口兼容

### 6.4 代码位置

| 模块 | 路径 | 说明 |
|------|------|------|
| Cell 重构 | `source/source_cell/module_cell/` | Cell 模块 |
| Cell 容器 | `source/source_esolver/module_esolver/` | ESolver 接口 |
| NEB 实现 | `source/module_neb/` | 新增 NEB 模块 |
| DFPT 实现 | `source/module_dfpt/` | 新增 DFPT 模块 |
| 并行工具 | `source/base/parallel/` | MPI 通讯工具 |

## 七、测试与验证

### 7.1 功能测试

#### 7.1.1 NEB 测试

| 测试案例 | 描述 | 预期结果 |
|---------|------|---------|
| 01_simple_reaction | H2 dissociation | 能量曲线正确 |
| 02_diffusion | Cu adatom on Cu surface | 鞍点正确 |
| 03_barrier | N2 + H -> NH2 | 能垒误差 < 0.1 eV |

#### 7.1.2 DFPT 测试

| 测试案例 | 描述 | 预期结果 |
|---------|------|---------|
| 01_phonon_Al | Al fcc，声子谱 | 与文献一致 |
| 02_dielectric_Si | Si 介电常数 | 误差 < 5% |
| 03_elastic_NaCl | NaCl 弹性常数 | 与文献一致 |

### 7.2 性能测试

| 指标 | 单图像 | 4图像 | 8图像 | 16图像 |
|------|--------|-------|-------|--------|
| 并行效率 | 100% | > 85% | > 75% | > 65% |
| 内存扩展 | 1x | < 3.5x | < 6.5x | < 12x |
| 迭代次数 | 20 | ~20 | ~20 | ~22 |

### 7.3 基准测试脚本

```bash
#!/bin/bash
# benchmark_neb.sh - NEB性能测试

export OMP_NUM_THREADS=8

for nproc in 1 4 8 16; do
    for nimage in 1 4 8 16; do
        echo "Testing: nproc=$nproc, nimage=$nimage"
        mpirun -np $((nproc * nimage)) ./abacus INPUT_neb > log_p${nproc}_i${nimage}.out
        grep "converged" log_p${nproc}_i${nimage}.out | tail -1
    done
done
```

## 八、总结

### 8.1 重构要点

1. **Cell 模块化**：分离静态和动态数据，支持多图像独立 Cell
2. **并行架构**：二维进程网格，支持图像和 SCF 并行
3. **通讯优化**：非阻塞通讯，计算与通讯重叠
4. **接口扩展**：适配现有 ESolver，支持多图像计算

### 8.2 预期收益

- **内存效率**：多图像共享静态数据，内存占用减少 30-50%
- **计算效率**：图像级并行，加速比接近线性
- **扩展性**：支持大规模 NEB 和 DFPT 计算

### 8.3 未来展望

1. **动态负载均衡**：根据图像计算负载自动调整进程分配
2. **自适应图像**：在优化过程中自动增减图像
3. **机器学习加速**：用 ML 模型加速 NEB 优化

---

**最后更新**：2026-04-21

**版本**：v1.0