## 一、 🛠️ 函数基础 (Functions)

函数是一段只有在被调用时才会运行的代码 。

- **标准语法：** `返回类型 函数名(参数类型 参数名);` （例如：`int max(int x, int y);`）

- **声明 vs 定义：**
    - **声明 (Declaration/Prototype)：** 告诉编译器函数的名字、参数和返回值类型，通常放在 `.h` 头文件中，以分号 `;` 结尾 。(注：声明时可以省略形参名 )。
    - **定义 (Definition)：** 函数的具体实现代码（`{}` 里的内容），通常放在 `.cpp` 源文件中 。

- **命名空间 (Namespace)：** 
	- **作用：** 避免大型项目中的命名冲突 。
    - **定义：** `namespace 空间名 { ... }` 。
    - **调用：** 使用 `空间名::函数名()`，或者在文件开头声明 `using namespace 空间名;` 。
    - **嵌套：** 命名空间可以相互嵌套调用 。

---

## 二、 📦 面向对象核心概念 (OOP Basics)

面向对象编程（OOP）旨在通过编程实现现实世界的实体及其关系 。

| **核心概念**               | **简要说明**                                  |
| ---------------------- | ----------------------------------------- |
| **类 (Class)**          | 对象的“模板”或“蓝图”，定义了数据属性和操作方法 。               |
| **对象 (Object)**        | 类的具体实例化结果（定义类时不分配内存，实例化对象时才分配） 。          |
| **封装 (Encapsulation)** | 隐藏内部细节，只暴露接口。保护数据安全，防止意外修改 。              |
| **抽象 (Abstraction)**   | 提取核心特征，忽略底层实现细节（例如直接调用 `pow()` 而不关心其算法） 。 |
| **继承 (Inheritance)**   | 子类复用父类代码，并可扩展专属功能 。                       |
| **多态 (Polymorphism)**  | 同一接口，不同实现，提高代码灵活性 。                       |

---

## 三、 🏛️ 类的构建与成员 (Class Anatomy)

### 1. 访问限定符 (Access Modifiers)

控制类成员的可见性 ：
- `public`：公开的，任何地方（类外、对象）都可以直接访问 。
- `private`：私有的，只能被**类内部**的成员函数（或友元）访问 。(通常用来封装数据，外界只能通过 public 的函数来修改它们 )。
- `protected`：受保护的，类内部及其**子类**可以访问，外部不能访问 。
### 2. 构造函数与析构函数 (Constructor & Destructor)

| **类型**   | **语法标识** | **核心特点**                                                           |
| -------- | -------- | ------------------------------------------------------------------ |
| **构造函数** | `类名(参数)` | 与类同名，**无返回类型**。创建对象时**自动调用**，用于初始化变量 。分为默认、参数化、拷贝构造三种 。            |
| **析构函数** | `~类名()`  | 前加波浪号 `~`，**无参数无返回**。对象生命周期结束（超出作用域）时**自动调用**，用于释放内存（如 `delete`） 。 |
#### A. 释放动态内存 (The `delete` Pattern)
如果你在构造函数里 `new` 了，这里必须 `delete`。
```cpp
~MyClass() {
    if (ptr != nullptr) {
        delete ptr;   // 释放单个对象
        ptr = nullptr; // 良好的习惯：重置为悬空指针
    }
    // 或者如果是数组
    delete[] arrayPtr;
}
```
#### B. 释放系统资源

例如你在类里打开了文件流（虽然 `fstream` 会自理，但有时我们需要手动控制）：
```cpp
~FileHandler() {
    if (file.is_open()) {
        file.close();
    }
}
```

#### 从构造到析构：留言板类实例
```cpp unwrap:true icon:true title:MessageBoard.cpp
#include <iostream>
#include <string>

class MessageBoard {
private:
    static int total_boards;  // 静态成员：统计存活的对象数量
    std::string* messages;    // 指针：准备在堆上申请数组
    int size;                 // 数组大小

public:
    // 【构造函数】：在这里进行 new 操作
    MessageBoard(int n) {
        size = n;
        messages = new std::string[size]; // 1. 按需申请堆内存
        total_boards++;                    // 2. 更新全局计数
        std::cout << "构造：申请了 " << size << " 条留言空间。当前板数：" << total_boards << std::endl;
    }

    // 【析构函数】：在这里进行 delete 操作
    ~MessageBoard() {
        if (messages != nullptr) {
            delete[] messages;        // 1. 释放对应的堆内存（注意是数组 delete[]）
            messages = nullptr;       // 2. 指针置空，防止野指针
        }
        total_boards--;               // 3. 更新全局计数
        std::cout << "析构：释放了内存。当前剩余板数：" << total_boards << std::endl;
    }

    // 静态成员函数：查看全局状态
    static int getCount() {
        return total_boards;
    }
};

// 静态变量必须在类外初始化
int MessageBoard::total_boards = 0;

int main() {
    std::cout << "开始时板数：" << MessageBoard::getCount() << std::endl;

    {
        // 在大括号（作用域）内创建对象
        MessageBoard myBoard(5); 
        std::cout << "执行中..." << std::endl;
    } // <--- 这里超出作用域，析构函数会被【自动调用】

    std::cout << "结束时板数：" << MessageBoard::getCount() << std::endl;
    return 0;
}
```
### 3. 静态成员与 this 指针 (Static & This)

- **`this` 指针：** 指向当前调用该成员的**对象实例**，仅在类的**非静态**成员函数内有效 
- **静态成员变量 (`static`)：** 所有对象共享的**唯一副本**。无需创建对象也可通过 `类名::变量名` 调用 。
>[!important]
>所谓的**共享唯一副本**指的是我的**所有的类实例拥有的这个静态成员都共享**
>这样可以方便的**全局统计和状态共享**
>而**静态函数不需要实例化，直接调用**，是专属工具包

- **静态成员函数：** 没有 `this` 指针，只能访问类的其他**静态**成员，不能访问普通成员变量 。
```cpp
class MathHelper { 
	public: 
		static int add(int a, int b) {
			 return a + b; 
			 } 
		}; 
// 调用方式： 

int result = MathHelper::add(5, 10);
```

---

## 四、 ➕ 运算符重载 (Operator Overloading)

在不改变原始含义的情况下，赋予现有运算符特殊的、针对自定义类的计算能力（编译时多态） 。

- **常见重载场景：** 矩阵相乘 `*=`、打印输出 `<<`、通过括号取值 `()` 。
- **⚠️ 避坑指南（内存管理）：**
    - 如果在类中使用了 `new` 动态分配内存（如指针数组） ，**必须**在析构函数中使用 `delete[]` 释放内存，否则会导致**内存泄漏** 。
    - **浅拷贝风险：** 如果类内部有指针，直接赋值会导致两个对象指向同一块内存。必须重写“拷贝构造函数”和“赋值运算符 (`operator=`)”来实现**深拷贝** 。

---

## 五、 🔀 结构化编程控制流 (Control Flow)

禁止随意使用 `goto`，使用三种基本结构代替混乱跳转 。
#### 1. 循环 (Loops)
用于在特定条件下重复执行代码 ：

|**循环类型**|**语法结构**|**执行逻辑**|
|---|---|---|
|**for**|`for(初始化; 条件; 迭代更新) { ... }`|适合已知循环次数。先判断条件，再执行循环体 。|
|**while**|`while(条件) { ... }`|适合未知次数。**先判断条件**，为真才执行 。|
|**do-while**|`do { ... } while(条件);`|**先执行一次循环体**，然后再判断条件是否继续 。|

#### 2. 分支 (Branches)
用于检查表达式以决定执行路径 ：
- **if / else：** 基础条件判断。
- **switch：** 针对同一变量的多个离散值进行匹配跳转。