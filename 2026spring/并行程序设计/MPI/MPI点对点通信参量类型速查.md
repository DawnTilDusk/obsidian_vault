### 一、 MPI 常用内置常量与包装类型 (The "Wrappers")

在填入 API 参数时，你不可避免地会用到 MPI 自己封装的数据类型和宏定义。

#### 1. 数据类型映射 (`MPI_Datatype`)

通信时，必须用 MPI 的类型替代 C++ 的原生类型，告诉底层系统要打包多大的内存区块：

- `{c++}int` $\rightarrow$ **`MPI_INT`**
- `{c++}double` $\rightarrow$ **`MPI_DOUBLE`**
- `{c++}float` $\rightarrow$ **`MPI_FLOAT`**
- `{c++}char` $\rightarrow$ **`MPI_CHAR`**

#### 2. 通信域 (`MPI_Comm`)

- **`MPI_COMM_WORLD`**：包含所有启动进程的默认全局通信域（99% 的情况直接填这个）。

#### 3. 偷懒专用常量 (Wildcards)

- 接收任意来源：**`MPI_ANY_SOURCE`**（填在 `source` 位置）
- 接收任意标签：**`MPI_ANY_TAG`**（填在 `tag` 位置）
- 忽略状态信息：**`MPI_STATUS_IGNORE`**（填在 `&status` 位置，如果你不想查阅具体状态的话）

---
### 二、 环境与信息管理 API 传参速查

这些函数的特点是：参数多为**指针**，因为它们要把系统的状态值“写”回你定义的变量里。

| **API 函数签名**                                  | **常用传参示例**                              | **易错点提示**             |
| --------------------------------------------- | --------------------------------------- | --------------------- |
| `int MPI_Init(int *argc, char ***argv)`       | `MPI_Init(&argc, &argv);`               | 必须传入 `main` 函数的参数地址。  |
| `int MPI_Finalize(void)`                      | `MPI_Finalize();`                       | 无参数，但绝不能漏掉。           |
| `int MPI_Comm_rank(MPI_Comm comm, int *rank)` | `MPI_Comm_rank(MPI_COMM_WORLD, &rank);` | `rank` 必须传地址 `&rank`。 |
| `int MPI_Comm_size(MPI_Comm comm, int *size)` | `MPI_Comm_size(MPI_COMM_WORLD, &size);` | `size` 必须传地址 `&size`。 |
| `double MPI_Wtime(void)`                      | `double t = MPI_Wtime();`               | 直接返回 `double` 类型的秒数。  |

---

### 三、 点对点通信 API 传参剖析 (Point-to-Point)

这是最复杂的参数群，通常有 **6 到 7 个参数**。为了好记，你可以把它们分为三段：**【数据区】【信封区】【控制区】**。

#### 1. 阻塞通信 (Blocking)

- **发送 (Send)**:
    `{c++}MPI_Send(const void *buf, int count, MPI_Datatype datatype, int dest, int tag, MPI_Comm comm)`

- **接收 (Recv)**:
    `{c++}MPI_Recv(void *buf, int count, MPI_Datatype datatype, int source, int tag, MPI_Comm comm, MPI_Status *status)`

|**区域**|**参数名**|**类型与说明**|**代码示例**|
|---|---|---|---|
|**数据区**|`buf`|发送/接收缓冲区的**首地址**（若是数组直接写数组名，若是单变量需加 `&`）|`&val` 或 `arr`|
||`count`|发送/接收的元素**个数**（不是字节数！）|`1` 或 `100`|
||`datatype`|元素的数据类型|`MPI_INT`|
|**信封区**|`dest` / `source`|目标或来源进程的编号 (`rank` 值)|`1` 或 `from`|
||`tag`|消息标签（双方必须一致，用于区分不同消息）|`0` 或 `99`|
|**控制区**|`comm`|通信域|`MPI_COMM_WORLD`|
||`status`|_(仅 Recv)_ 接收状态的记录结构体地址|`&status` 或 `MPI_STATUS_IGNORE`|

#### 2. 非阻塞通信 (Non-blocking)

非阻塞的核心区别在于：它们多了一个 **`MPI_Request`** 句柄（必须传地址），且 Recv 阶段不需要 `status`（把 status 移交给了 `Wait` 阶段）。

- **发送 (Isend)**:
    `{c++}MPI_Isend(const void *buf, int count, MPI_Datatype datatype, int dest, int tag, MPI_Comm comm, MPI_Request *request)`

- **接收 (Irecv)**:
    `{c++}MPI_Irecv(void *buf, int count, MPI_Datatype datatype, int source, int tag, MPI_Comm comm, MPI_Request *request)`

---

### 四、 状态等待 API 传参 (Wait)

在使用 `Isend/Irecv` 后，必须配套使用 `Wait` 系列函数来确认内存可以被安全读写。

| **API 函数签名**                                                                                   | **常用传参示例**                                   | **适用场景**                                                            |
| ---------------------------------------------------------------------------------------------- | -------------------------------------------- | ------------------------------------------------------------------- |
| `{c++}MPI_Wait(MPI_Request *request, MPI_Status *status)`                                      | `MPI_Wait(&req, MPI_STATUS_IGNORE);`         | 等待**单个**请求完成。记得 `req` 要加 `&`。                                       |
| `{c++}MPI_Waitall(int count, MPI_Request array_of_requests[], MPI_Status array_of_statuses[])` | `MPI_Waitall(2, reqs, MPI_STATUSES_IGNORE);` | 等待**数组内所有**请求完成。`reqs` 本身是数组，不加 `&`。忽略状态用特殊宏 `MPI_STATUSES_IGNORE`。 |

---

### 💡 极速拷贝模板：标准参数组合

遇到写代码卡壳，直接对照下面的变量类型和传参方式来写：

```c++
// 1. 准备变量
int send_data = 100;
int recv_data;
MPI_Request reqs[2]; // 请求数组
MPI_Status stats[2]; // 状态数组

// 2. 发起非阻塞接收 (注意 buf 加 &，reqs 传对应位置的地址)
MPI_Irecv(&recv_data, 1, MPI_INT, 0, 0, MPI_COMM_WORLD, &reqs[0]);

// 3. 发起非阻塞发送
MPI_Isend(&send_data, 1, MPI_INT, 1, 0, MPI_COMM_WORLD, &reqs[1]);

// 4. 等待它们全部完成 (2是数组长度，reqs是数组名，stats是数组名)
MPI_Waitall(2, reqs, stats); 

// 如果你不想管状态，可以把 stats 换成占位符：
// MPI_Waitall(2, reqs, MPI_STATUSES_IGNORE);
```