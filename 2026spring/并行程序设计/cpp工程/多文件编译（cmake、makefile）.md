## 编译架构全景图

> **源代码** $\rightarrow$ **CMakeLists.txt** $\xrightarrow{\text{CMake}}$ **Makefile** $\xrightarrow{\text{make}}$ **可执行文件**

1. **编写**：你写了 `main.cpp` 和 `MessageBoard.cpp`。
2. **配置**：运行 `cmake` $\rightarrow$ 生成了 **Makefile**。
3. **构建**：运行 `make` $\rightarrow$ 调用 `g++`。
4. **编译**：`g++` 把两个 `.cpp` 分别变成了 `main.o` 和 `MessageBoard.o`。
5. **链接**：`g++` 把两个 `.o` 拼在一起，生成了 **`main.exe`** (或 Linux 下的 `a.out`)。

## 1. C++ 编译基础流程

编译是将源代码转换为可执行文件的过程，主要分为四个阶段：

- **预处理 (Preprocessing)**：处理 `#include`、`#define` 等宏定义 。
- **编译 (Compilation)**：将源代码转换为汇编代码 。
- **汇编 (Assembly)**：将汇编代码转换为机器码（`.o` 或 `.obj` 目标文件） 。
- **链接 (Linking)**：将多个目标文件和库链接成最终的可执行文件 。

**常用指令 (GCC/G++)：**

- **一步到位**：`g++ main.cpp -o app`
- **只编译不链接**：`g++ -c main.cpp` (生成 `main.o`)
- **包含头文件路径**：`g++ main.cpp -I/path/to/include -o app`
- **分步查看**：
    - `-E`：预处理后的结果
    - `-S`：生成的汇编代码

---

## 2. 预处理与条件编译

>[!warning]
>**宏**是在编译开始阶段预处理，进行纯文本替换，错误难以追踪
>所以不要随便用宏去定义常量，这没有意义，而且不好调试
>比如说`{cpp}  #define PI 3.14` 就是非常不好的习惯
>正确的方式是`{cpp} const int PI = 3.14`


- **条件编译**：用于跨平台适配、调试开关等 。
    - 指令：`#ifdef`, `#ifndef`, `#if`, `#else`, `#endif` 。

- **头文件保护 (Header Guard)**：防止重复包含 。
    - 标准做法：`#ifndef HEADER_H`, `#define HEADER_H`, ...内容..., `#endif` 。

- **内置宏**：`__FILE__` (文件名), `__LINE__` (行号), `__DATE__` (日期), `__TIME__` (时间) 。
- **命令行定义宏**：`g++ main.cpp -DDEBUG` (相当于在代码中 `#define DEBUG`) 。

---

## 3. Makefile 速查表

**Makefile** 用于自动化多文件编译，通过检测文件修改时间来决定是否重新编译 。

**语法：** ```
```
[target] : [prerequisition]
	[command]
```

**核心架构：**

```bash
# 变量定义
CXX = g++
OBJS = main.o fun.o
TARGET = program

# 显式规则
$(TARGET): $(OBJS)
	$(CXX) -o $(TARGET) $(OBJS)

# 模式规则 (简化推导)
%.o: %.cpp
	$(CXX) -c $< -o $@

# 伪目标
clean:
	rm -f $(TARGET) $(OBJS)
```

- **核心指令**：直接输入 `make` 运行第一个目标；`make clean` 执行清理 。
- **特殊变量**：`$@` (目标文件), `$<` (第一个依赖文件), `$^` (所有依赖文件) 。

---

## 4. CMake 速查表

**CMake** 是 Makefile 的“翻译官”，用于跨平台构建 。

**核心流程 (现代标准流)** ：

1. **配置 (Configure)**：`cmake -S . -B build` (指定源码在当前目录，构建在 build 文件夹) 。
2. **编译 (Build)**：`cmake --build build -j 4` (使用 4 个 CPU 并行编译) 。
3. **安装 (Install)**：`cmake --install build --prefix /usr/local` 。

**CMakeLists.txt 基础指令：**

- `cmake_minimum_required(VERSION 3.10)`：版本要求 。
- `project(MyProject)`：项目名称 。
- `set(CMAKE_CXX_STANDARD 11)`：设置 C++ 标准 。
- `add_executable(app main.cpp)`：生成可执行文件 。
- `add_subdirectory(src)`：添加子目录模块 。
- `target_link_libraries(app lib1)`：链接库文件 。

---

## 5. 其他构建工具与补充

- **Configure 工作流**：`./configure` -> `make` -> `make install` 。
- **可视化依赖图**：`cmake -B build --graphviz=deps.dot` 配合 Graphviz 工具生成依赖关系图 
- **命令行传参**：`cmake -DENABLE_FEATURE=ON ..` (覆盖 `CMakeLists.txt` 中的设置) 。

## 6.头文件
[[头文件的引用]]