### 一、 MPI 核心认知与进阶技巧

- **MPI 本质**：它不是一种新的编程语言，而是一个跨语言的**通讯库标准**（常用实现如 OpenMPI、MPICH）。

- **内存架构**：基于**分布式内存**（Distributed Memory），每个进程有自己独立的内存空间，变量互不相通，必须通过消息传递来交换数据。

- **预处理适配（串行/并行兼容）**：为了让同一份代码既能用 `g++` 串行编译，也能用 `mpicxx` 并行编译，通常使用宏定义包裹 MPI 代码：

```cpp
    #ifdef MPI
        MPI_Init(&argc, &argv);
    #endif
```

- **多语言混编**：在 C++ 中调用 Fortran 代码时，需使用 `extern "C"` 避免编译器修改函数名（Name Mangling）。

- **C++执行Shell**：可配合 `std::stringstream` 和 `system()` 函数在 MPI 程序中动态创建文件夹等（如 `system("mkdir output");`）。

---

### 二、 编译、构建与运行指令速查

#### 1. 命令行直接编译运行

- **编译 (C++)**：`mpicxx -o hello_world hello_world.cpp`
- **运行**：`mpirun -np <进程数> ./hello_world`

#### 2. CMake 自动化构建（工程必备）

在 `CMakeLists.txt` 中引入 MPI 的标准写法：

```Cmake
find_package(MPI REQUIRED) # 查找MPI库
add_executable(my_app main.cpp)
# 将MPI的头文件和库链接到你的程序
target_link_libraries(my_app PUBLIC MPI::MPI_CXX) 
```

#### 3. 超算集群提交 (SLURM 系统)

- 编写 `job.sh` 脚本：

```sh
    #!/bin/bash
    #SBATCH -J my_mpi_job       # 作业名
    #SBATCH -p compute          # 队列名
    #SBATCH -N 2                # 节点数
    #SBATCH --ntasks-per-node=4 # 每个节点的进程数 (共8个进程)
    srun ./my_app               # SLURM环境下用 srun 替代 mpirun
```

- **提交作业**：`sbatch job.sh`

---

### 三、 核心 API：基础与环境管理

| **功能**      | **API 调用**                              | **说明/记忆点**                              |
| ----------- | --------------------------------------- | --------------------------------------- |
| **初始化**     | `MPI_Init(&argc, &argv);`               | 必须在所有 MPI 操作前调用（除 `MPI_Initialized` 外）。 |
| **结束**      | `MPI_Finalize();`                       | 必须在程序退出前调用，清理环境。                        |
| **我是谁(编号)** | `MPI_Comm_rank(MPI_COMM_WORLD, &rank);` | `rank` 范围从 `0` 到 `size - 1`。            |
| **我们有几个人**  | `MPI_Comm_size(MPI_COMM_WORLD, &size);` | 获取当前通信域的总进程数。                           |
| **获取节点名**   | `MPI_Get_processor_name(name, &len);`   | 获取运行当前进程的物理机器名（用于Debug分配情况）。            |
| **计时器**     | `double start = MPI_Wtime();`           | 返回自某特定过去的秒数。通常用 `结束时间 - 开始时间` 算耗时。      |

---

### 四、 点对点通信 API：阻塞 vs 非阻塞

通信的核心是 **“信封匹配”** ：Source/Dest（收发方）、Tag（标签）、Communicator（通信域）必须完全对齐才能通信。

#### 1. 阻塞通信 (Blocking)

**特点**：发件人必须等到信件被安全放进系统或被对方收走，才会执行下一行代码。极易引发**死锁**（如两个进程互相等待接收）。

- **发送**：`MPI_Send(&buf, count, datatype, dest, tag, comm)`
- **接收**：`MPI_Recv(&buf, count, datatype, source, tag, comm, &status)`

#### 2. 非阻塞通信 (Non-blocking) - 🔥 工程推荐

**特点**：发出收/发请求后立刻返回，CPU 可以继续干别的事（计算通信掩盖）。必须配合 `Wait` 来确认是否完成。

- **发送**：`MPI_Isend(&buf, count, datatype, dest, tag, comm, &request)`
- **接收**：`MPI_Irecv(&buf, count, datatype, source, tag, comm, &request)`
- **等待单个完成**：`MPI_Wait(&request, &status)`
- **等待多个完成**：`MPI_Waitall(count, array_of_requests, array_of_statuses)`

_(注：参数类型中，C++ 常用的有 `MPI_INT`, `MPI_DOUBLE`, `MPI_CHAR` 等。)_

---

### 五、 经典代码模板：非阻塞循环传数

这是处理 MPI 时最经典、最具代表性的防死锁结构。每个进程将自己的编号传递给下一个进程，首尾相连成一个环。

```cpp
#include <iostream>
#include "mpi.h"

int main(int argc, char **argv) {
    MPI_Init(&argc, &argv);

    int rank, size;
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);
    MPI_Comm_size(MPI_COMM_WORLD, &size);

    // 1. 计算收发目标 (首尾相连形成环)
    int to = (rank + 1) % size;               // 发给下一个，最后一个发给0
    int from = (rank - 1 + size) % size;      // 接收上一个，0号接收最后一个

    int info_send = rank;       // 要发送的数据
    int info_recv = -999;       // 用于接收的数据的 buffer
    int tag = 0;

    // 2. 准备请求句柄
    MPI_Request reqs[2];
    MPI_Status stats[2];

    // 3. 发起非阻塞接收和发送 (Irecv 和 Isend 顺序无所谓，因为是非阻塞)
    // 强烈建议：先挂起 Irecv，再执行 Isend，效率最高且最安全
    MPI_Irecv(&info_recv, 1, MPI_INT, from, tag, MPI_COMM_WORLD, &reqs[0]);
    MPI_Isend(&info_send, 1, MPI_INT, to,   tag, MPI_COMM_WORLD, &reqs[1]);

    // 4. (此处可插入不依赖通信数据的其他计算逻辑) ...

    // 5. 等待通信彻底完成
    MPI_Waitall(2, reqs, stats);

    std::cout << "Processor " << rank << " received data: " << info_recv << " from " << from << std::endl;

    MPI_Finalize();
    return 0;
}
```