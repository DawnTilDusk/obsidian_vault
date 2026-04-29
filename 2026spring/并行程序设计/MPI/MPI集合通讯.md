### 一、 核心概念与准则 (The Golden Rules)

在写集合通讯代码时，请牢记以下准则，这能帮你避开 90% 的 Bug：

1. **同进同退（全局性）**：集合通信函数**必须被通信域（如 `MPI_COMM_WORLD`）内的所有进程调用**。如果有一个进程没执行到该代码，整个程序就会死锁卡住。

2. **没有标签（No Tags）**：不同于点对点通信（Send/Recv），集合通信**不需要**指定 Tag，它们靠执行顺序和通信域来匹配。

3. **匹配的参数**：所有进程调用的 `count`（数量）、`datatype`（类型）、`op`（操作符）、`root`（根进程）在大多数情况下必须保持完全一致。

4. **加速比定律**：
    - **Amdahl 定律**：描述**强可扩展性**（问题规模不变，加核心）。程序的加速极限受限于“不能并行的那部分代码（串行分量）”。
    - **弱可扩展性**：增加核心的同时等比例增加任务量，看单核效率是否下降。

---

### 二、 核心 API 功能速查与代码框架

遇到需求时，先从这里找对应的模型。

#### 1. 同步 (Synchronization)

- **指令**：`MPI_Barrier`
- **应用场景**：发令枪。等所有人跑到起跑线，再一起往下执行（常用于计时前/后，或者 I/O 操作前防止冲突）。
- **代码示例**：
```cpp unwrap=true
    std::cout << "进程 " << rank << " 准备好了" << std::endl;
    MPI_Barrier(MPI_COMM_WORLD);//所有人在这里卡住，直到最后一个人到达
    std::cout << "所有人准备完毕，一起出发！" << std::endl;
```
#### 2. 一对多 (One-to-All)

- **广播 `MPI_Bcast`**：**（复制复印）** 老板把**同一份**文件复印发给所有人。
- **分发 `MPI_Scatter`**：**（发扑克牌）** 老板把**一堆**文件拆开，每人发不同的一张。
- **代码示例 (Bcast)**：
```cpp
    int value;
    if (rank == 0) value = 999; // 只有老板(root=0)有数据
    MPI_Bcast(&value, 1, MPI_INT, 0, MPI_COMM_WORLD); 
    // 执行完后，所有进程的 value 都变成了 999
```

#### 3. 多对一 / 全局计算 (All-to-One / Global)

- **规约 `MPI_Reduce`**：**（收集并计算）** 老板把所有人的报表收上来，并计算总和/最大值。结果**只有老板知道**。

- **集中 `MPI_Gather`**：**（收考卷）** 老板把所有人的试卷按顺序收上来，拼成一叠。不作计算。

- **全规约 `MPI_Allreduce`**：**（黑板计算）** 所有人把数据写在黑板上，求出总和，**所有人都能看到结果**（相当于 Reduce + Bcast）。

#### 4. 高级洗牌操作 (Advanced Shuffling)

- **全集中 `MPI_Allgather`**：所有人把各自的数据交出来拼在一起，然后**每个人都拷贝一份完整的拼图**。

- **全交换 `MPI_Alltoall`**：每个人都给所有人发**不同**的数据（转置操作）。

- **规约并分发 `MPI_Reduce_scatter`**：先对所有人的数组做 Reduce，然后把结果数组切块，按规定（`recvcounts`）分发给对应的人。

- **前缀和 `MPI_Scan`**：进程 $i$ 收到的是进程 $0$ 到进程 $i$ 数据的累积操作（如累加和）。

---

### 三、 API 传参详解与包装格式 (速查字典)

MPI 的传参非常规律，通常分为：**发送区、接收区、计数、类型、\[操作符]、\[根进程]、通信域**。

> **🔴 易错警告（The "IN_PLACE" Rule）**：
> 
> 在 C++ 中，`sendbuf` 和 `recvbuf` **不能是同一个内存地址**（除非你明确使用 `MPI_IN_PLACE`）。

|**函数**|**参数格式**|**参数图解 / 记忆点**|
|---|---|---|
|**`MPI_Barrier`**|`(comm)`|最简单，只有一个通信域。|
|**`MPI_Bcast`**|`(buffer, count, datatype, root, comm)`|**注意**：`buffer` 既是 root 的发送区，也是其他人的接收区。|
|**`MPI_Reduce`**|`(sendbuf, recvbuf, count, datatype, op, root, comm)`|**操作符(op)**：常用 `MPI_SUM` (求和), `MPI_MAX`, `MPI_MIN`, `MPI_PROD` (乘积)。<br><br>  <br><br>**注意**：`recvbuf` 只有在 `root` 进程中才有意义，其他人可以传 `NULL` 或随意定义。|
|**`MPI_Allreduce`**|`(sendbuf, recvbuf, count, datatype, op, comm)`|**少了一个参数**：没有 `root`，因为所有人都会得到结果。|
|**`MPI_Scatter`**|`(sendbuf, sendcount, sendtype, recvbuf, recvcount, recvtype, root, comm)`|**注意 count**：`sendcount` 是指发给**每个**人的数量，而不是总数量！例如老板有 10 个苹果发给 2 个人，`sendcount` 填 5。|
|**`MPI_Gather`**|`(sendbuf, sendcount, sendtype, recvbuf, recvcount, recvtype, root, comm)`|**注意 count**：同上，`recvcount` 是指从**每个**人那里收到的数量。如果每人交 5 份，就填 5，不是填 10。|
|**`MPI_Allgather`**|`(sendbuf, sendcount, sendtype, recvbuf, recvcount, recvtype, comm)`|没有 `root` 参数。|
|**`MPI_Alltoall`**|`(sendbuf, sendcount, sendtype, recvbuf, recvcount, recvtype, comm)`|没有 `root` 参数。<br><br>  <br><br>**注意**：`sendcount` 是发给**每个**进程的数量。|
|**`MPI_Reduce_scatter`**|`(sendbuf, recvbuf, recvcounts[], datatype, op, comm)`|**特殊参数**：`recvcounts` 是一个**数组**（每个进程都有这个数组的拷贝），里面写明了每个人该分到多少个结果。|
|**`MPI_Scan`**|`(sendbuf, recvbuf, count, datatype, op, comm)`|参数同 Allreduce，执行前缀累积。|

---

### 💡 实战避坑建议 (针对作业与考试)：

1. **遇到段错误 (Segmentation Fault)**：检查 `Scatter/Gather` 的 `count` 参数。如果进程数 `size` 无法整除数据总量，会导致数组越界或空指针（见课件 Scatter 隐患）。实际工程中通常使用 `MPI_Scatterv` 和 `MPI_Gatherv` 来处理不均匀的数据分配。

2. **指针传递**：对于单个变量，记得加 `&`（如 `&value`）。对于数组/`std::vector`，传数组名或 `.data()`。