# Chapter 2 -- 用户自定义类型 User-defined Types UDT

# Overview
* 内置类型built-in types
    * 基本类型，const, 声明操作符

* **UDT**
    * struct
    * **class**
    * union
    * **enum (enumeration枚举)**

# struct
* 将所需元素组合成一种数据结构
    ```cpp
    struct Vector { // 用作举例，实际使用标准库vector
        double* elem; // 指向【元素】的指针
        int sz; // vector大小
    };

    Vector v;
    ```
* 访问struct成员: `.`操作符
    * 可通过名字/引用/指针访问
        ```cpp
        v.elem; // 通过名字访问
        Vector& rv = v;
        rv.sz; // 通过引用访问

        Vector* pv = &v;
        pv->elem; // 通过指针访问
        // 通过指针访问：【使用`->`操作符】
        ```
* 函数示例：
    ```cpp
    void vector_init(Vector& v, int s){
        // 使Vector指针指向实际内容
        v.elem = new double[s]; // 分配数组空间：包含s个double类型的值
        v.sz = s;
    }
    ```
    * `new`操作符：
        * 从自由存储空间区域**分配内存**
        * (分配在自由存储区的对象会)一直存活，直到`delete`释放
* **数据和对应操作分离**，成员默认都是public


# class
* 对比struct: struct默认的成员访问权限是public，而**class默认是 private** (唯一语法区别)
* 将类型的**接口(public)**和对应**实现(private)**分离开来
* 构造函数：
    * 和类同同名
    * 用来构造类对象
        ```cpp
        class Vector {
        public:
            Vector(int s): elem(new double[s]), sz(s) {}; // 初始化列表
            double& operator[](int i) {return elem[i];}; // 通过下标访问
            int size() { return sz; }
        private:
            double* elem;
            int sz;
        };
        ```
    * **构造函数初始化列表** `Vector(int s): elem(new double[s]), sz(s) {};` 【**初始化**一步到位】
        * 冒号`:`分割构造函数和**初始化**列表
        * **构造函数成员直接初始化 `member(args)`**
            * 一步到位，避免了默认初始化再赋值的开销
        * **构造函数内赋值**的版本：
            ```cpp
            Vector(int s){
                elem = new double[s]; // 但这里是：先默认初始化elem，再赋值
                sz = s;
            };
            ```
        * 对于 **`const`和引用成员**，**只能使用初始化列表进行初始化**
            * recall, **构造函数内部操作是赋值，不是初始化**
           * `const`成员：只能初始化，不能赋值
           * 引用成员：必须在初始化时绑定到某个对象
                * 示例：
                    ```cpp
                    class A {
                        int& ref;

                    public:
                        A(int& r): ref(r) {} // 正确，ref在初始化列表中被初始化
                        """
                        A(int& r) { 
                            ref = r;  // 错误
                        } 
                        """
                    };
                    ```
        * 初始化列表的**执行顺序**：
            * 按成员在类中**声明的顺序**执行，而不是初始化列表中的顺序
            * 示例：
                ```cpp
                class A {
                    int x;
                    int y;
                public:
                    A(int a, int b): y(b), x(a) {} // 虽然y在前，但x会先被初始化
                };
                ```
            * --> **初始化列表的顺序应当与成员声明顺序一致**
        * **优先使用初始化列表**

# enum
* 用于枚举一系列值
* 示例：
    ```cpp
    enum class Color {red, green, blue}; 
    enum class Traffic_light {red, yellow, green};
    ```
    * `enum Color {red, green, blue}; ` 也是可以的，但：
        * 枚举值在**全局作用域**内（不推荐）
            * 即：枚举值拥有和`enum`相同的作用域
        * (并非强类型) **enum值可以隐式转换为int类型**
            * 从**0**开始依次编号
            * `int col = green; // col = 1`
    * **【在`enum`后面使用`class`】**
        * 表示enum类型为**强类型枚举（strongly typed enumeration）**
            * 枚举值的作用域在各自的`enum class`内
        * 允许同一个值在不同的`enum class`中重复出现
        * 
            ```cpp
            Color col=Color::red; 
            Traffic_light light=Traffic_light::red; // 含义不同

            Color col = red // 错误，red不在全局作用域内
            // 必须写成：
            Color col = Color::red;
            // 或：
            auto col = Color::red;
            ```
    * 显式初始化enum值类型 **（默认是int）**：
        ```cpp
        enum class Color : char {red, green, blue}; 
        Color x {"red"};
        ```

# Union
* 特殊的Struct
* 所有成员共享同一块内存
    * --> 实际占用内存：最大成员的大小
    * --> 同一时刻只能保存一个成员的值
    * 节省内存
* <span style="color:red">**用法省略**</span>
    * `std::variant` 更常用
        * 一种**类型安全**的union，它能够记住当前存储的是哪种类型，并防止错误访问。
