# CHapter 5 -- Class

# Overview
* c++最核心的语言特性：类
* 用户自定义类型
* 库提供的往往就是类
* 三种重要类的基本支持：具体类、抽象类、类层次结构中的类

# 具体类
* just like built-in types
* **具体类的成员变量（实现）是其定义的一部分**
* 指**实现所有操作**、可以**直接实例化和使用**的类
    * 不同于抽象基类，用户可以直接创建具体类的对象
* **带有精致接口的资源管理器**
* 是最简单的类，**优先选择**（尤其是对性能要求高的组件）
* **值语义：对象可以放在任何地方**
    * **栈上 `Point p(1, 2);`**
        * 默认选择
        * 栈比较小（1-8MB）
        * **自动内存管理**：作用域结束自动释放
        * 性能最快：分配和释放高效
    * **堆上 `Point* p = new Point(1, 2);`**
        * 也称为**动态存储/自由存储**
        * **手动内存管理**：需要手动`delete`释放
        * 堆很大（GB级别），**可以放大对象**
        * 用于长期存活的大对象
        * **推荐使用智能指针**
    * **静态存储区 `static Point p(1, 2);`**
        * 程序启动时创建，程序结束时销毁
        * 首次使用时初始化，之后复用
        * 性能很快，一次性分配，自动管理
    * 嵌入其他对象 `Line l(p1, p2);`
* 例子：`vector`, `string`
    * **成员变量出现在具体类的每一个对象中**
    * --> 实现可以在时间和空间上达到最优
* 被限定于私有的成员变量：
    * 只能通过成员函数访问
        * recall, *成员变量出现在具体类的每一个对象中*
        * --> 如果成员变量发生明显改动：需要重新编译整个程序
* **只有当函数确实需要直接访问成员变量时，才将其定义为成员函数**


## 一种算术类型
* 例子：`complex`类 (标准库的简化版本)
    ```cpp
    class Complex {
        double re; 
        double im; 
    public:
        Complex() : re(0), im(0) {}            // 默认构造函数 {0,0}
        Complex(double r) : re(r), im(0) {}        // 单标量构造函数
        Complex(double r, double i) : re(r), im(i) {} // 双标量构造函数
        Complex(Complex const& z) : re(z.re), im(z.im) {} // 拷贝构造函数

        double real() const { return re; }
        void real(double r) { re = r; }
        double imag() const { return im; }
        void imag(double i) { im = i; }

        Complex& operator+=(Complex const& z) {
            re += z.re;
            im += z.im;
            return *this;
        }

        // 其他成员函数...
    }


    Complex operator+(Complex a, Complex b) {
        // 该函数不需要直接访问成员变量，其定义可以和类分开
        // 可以使用类的成员函数实现
        return a+=b;
    }
    ```
    * **默认构造函数**：不需要实参就可以调用的构造函数
        * 有效防止未初始化的对象
    * **拷贝构造函数**：用同类型的另一个对象初始化新对象的构造函数
        * 默认情况下，拷贝**赋值函数和拷贝初始化函数**会被隐式生成
    * 简单操作，如赋值和`+=`，不应该以函数调用形式实现
        * 定义在类内部的函数默认是`inline`的
        * 也可以在类外定义`inline`函数
    * `const`修饰符
        * in `double real() const { return re; } // 修改实部`
        * **表示该成员函数不会修改对象的状态**
        * 留意`const`的位置
        * `const`成员函数可以被`const`和非`const`对象调用
        * 但非`const`成员函数只能被非`const`对象调用
    * 传值方式：传递实参
        * 如`Complex operator+(Complex a, Complex b)`
        * 传递的是拷贝（形参），修改拷贝不会修改实参（调用者函数的实参）
* 定义操作符应当主要模仿其在内置类型中的行为

## 容器
* 容器指包含多个元素的对象
* 例如，`Vector`类
    * `Vector`的对象都是容器，因此称`Vector`为容器类
* 析构函数：
    * 确保构造函数分配的资源被正确释放/一定会被销毁
        ```cpp
        class Vector {
            public:
                Vector(int s) : elem(new double[s]), sz(s) { // 构造函数
                    for (int i=0; i!=s; ++i)
                        elem[i] = 0;  // 初始化为0
                } 

                // 析构函数
                ~Vector() {delete[] elem;}
                
            // ...
        }
        ```
    * 类名前加`~`
    * 属于构造函数的补充
    * `delete`：释放一个独立对象
    * `delete[]`：释放一个数组
* **数据句柄模型 handle-to-data model**
    * **handle句柄**：一个小对象，它持有指向实际数据（可能是大数据）的指针
        * handle可以在栈上分配， data在堆上分配
    * 例子：`Vector`类：`elem`指针式handle, 指向堆上分配的数组data
    * **常用于管理在对象生命周期中大小可变的数据**
    * 优势：
        * otherwise, 大数据必须放在栈上-->**栈溢出**
        * 避免裸`new`和`delete`的使用
            * 不使用handle: `Vector *v = new Vector(100); ... delete v; //手动管理`
            * 使用handle: `Vector v(100); ... // 自动管理，RAII`
* **资源获取初始化 RAII resource acquisition is initialization**
* **在构造函数中获取资源，在析构函数中释放资源**
    * 避免使用裸`new`和`delete`
    * 防止在普通代码中分配内存，而是隐藏在行为良好的抽象的实现内部
    * 避免资源泄露

## 容器的初始化
* 以`Vector`类为例：如何将元素存入容器：
    1. **初始值列表构造函数**
    2. **`push_back()`: 在序列的末尾添加新元素**
    * 示例：
        ```cpp
        class Vector {
        public:
            Vector(); // 默认构造函数：构建空Vector
            Vector(std::initializer_list<double> lst); // 传入初始化列表
            // ...
            void push_back(double d); // 在末尾添加
            // ...
        };
        ```
        * `std::initializer_list<double>`: `Vector v = {1, 2, 3};`使用`{}`时编译器自动创建并提供给程序
        * ```cpp
            // 初始化列表构造函数的定义
            Vector::Vector(std::initializer_list<double> lst)
            :elem(new double[lst.size()]), sz(lst.size()) {
            std::copy(lst.begin(), lst.end(), elem); // 将lst的内容复制到elem指向的数组中
            }
            ```
    * `push_back()`传入多个元素：
        ```cpp
        Vector read(istream& is){
            Vector v; // 初始化
            for (double d; is >> d; )
                v.push_back(d);
            return v;
        }
        // 这里使用for以便将d作用域限制在循环内部
        // while: double d; while (is >> d) v.push_back(d); 声明在循环外部
        ```
    * 移动构造函数
        * 返回大规模数据
        * `Vector v = read(cin);`

# 抽象类
* **将使用者和类的实现细节完全隔离开来**
    * recall, 具体类：类的实现是类的定义的一部分
    * 即，**将接口和实现完全解耦**
    * --> **放弃纯局部变量**
    * **没有构造函数**：不需要初始化数据
    * **有一个`virtual`的析构函数：虚析构函数**
* **虚函数**
    * 虚函数表`vtbl`
        * **每个类**有一个虚函数表，存储该类的虚函数**地址**
        * 如`Vector::operator[]()`; `Vector::size()`...
        * --> **调用函数的实现只需要知道Container中vtbl指针的位置以及每个虚函数对应的索引**
        * 称为**虚调用机制**
* **纯虚函数**
    * **有纯虚函数的类被称为抽象类**
    * **纯虚函数只给出函数签名，不给默认实现，具体实现延迟到派生类**
        * **抽象类不能直接实例化**
            * 用户只能通过派生类拿到具体实现
            * 同一接口可以有多种派生类实现
    * 抽象类声明**纯虚函数**，明确所有子类必须提供的能力
* **必须在自由存储空间为对象分配空间**
    * 无法确定抽象类型的具体实现（包括大小）
    * **通过指针/引用访问对象**
* 示例：`Container`类
    ```cpp
    class Container {
    public:
        virtual double& operator[](int) = 0; // 纯虚函数
        virtual int size() const = 0;         // 纯虚函数
        virtual ~Container() {}                // 虚析构函数
    };
    ```
    * `virtual`：表示该函数是**虚函数**，可以在派生类中重写
    * **`= 0`：表示该函数是纯虚函数**，派生类**必须**提供实现
    * `virtual ~Container() {}`：虚析构函数，确保通过基类指针删除派生类对象时调用正确的析构函数
    * 实例化：
        ```cpp
        Container c; // 错误：不能单纯定义一个抽象类对象
        Container* p = new Vector(10); // Vector是Container的派生类
        ```
    * `Container`的用法：
        ```cpp
        // 可以在完全不清楚Container的实现细节的情况下使用
        // 使用前提：创建了可供操作的派生类对象
        void f(Container& c) {
            const int sz = c.size();
            for (int i = 0; i != sz; ++i)
                cout << c[i] << '\n';
        }
        ```
        * 灵活性高：可以传入任何派生类对象
        * 但只能用指针/引用来操作对象
* 派生类：
    ```cpp
    class Vector : public Container {
    public:
        double& operator[](int i) override { return elem[i]; }
        int size() const override { return sz; }

        ~Vector() {}
        // ...
    }
    ```
    * `: public`继承
        * `Vector`派生自`Container`
        * `Container`是`Vector`的基类/父类
        * 关系：继承
    * `override`：可选，表示该函数重写了基类的虚函数
    * `~Vector() {}`：析构函数
        * `Container`的析构函数是虚的，因此`Vector`的析构函数自动是虚的
            * 但建议手动说明
        * **`~Vector()`覆盖了`~Container()`**
        * **`~Vector()`会被所属类的析构函数`~Container()`隐式调用**
        * 例如，下面的`smiley`类析构函数会先调用自己的析构函数（确保释放自己的`eyes`, `mouth`等特有对象），然后调用`Circle`的析构函数，最后是`Shape`的析构函数 **【自底向上】**


# 类层次结构
* 通过**派生**（如`public`）创建的一组在框架中有序排列的**类**
* 类层次结构一般**从基类开始向下生长**
* 例如：`Shape`基类
* 要定义一种具体的形状：
    * 指明属于`Shape`类层次结构
    * 规定其特有的属性
    * 示例：
    ```cpp
    class Shape {       
    public:
        virtual Point center() const = 0; // 纯虚函数
        virtual void draw() const = 0;
        // ...
        virtual ~Shape() {}
    };
    ```
    ```cpp
    class Circle : public Shape {
    public:
        Circle(Point p, double r) : x(p), radius(r) {} // 构造函数
        Point center() const override { return x; } // 重写纯虚函数
        void draw() const override;
        // ...
    private:
        Point x;
        int r;
    };
    ```
    ```cpp
    class Smiley : public Circle {
    public:
        Smiley(Point p, double r) : Circle{p, r}, mouth{nullptr} {}
        ~Smiley() {
            delete mouth; 
            for (auto eye : eyes)
                delete eye;
        }
        Point center() const override { return x; }
        void draw() const override;
        // ...

        void add_eye(Shape *s){ // 派生类特有的成员函数
            eyes.push_back(s);
        }
        void set_mouth(Shape *s){
            mouth = s;
        }
    
    private:
        std::vector<Shape*> eyes; // Smiley特有的数据成员
        Shape* mouth;
    };


    // Smiley类draw函数的定义
    // 调用Smiley对象的draw函数时，会调用Circle的draw函数，然后调用Smiley特有的draw代码
    void Smiley::draw() const {
        Circle::draw(); // 调用基类的draw函数
        // 画眼睛
        for (auto eye : eyes)
            eye->draw(); // 虚函数多态调用（指针）
        mouth->draw();
    }
    ```
* **类层次结构的益处**
1. **接口继承**
    * 派生类的对象可以**作为基类对象使用**
    * 往往属于**抽象类**
2. **实现继承**
    * 基类负责提供可以**简化派生类实现**的函数或数据
* **类层次结构导航--使用`dynamic_cast`**
* 使用前提：**有多态基类指针/引用**
* 使用场景：用于在运行时判断实际的**派生类类型**，再进行相应处理
* 能用**虚函数分派**解决的，优先使用虚函数；**非多态基类不可用**
* 示例：
    ```cpp
    // 多态指针示例
    void process_shape(Shape* s) { 
        // 如果不符合，返回nullptr
        if (Circle* c = dynamic_cast<Circle*>(s)) {
            // ...
        } else if (Rectangle* rect = dynamic_cast<Rectangle*>(s)) {
            // ...
        } else {
            // ...
        }
    }

    // 多态引用示例
    void process_shape(Shape& s) { 
        // 使用引用：如果不符合，会抛出`std::bad_cast`异常
        try {
            Circle& c = dynamic_cast<Circle&>(s);
            // ...
        } catch (std::bad_cast& e) {
            // ...
        }
    }
    ``` 
* 避免资源泄露
    * 指：**获取了资源（如动态内存），但未能正确释放**
    * 会导致：系统无法使用这些资源，最终耗尽
    * --> **使用`unique_ptr/shared_ptr`等智能指针管理动态内存，代替裸指针：**
        ```cpp
        #include <memory>

        class Smiley : public Circle {
        public:
           // ...
        
        private:
            Vector<std::unique_ptr<Shape>> eyes; // 使用智能指针
            std::unique_ptr<Shape> mouth;
        };
        ```
        * --> **不需要为`Smiley`类定义析构函数**
        * 编译器会隐式生成析构函数，自动调用智能指针的析构函数