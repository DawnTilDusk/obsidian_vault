---
epoch: 1777277633950
modelKey: "gpt-4|github-copilot"
topic: "Model Not Supported Error"



tags:
  - copilot-conversation
---

**user**: 你是谁
[Timestamp: 2026/04/27 16:13:53]

**ai**: 
<errorChunk>400 The requested model is not supported.</errorChunk>
[Timestamp: 2026/04/27 16:13:55]

**user**: 你是谁
[Timestamp: 2026/04/27 16:14:05]

**ai**: 
<errorChunk>400 The requested model is not supported.</errorChunk>
[Timestamp: 2026/04/27 16:14:06]

**user**: 你是谁
[Timestamp: 2026/04/27 16:14:15]

**ai**: 我是 Obsidian Copilot，你的智能助手，专门为 Obsidian 笔记应用集成 AI 功能。可以帮助你整理、查询、总结和优化你的笔记内容。如果你有任何关于 Obsidian 或笔记管理的问题，随时可以问我！
[Timestamp: 2026/04/27 16:14:31]

**user**: 你能生成文档吗
[Context: Notes: 2026spring/并行程序设计/MPI/MPI参量类型速查.md]
[Timestamp: 2026/04/27 16:16:46]

**ai**: 可以的！根据你提供的内容和上下文，我可以帮你生成结构化、条理清晰的文档。请说明你需要生成哪方面的文档（比如：项目说明书、技术手册、API 速查表、用户指南等），以及需要包含哪些具体内容或格式要求。如果你有特定的模板或章节结构，也可以告诉我，我会按照你的需求来组织和输出文档。
[Timestamp: 2026/04/27 16:17:11]

**user**: MPI_Request句柄有什么意义
[Context: Notes: 2026spring/并行程序设计/MPI/MPI参量类型速查.md]
[Timestamp: 2026/04/27 16:21:44]

**ai**: MPI_Request 句柄是 MPI 非阻塞通信（如 MPI_Isend、MPI_Irecv）中的一个**请求对象**，用于追踪和管理一次异步通信操作的状态。

其意义主要体现在以下几个方面：

- **标识一次未完成的通信操作**：每当你调用 MPI_Isend 或 MPI_Irecv 发起非阻塞通信时，MPI 会立即返回，不会等待通信完成。此时，MPI_Request 句柄就记录了这次通信的“任务单”，你可以用它后续查询或等待该操作的完成[^1][^2]。

- **配合 MPI_Wait/MPI_Waitall 等函数**：只有通过 MPI_Request，你才能用 MPI_Wait(&request, ...) 或 MPI_Waitall(count, requests, ...) 等函数等待这些异步操作的完成，确保数据已经安全地发送或接收完毕[^1][^2]。

- **支持并发和批量通信**：MPI_Request 可以是单个变量，也可以是数组。这样你可以同时管理多个并发的通信任务，最后用 MPI_Waitall 一次性等待全部完成，极大提高并行效率[^1][^2]。

- **内存安全**：只有在对应的 MPI_Request 被 MPI_Wait 或 MPI_Waitall 等函数“回收”后，相关的发送/接收缓冲区才可以被安全读写，否则可能出现数据竞争或未定义行为[^1][^2]。

简而言之，MPI_Request 是非阻塞通信的“凭证”，它让你能够灵活地安排通信与计算的重叠，并安全地管理通信生命周期。

#### Sources
[^1]: [[MPI点对点通信参量类型速查]]
[^2]: [[MPI点对点通信速查手记]]
[Timestamp: 2026/04/27 16:22:47]

**user**: 我怎么启动多个进程？
[Context: Notes: 2026spring/并行程序设计/MPI/MPI参量类型速查.md]
[Timestamp: 2026/04/27 16:44:16]

**ai**: 在 MPI 并行程序中，启动多个进程通常有两种常见方式：

---

### 1. 命令行直接启动

最常用的方式是使用 `mpirun` 或 `mpiexec` 命令，在命令行中指定进程数。例如：

```bash
mpirun -np 4 ./your_program
```
- `-np 4` 表示启动 4 个进程。
- `./your_program` 是你编译好的可执行文件。

你也可以用 `mpiexec`，用法类似：

```bash
mpiexec -np 8 ./your_program
```
这会在本机上启动 8 个 MPI 进程。

---

### 2. 集群/超算作业脚本启动

在集群或超级计算机上，通常通过作业调度系统（如 SLURM）提交作业脚本。例如：

```bash
#!/bin/bash
#SBATCH -J my_mpi_job       # 作业名
#SBATCH -p compute          # 队列名
#SBATCH -N 2                # 节点数
#SBATCH --ntasks-per-node=4 # 每个节点的进程数 (共8个进程)
srun ./your_program         # SLURM环境下用 srun 替代 mpirun
```
然后用 `sbatch job.sh` 提交作业脚本即可[^1]。

---

### 3. 其他参数说明

- 你可以通过 `MPI_Comm_rank` 和 `MPI_Comm_size` 在代码中获取当前进程编号和总进程数，实现进程间的分工协作[^2][^3]。
- 在多节点环境下，`mpirun`/`mpiexec` 可以通过参数或主机文件指定分布在哪些节点上运行。

---

### 4. 典型编译与运行流程

```bash
mpicxx -o my_app main.cpp      # 编译
mpirun -np 4 ./my_app          # 运行4个进程
```
---

### 5. 参考

- 你可以在 [[MPI点对点通信速查手记]] 的“编译、构建与运行指令速查”部分找到详细示例[^1]。

---

#### Sources
[^1]: [[MPI点对点通信速查手记]]
[^2]: [[MPI点对点通信参量类型速查]]
[^3]: [[09_hardware]]
[Timestamp: 2026/04/27 16:44:54]