这份笔记为你提炼成了**软件测试与 GoogleTest (gtest) 速查表**。重点保留了框架逻辑、核心语法、断言清单以及常用指令，略去了具体的背景故事和复杂的项目举例，方便你随时查阅和复习。

---

### 一、 软件测试核心理念

- **测试的本质是“破坏”与“证伪”**：测试无法证明程序没有 Bug，只能证明 Bug 的存在（Dijkstra 提出）。

- **心理学误区**：不要为了“证明代码正确”去测试，而应一开始就假设代码有错，并试图找出它。

- **好的测试应具备的条件**：
    
    1. **独立且可重复**：测试之间互不干扰。
    2. **结构清晰**：与被测代码结构相呼应（如使用测试套件 TestSuite）。
    3. **可移植/复用**：跨平台兼容。
    4. **失败信息丰富**：遇到非致命错误不中断，继续跑完并汇报所有错误。
    5. **低管理成本**：框架自动追踪测试，无需手动枚举。
    6. **快速且节省资源**：共享初始化/清理过程。

---

### 二、 测试分类与工程体系

- **黑盒测试**：关注功能实现，内部逻辑不可见。
- **白盒测试**：关注内部结构、分支和路径（**单元测试**属于此类）。
- **灰盒测试**：黑白结合（如集成测试）。
- **CI/CD (持续集成与持续部署)**：
    - **CI (Continuous Integration)**：代码提交后自动触发编译和自动化测试，尽早发现 Bug。
    - **CD (Continuous Deployment)**：测试通过后自动化发布/上线。

---

### 三、 GoogleTest (gtest) 基础使用

#### 1. 编译与运行

- **编译指令**：需要链接相关的库。
```bash
    g++ 01_test.cpp -lgtest -lpthread -lgtest_main
```
_(注：包含了 `-lgtest_main` 就可以省略自己写 `main` 函数)_

#### 2. 基本测试宏：`TEST`

用于普通、相互独立的测试用例。
```c++
TEST(TestSuiteName, TestName) {
    // 测试逻辑与断言
    EXPECT_EQ(1, 1);
}
```

- **注意**：`TestSuiteName` 和 `TestName` 命名中**绝对不要包含下划线 `_`**，推荐使用首字母大写（驼峰命名法）。

#### 3. 测试固件宏：`TEST_F`

用于需要**复用准备工作**（如共享对象、初始化相同数据）的场景。

- **步骤 1：定义 Fixture 类**（继承自 `testing::Test`）。
    - 使用 `protected:` 权限存放共享数据（以便 `TEST_F` 访问）。
    - 重写 `SetUp()`：每个测试执行**前**自动调用。
    - 重写 `TearDown()`：每个测试执行**后**自动调用。

- **步骤 2：编写测试**
```c++
    TEST_F(Fixture类名, 测试用例名) {
        // 直接使用 Fixture 类中的 protected 成员变量/函数
    }
```
#### 4. Mock（模拟）

用于测试依赖了某个“重型类”，但不想实际编译或运行那个类时，构建一个“假的接口”。

- 引入头文件：`#include <gmock/gmock.h>`
- 使用宏：`MOCK_METHOD(返回类型, 方法名, (参数列表));`
- 设定预期行为：`EXPECT_CALL(...).WillOnce(::testing::Return(预期值));`

---

### 四、 gtest 断言速查表 (Assertions)

- **`ASSERT_*` (致命断言)**：失败时立刻**终止**当前函数。
- **`EXPECT_*` (非致命断言)**：失败时记录错误并**继续执行**当前函数（**推荐默认使用**）。
- **自定义报错信息**：在断言后追加流操作符 `<< "你的报错信息"`。
#### 1. 布尔与数值比较

|**致命断言 (ASSERT)**|**非致命断言 (EXPECT)**|**验证条件**|
|---|---|---|
|`ASSERT_TRUE(condition)`|`EXPECT_TRUE(condition)`|条件为 true|
|`ASSERT_FALSE(condition)`|`EXPECT_FALSE(condition)`|条件为 false|
|`ASSERT_EQ(val1, val2)`|`EXPECT_EQ(val1, val2)`|等于 (==)|
|`ASSERT_NE(val1, val2)`|`EXPECT_NE(val1, val2)`|不等于 (!=)|
|`ASSERT_LT(val1, val2)`|`EXPECT_LT(val1, val2)`|小于 (<)|
|`ASSERT_LE(val1, val2)`|`EXPECT_LE(val1, val2)`|小于等于 (<=)|
|`ASSERT_GT(val1, val2)`|`EXPECT_GT(val1, val2)`|大于 (>)|
|`ASSERT_GE(val1, val2)`|`EXPECT_GE(val1, val2)`|大于等于 (>=)|

#### 2. 浮点数比较（重点：避免精度误差）

|**致命断言 (ASSERT)**|**非致命断言 (EXPECT)**|**验证条件**|
|---|---|---|
|`ASSERT_FLOAT_EQ(v1, v2)`|`EXPECT_FLOAT_EQ(v1, v2)`|float 类型极小误差内相等|
|`ASSERT_DOUBLE_EQ(v1, v2)`|`EXPECT_DOUBLE_EQ(v1, v2)`|double 类型极小误差内相等|
|`ASSERT_NEAR(v1, v2, err)`|`EXPECT_NEAR(v1, v2, err)`|两者差的绝对值 <= err (自定义阈值)|

#### 3. 字符串比较 (C string)

|**致命断言 (ASSERT)**|**非致命断言 (EXPECT)**|**验证条件**|
|---|---|---|
|`ASSERT_STREQ(s1, s2)`|`EXPECT_STREQ(s1, s2)`|字符串内容相等|
|`ASSERT_STRNE(s1, s2)`|`EXPECT_STRNE(s1, s2)`|字符串内容不相等|
|`ASSERT_STRCASEEQ(s1, s2)`|`EXPECT_STRCASEEQ(s1, s2)`|忽略大小写相等|

#### 4. 异常与死亡测试

|**致命断言 (ASSERT)**|**非致命断言 (EXPECT)**|**验证条件**|
|---|---|---|
|`ASSERT_THROW(stmt, type)`|`EXPECT_THROW(stmt, type)`|语句抛出指定 type 类型的异常|
|`ASSERT_ANY_THROW(stmt)`|`EXPECT_ANY_THROW(stmt)`|语句抛出任意异常|
|`ASSERT_NO_THROW(stmt)`|`EXPECT_NO_THROW(stmt)`|语句不抛出异常|
|`ASSERT_DEATH(stmt, regex)`|`EXPECT_DEATH(stmt, regex)`|**死亡断言**：语句导致程序崩溃退出，且错误信息匹配正则|

---

### 五、 命令行与 CMake 进阶指令

#### 1. gtest 可执行文件常用参数 (`./a.out`)

- `--gtest_list_tests`：列出所有测试但不运行。
- `--gtest_filter=Positive.*`：**选择性运行**（只跑 Positive 套件下的测试）。支持通配符 `*`、`?`，使用 `:` 分隔，使用 `-` 排除（如 `--gtest_filter=-Positive.TestB`）。
- `--gtest_output=xml:report.xml`：输出 XML 或 JSON 格式的测试报告。
- `--gtest_repeat=10`：重复运行测试 10 次。
- `--gtest_break_on_failure`：遇到失败直接进入调试器断点。

#### 2. CMake 结合 CTest

在 `CMakeLists.txt` 中配置：
```CMake
enable_testing()
add_test(NAME AllTests COMMAND run_tests) # run_tests为生成的可执行文件
```

在 build 目录下触发测试的终端指令：

- `make test`：运行测试（默认简略输出）。
- `make test ARGS="-V"`：运行并输出详细日志。
- `make test ARGS="-R Math"`：只运行名字含 "Math" 的测试。
- `ctest --verbose --repeat until-pass:3`：重复运行直到通过（最多 3 次，查偶发 Bug）。

---

### 六、 测试驱动开发 (TDD - Test-Driven Development)

- **三项核心法则**：
    1. **红 (Red)**：禁止编写未失败的业务代码（先写一定会报错的测试用例）。
    2. **绿 (Green)**：仅编写刚好能让当前失败的测试通过的**最小代码**（杜绝过度设计）。
    3. **重构 (Refactor)**：测试通过后，在保障测试依然绿色的前提下，优化和精简代码结构。

- **核心优势**：极早发现 Bug 降低成本；倒逼代码高内聚低耦合；为未来的重构和修改提供安全保障。