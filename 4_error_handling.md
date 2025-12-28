# Chapter 4 -- Error Handling

# Exceptions异常
* `throw`语句
    * 当程序检测到一个无法在函数内部解决的错误（如数组越界）：立即停止当前函数的执行，并**抛出一个异常对象**
    * ```cpp
        double& vector::operator[](int i) {
            if (i<0 || i>=size())
                throw out_of_range("Vector::operator[]: index out of range");
            return elem[i]; // 如果上面throw了，这行代码不会被执行
        ```
        * 创建异常，并转移异常的控制权到直接或间接调用该函数的用户
        * `out_of_range`：标准库`<stdexcept>`中定义的异常类
        * 当`throw`被执行：编译器会**退出当前作用域**，**回溯函数的调用栈**（试图在调用处的上下文寻找一个`try...catch`块来处理异常），如：
            ```cpp
            void f(Vector& v) {
                // ...
                try {
                    Vector v2 = compute(v); // 假设compute函数抛出out of range异常 --> 退回到调用compute函数的f函数 --> 利用f函数中的try块处理异常
                }
                catch (out_of_range& e) { // 捕获异常
                    cerr << "Caught an exception: " << e.what() << '\n';
                }
            }
            ```
        * 在此过程中析构函数的调用：
            * recall, 捕捉异常，跳出当前作用域
            * 如果`compute`函数定义了局部变量，这些变量会在跳出`compute`函数时被销毁 --> 即，调用它们的析构函数（否则，异常直接跳走而不清理内存，这部分的内存将永远拿不回来）
        * `out_of_range& e`：通过引用捕获异常对象，避免拷贝开销
        * `cerr`：标准错误输出流
        * `e.what()`
        * **不要过多使用`try`语句**
            * 可以通过RAII和约束条件来减少异常的使用

# 约束条件invariant
* 约束条件：程序中必须始终保持为真的条件
* **函数的前提条件/基本假定**
    * 例：数组下标必须在合法范围内
* **类约束条件(约束条件)**
    * 如果类`Vector`的约束条件是:`elem`所指向的数组的大小始终等于`sz`
    * 类构造函数：为类建立约束条件


# 错误处理的其他方法
* 设计异常机制目的：提供一个**将错误检测和错误处理分离**的方法
* 异常:
    * 用于报告在当前函数内部无法处理的错误
    * **与构造函数和析构函数配合使用**（即**RAII技术**），确保资源的正确管理
## 1. 抛出异常throw exceptions
* 最通用的方法
* 不需要手动检查返回值
    * 如果出错，程序会自动跳转到最近的 `try-catch` 块
    * 如果没有，程序会直接终止
    * **--> 错误不可能被忽略 / 强制调用者处理错误**
* `#include <stdexcept>`
* 建议在以下**不得不由上层处理 或 语法限制**的情况下使用：
    * 失败是罕见且不可预期的（**意外情况**）
        * e.g., 内存耗尽
    * **错误无法在当前函数内处理，需要层层上报**
        * e.g., 深层嵌套的函数调用
        * 如果使用返回错误代码：必须在每一层都检查返回值，代码臃肿且易出错
    * 构造函数失败：
        * 构造函数没有返回值，无法通过返回错误代码报告错误
    * 运算符重载失败：
        * 无法在数学表达式中插入错误处理代码
    * 需要撤回操作
## 2. 返回错误代码return error codes
* 老做法
* 示例：
    ```cpp
    bool safeDivide(double a, double b, double& result) {
    if (b == 0) {
        return false; 
    }
    result = a / b;
    return true;  
    }
    ```
* 建议在以下属于**常规流程或受限环境**的情况下使用（而不是抛出异常）：
    * **失败是常见的且可预期的（不属于“意外”）**
        * e.g., 文件不存在
    * 期望调用者立即处理
        * e.g., 用户输入错误
        * 调用者可以通过检查返回值立即处理错误
    * 并行任务
        * 需要知道具体哪个任务失败
    * 资源受限的环境
        * e.g., 嵌入式系统
        * 在极低内存环境下，支持异常处理的开销可能过大
## 3. 终止程序terminate/exit/abort
* 极端的错误处理方式
* 当错误无法恢复/系统状态已损坏
* 调用`std::terminate()`、`std::exit()`或`std::abort()`
* 当一个被声明为`noexcept`的函数抛出异常时，程序会自动调用`std::terminate()`    


# 断言assert
* 在代码编写/编译阶段就发现错误
## 1. 运行时断言：`assert()`
* 在**运行时**断言某个条件必须满足
    * *处理绝不应该发生的事
    * 如果发生了，那就是有bug
* `assert(condition)`：检查`condition`是否为真
    * 示例：
        ```cpp
        #include <cassert>

        double safeDivide(double a, double b) {
            assert(b != 0 && "Denominator must not be zero");
            // ...
        }
        ```
* 只在debug模式下工作
    * 如果为假：程序打印错误信息（文件名和行号）并终止
    * release模式下：`assert`语句被忽略
* 简单粗暴，不够灵活
## 2. 静态断言：`static_assert()`
* 在**编译时**断言某个条件必须满足
* `static_assert(condition, message)`：检查`condition`是否为真
    * 示例：
        ```cpp
        static_assert(sizeof(int) == 4, "This code requires 4-byte integers");
        ```
* 错误在编译时被捕获，如果检查不通过，错误不会被带到运行时
## 3. `noexcept`关键字
* 用于指示编译器和调用者：函数不会抛出异常
* `void f() noexcept { ... }`
* 编译器知道函数不会抛出异常，从而生成更高效的代码
* 强制终止：
    * 如果`noexcept`函数抛出异常，程序会调用`std::terminate()`并终止
* 不要滥用`noexcept`：
    * 只有在**绝对确定**函数不会抛出异常时才使用
    * 否则，可能导致程序意外终止

# 小结
* 尽量在编译器发现错误
* 运行时错误优先用异常
* 只有在预期的轻微错误才使用返回值