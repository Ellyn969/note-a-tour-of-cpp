# Chapter 1--基础知识

# 程序
* 编译型语言
* source code -> **compiler** -> object code -> **linker** -> executable file
* executable file: 特定硬件和操作系统 （不可移植）
    * cpp程序的可移植性：源代码可移植

* 声明**标准库**：
    ```cpp
    #include <iostream> // 声明输入输出标准库
    ```

* 静态类型语言
    * 变量类型在**编译时**确定：`int x=3;`
    * 变量类型不可变
    * python等：动态类型：运行时确定

# 函数
* 函数示例：
    ```cpp
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
    * `x&&y` : 逻辑与
    * `x|y` : 逻辑或
    * `x^y` : 按位异或：不同为1
    * `~x` : 按位求补：每位取反
* 初始化的讲究
    * `=` or `{}`
        * `{}` 更安全，防止窄化转换
        ```cpp
        int x1 = 3.5; // x1 == 3, 自动承受窄化结果
        int x2{3.5};  // error: narrowing conversion：报错
        ```
    * 可以不显式初始化的情况：
        * 变量类型可以自动推断
        ```cpp
        auto a=3
        ```

# 作用域和生命周期
* 当声明一个变量：引入作用域
    * 局部作用域
        * 函数内部
    * 类作用域
    * 命名空间作用域

# 常量
1. `const`:
    * 值在**声明**后不能改变
    * 但**初始化**可以在编译时 或 运行时
    * e.g.,
        ```cpp
        const int x = 3; // 编译时
        const int y = sqrt(16); // 运行时
        ```
    * 定义常量 `const double Pi = 3.14;`
    * 用作函数参数：在函数**内部**不会被修改
        * 但调用函数可以传入不同的值
            ```cpp
            void some_function(const int x);
            some_function(5);
            some_function(6);
            ```
2. `constexpr`
    * 值必须在**编译时**确定
    * e.g.,
        ```cpp
        constexpr int x = 3; // 编译时

        constexpr int b = some_function(16);  // constexpr函数：在编译时求值；普通函数：运行时求值
        ```
    * 为了使一个函数可在常量表达式中使用：必须声明为/定义为 `constexpr` 函数，且要满足一定条件
        ```cpp
        constexpr int square(int x) {
            return x * x;
        }
        ```
        * 只能使用输入参数作为信息；不能修改非局部变量
    * 编译时配置，如数组大小
        ```cpp
        constexpr int size = 100;
        int arr[size];
        ```


# 指针、数组和引用
1. 数组：  
    * zero-based indexing
2. 指针：
    * **【`&`取地址；`*`取内容】**
        ```cpp
        char* p = &v[4]; // p指向v[4]的地址
        char x = *p; // x 或 *p, 代表指针p指向的内容
        ```
    * 空指针`nullptr`
        * `int* p = nullptr;`
        * 试图确保指针永远指向有效对象（解引用`*`有效）
        * **使用指针，一般都先检查指针是否为空**
    * `++`用于指针：移动到下一个元素
        ```cpp
        if (p==nullptr):
            return 0;
        for (; *p!=0; *p++)
        // 或
        while (*p != 0):
            // 省略
            ++p;
        ```
    * 数组和指针：
        ```cpp
        int arr[5] = {1,2,3,4,5};
        int* p = arr; // 数组[首元素]
        p[1] // 访问第二个元素，等价于 *(p + 1)：【基于当前指针位置】
        ```
    
3. 引用：
    ```cpp
        int x = 5;
        int& ref = x; // ref是x的引用
        ref = 10; // 修改ref，x也变为10
        ref = 10; // 修改ref，x也变为10
    ```
    * 对比指针：
        * 无需解引用`*`就能访问值
        * 引用不能为空（必须指向有效对象）
        * 引用必须在声明时初始化（指针可以先声明后赋值）
    * **引用初始化之后就不能再指向其他的【对象】**，但可以通过引用修改对象的值
        ```cpp
        // naive version
        void print(){
            for (auto x : v): // 对于每个元素：都复制一份到x并输出
                std::cout << x << std::endl;
        }
        ```
        ```cpp
        // ref. version
        void print(vector<int>& v){ // 使用引用作为函数参数，避免拷贝开销
            for (auto x : v):
                std::cout << x << std::endl; // 只传递引用（地址），不复制数据
        }
        ```
    
* 指针、引用和const:
    * `const int* p`: 指向常量int的指针：即p指向的**内容**不可修改，但p本身可以指向其他地址
    * `int* const p`: 常量指针：p本身不可修改（必须指向**同一地址**），但p指向的**内容**可以修改
    * `const int* const p`
    * `const int& r`: 指向常量int的引用：r引用的**内容**不可修改(指向的地址本来就不能修改)


# 检验
* switch语句
    * **用于检查一个值是否存在于一组常量中**
        * case标签必须是常量表达式
    * 只能用于整数类型（包括字符类型char）
    * `default`: optional
* if语句：
    * 在if语句条件内定义变量：
        ```cpp
        if (int x = v.size()) // x非0为true
        ```
        ```cpp
        if (int x = v.size(); x > 0) // x>0为true
        ```
        * 将变量x的作用域限制在if语句内，用x返回的值作为条件判断
    


# c++: 映射到硬件
* 指针 = 内存地址数值
    * `p+1 = p + sizeof(T)`
    * `*p`: 读/写该地址
* 数组 = 连续内存块
    * `arr[i] = base_address + i * sizeof(T)`
    * 无边界检查：越界访问后果严重，UB
* ...