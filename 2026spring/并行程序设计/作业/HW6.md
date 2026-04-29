1. 明确交付物（你需要提交什么）

- 你要写的是“单元测试”，测试对象是 realArray 这一类（在 realarray.h 里）。
- 一般需要交：
  - realarray_test.cpp （按作业要求的命名规则： {class_name}_test.cpp ）
  - compile_test.sh （一键编译测试）
  - run_test.sh （一键运行测试）
- 你可以参考仓库里已有的“非 gtest 冒烟测试”目录： test_realarray （里面的 main.cpp/compile.sh 只是示例，不是 gtest 版）。
2. 先把接口与行为“读懂并列清单”（这一步决定你测什么）

- 打开并通读：
  - realarray.h
  - realarray.cpp
- 同时用这份任务清单当作“测试需求来源”（它把常用功能分了 1–10 步）： homework_realarray.md
- 你需要把接口按模块拆成测试点（建议至少覆盖）：
  - 构造：默认构造、3D 构造、4D 构造、拷贝构造
  - 重分配： create(3D) 、 create(4D)
  - 访问： operator()(i,j,k) 与 operator()(i,j,k,l) （含 const/non-const）
  - 批量操作： operator=(double) 、 zero_out()
  - 元信息： getSize/getDim/getBound1..4
  - 可选： zeros(T* u,int n) 、 getArrayCount() （注意它的语义很“怪”，见第 6 步提示）
3. 选一个 GoogleTest 安装/引入方式（决定你的编译脚本怎么写） 你有两条常见路线，选其一即可：

- 方案 A：系统已安装 gtest（最省事，前提是你机器能用）
  - 编译时链接： -lgtest -lgtest_main -pthread
- 方案 B：把 googletest 作为第三方源码放进项目（最稳，不依赖系统）
  - 常见做法：在项目里放 third_party/googletest/ ，编译时直接把 gtest 源码一起编进去，或用 CMake 构建后链接静态库
  - 你的 compile_test.sh 里需要体现你用的路径与编译方式
4. 放置文件与目录（建议你这样组织，避免和仓库现有结构打架）

- 建议位置：继续沿用仓库的测试目录风格，把你的作业文件放在
  - source/source_base/test_realarray/realarray_test.cpp
  - source/source_base/test_realarray/compile_test.sh
  - source/source_base/test_realarray/run_test.sh
- 好处：和其他模块（ test_vector3/test_matrix/... ）一致，老师/助教也更容易跑。
5. 设计你的 gtest 用例（从“功能覆盖”到“边界覆盖”） 把 homework_realarray.md 的 1–10 任务“翻译成断言”，每个任务拆成 1 个或多个 TEST ：

- 默认构造（Task 1）
  - 断言： getDim() 、 getSize() 、各 getBound*() 是否符合实现
- 3D/4D 构造（Task 2/3）
  - 断言：维度、size=各维乘积、bounds 正确
  - 边界：传 0/负数维度时会被钳制到 1（实现里有处理），要加一条测试验证
- create()（Task 4/5）
  - 断言：create 后 dim/bounds/size 更新正确；并且之前写入的数据不应被“错误保留”（通常会重置内存，至少你应重新填充值再测）
- 元素访问与赋值（Task 6/10）
  - 选一个很小的维度（如 2×2×2 或 2×2×2×2）
  - 用嵌套循环写入唯一值（例如 value = 100*i + 10*j + k ），再逐点读出做 EXPECT_DOUBLE_EQ
- 标量赋值（Task 7）
  - a = 5.5 后逐元素检查全等
- zero_out（Task 8/10）
  - 先填非零，再 zero_out() ，逐元素检查为 0
- 拷贝构造（Task 9）
  - 拷贝后：内容相等；修改原对象后，拷贝对象不变（验证“深拷贝”）
6. 处理“危险/特殊行为”（避免把测试写成不稳定的随机炸弹）

- 越界访问： operator() 内部用 assert 做越界检查
  - 如果你要测越界，建议用 gtest 的 death test（如 EXPECT_DEATH ）
  - 注意：如果编译时定义了 NDEBUG ，assert 会被关掉，death test 就失效；你的 compile_test.sh 里不要加 -DNDEBUG
- 拷贝赋值 operator=(const realArray&) ：实现只按“左侧 size”循环拷贝，不检查右侧 size
  - 不要写“左右维度不同”的赋值测试（这会触发未定义行为：可能崩溃也可能悄悄错），除非你明确用 death test 并能稳定复现
- getArrayCount() ：构造会 ++，析构没有 --
  - 不要断言“它等于当前存活对象数”；最多只做“单调递增”这类弱断言，或者干脆不测
7. 写 compile_test.sh（把 realarray.cpp 和你的测试一起编成一个可执行文件）

- 你的编译输入至少包括：
  - realarray_test.cpp
  - ../realarray.cpp （realArray 的实现）
- 如果你用方案 A（系统 gtest），脚本核心就是“g++ 编译 + 链接 gtest + pthread”
- 如果你用方案 B（源码 gtest），脚本核心就是“把 gtest 源码也编进去/或先构建库再链接”
- 产物建议命名： realarray_test （可执行文件）
8. 写 run_test.sh（统一运行入口）

- 做到一键运行即可： ./realarray_test
- 可选：加上 gtest 常用参数（例如输出更清晰、失败时返回非 0），但保持脚本尽量简单
9. 本地自检（你交之前一定要自己跑通）

- 运行 compile_test.sh ，确认没有编译/链接错误
- 运行 run_test.sh ，确认所有用例 PASS
- 如果用了 death test，再额外确认在你当前编译选项下 assert 没被关掉
10. 最终提交检查清单（交之前对照一遍）

- 文件命名符合要求： realarray_test.cpp / compile_test.sh / run_test.sh
- 测试覆盖到：构造、create、访问、标量赋值、zero_out、拷贝构造（至少这些）
- 边界覆盖至少 1–2 个：例如“维度传 0/负数被钳制到 1”、或“越界访问 death test（可选）”
- 助教拿到你的仓库/压缩包后：能直接 bash compile_test.sh && bash run_test.sh 跑出结果