# 程序

* 编译型语言
* source code -> **compiler** -> object code -> **linker** -> executable file
* executable file: 特定硬件和操作系统 （不可移植）

* 声明标准库：
示例：
  ```cpp
  #include <iostream> // 声明输入输出标准库
  ```


# 函数
* ```cpp
    void square(int x)
    ```
* 允许函数重载（overload）：同名函数，参数类型不同 -- 自动选择合适函数调用
    * 如果选不出：报错


# 类型，变量和运算
* declaration
* <span style="color:red">字节数 P7</span>
  * sizeof(type)
* 逻辑操作符：
    * 【按位 和 逻辑】
        * 逻辑：easy: &&, ||, !
        * 按位：二进制位运算
    * x&&y : 逻辑与
    * x|y : 逻辑或
    * x^y : 按位异或：不同为1
    * ~x : 按位求补：每位取反
* 初始化的讲究
    * `=` or `{}`
        * `{}` 更安全，防止窄化转换 (double -> int, int -> char)
        * 有`{}` ，可以不用`=` 
    * ```cpp
      int x1 = 3.5; // x1 == 3, 自动承受窄化结果
      int x2{3.5};  // error: narrowing conversion：报错
      ```
    * 可以不显式初始化的情况：
        * 变量类型可以自动推断
        * ```cpp
            auto a=3
        ```

# 作用域和生命周期
* 当声明一个变量：引入作用域
    * 局部作用域
        * 函数内部
    * 类作用域
    * 命名空间作用域

# 常量


    


