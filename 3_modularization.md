# Chapter 3 -- Modularization 模块化

# Overview
* **IMPORTANT: 将接口和实现分离开来**
    * 通过**声明**来表示接口，如：
        ```cpp
        double sqrt(double);  // 指定了函数所需的

        class Vector { // 指定了一个类型所需的
            // 省略部分
            double* elem;
            int sz
        }
        ```
        * 一个实体（如函数）可以有很多声明，但只有一个定义

# 分离编译
* 用户只能看到接口部分（声明），而看不到实现部分（定义）
* 法1：**头文件(header files)**存放**声明**
    * 以`.h`或`.hpp`结尾
    * 通过`#include`指令包含进源文件（即，你需要声明该函数的地方）
* 法2：**模块(modules)**存放声明和定义
    * 以`.cppm`结尾
    * 通过`import`指令包含进源文件
    * 通过`export`指令区分**接口部分**和**实现部分**
* --> 减少编译时间；强制要求程序中逻辑独立的部分分离开

## 头文件
* 示例：`vector.h`
    ```cpp
    class Vector {
    public:
        Vector(int s);
        double& operator[](int i);
        int size();
    private:
        double* elem;
        int sz;
    };
    ```
    * `#include "Vector.h"` 
       * 1. **访问/使用**接口的脚本，如`v[i]`
       * 2. 提供具体**实现**的脚本，如`Vector.cpp`
* **翻译单元**：一个可以单独编译的`.cpp`文件 + 它所包含的所有头文件
* 使用`#include`+头文件实现modularization的缺点：
    * 编译时间：如果某个头文件被include n次，该头文件的内容会被编译n次
        * `#include`实际上是**文本替换/复制粘贴**
        * 大型项目（包含嵌套头文件）编译速度慢
    * 依赖顺序：头文件引入顺序不同，可能导致代码含义改变（必须细致安排头文件引入顺序）
    * 不协调：如果在不同的源文件中，对同一个实体的定义出现了细微的差别，会导致崩溃
    * 传染性：
        * 代码膨胀：头文件中包含某头文件，会包含该头文件的全部内容，即使我原本的包含目的只是使用某个简单功能
        * 底层头文件的改动，可能会引起大量源文件的重新编译

## 模块
* C++20引入
* `export`示例：
    ```cpp
    export module vector; // 定义一个模块，名为Vector

    export class Vector { // 模块组件1：Vector类和所有成员函数【接口】
    public:
        Vector(int s);
        double& operator[](int i);
        int size();
    private:
        double* elem;
        int sz;
    };

    // 实现部分
    Vector::Vector(int s) : elem(new double[s]), sz(s) {}
    double& Vector::operator[](int i) { return elem[i]; }
    int Vector::size() { return sz; }

    // 模块组件2：非成员函数：==操作符【接口+实现】
    export bool operator==(const Vector& a, const Vector& b) {
        if (a.size() != b.size()) return false;
        for (int i = 0; i < a.size(); ++i)
            if (a[i] != b[i]) return false;
        return true;
    }
    ```
* `import`示例：
    ```cpp
    import Vector; // 导入vector模块：获取相关接口
    // 省略
    ```
* 旧风格的`#include`可以和`import`一起使用
* 模块优点/区别：
    * 模块只编译一次--编译速度快
    * 不依赖于引入顺序
    * 没有传染性：如果在模块内部`import`或`#include`了其他模块/头文件，这些内容不会传染到导入该模块的源文件中（不会隐式获得访问权）
    * 模块【只导出接口】，但头文件需要传递所有内容（接口+实现）
        * 可以放心导入大规模模块，如`import std;` (但是c++20还不支持)
    * 不需要强制把声明和实现分开写成两个文件

# 命名空间
* 用于：
    * 表达某些声明是属于一个整体的
    * 表明他们的名字不会与其他命名空间中的名字冲突
* 示例：（c++标准库支持complex,但我们想自己定义）
    ```cpp
    // 命名空间My_code
    // My_code::complex不会和std::complex冲突
    namespace My_code {
        class complex {
            // ...
        };
        
        complex sqrt(complex); // 在命名空间中的complex指的就是My_code::complex
        // 如果要用标准库的，需要加上std::

        // ...
        int main();
        // 以上是函数声明
    }

    // My_code::main函数定义
    int My_code::main() {
        // ...
    }

    int main() { // 全局main函数：定义在全局命名空间
        return My_code::main();
    }
    ```
* `using`声明
    * 将命名空间中的某个名字引入当前作用域
    * `using namespace std;`
    * `using`指令不会影响模块用户：属于实现细节，只会在模块内部生效
    * **不要在头文件使用`using`指令** （传染性）

# 函数参数与返回值
* 强烈不建议使用全局变量来传递信息（函数之间）
* **函数之间的信息传递方法需要考虑的问题：**
    * （参数传递/返回值的默认行为是**复制**）
    * 对象：复制 or 共享？
        * 传递大对象，没有必要复制 --> 使用引用`&`或指针共享
        * **一般小对象传值，大对象传引用/指针**
    * 对象是否可被修改？
        * 无需修改 --> 使用`const`修饰
        * 进一步地（更常用）：使用`const`引用
    * 对象是否被移动，从而留下一个空对象？
* example for '参数传递/返回值的默认行为是**复制**'
    ```cpp
    void f(vector<int> v, vector<int>7 rv){
        v[1]=99 // 实际是复制，修改v
        rv[2]=66
    }

    int main(){
        vector<int> v = {1,2,3,4,5};
        f(v, v);
        cout << v[1] << " " << v[2]; // 输出 2 66
    }
    ```
* **默认**函数参数：
    ```cpp
    void f(int x, int y = 0);
    ``

## 返回值
* 返回引用的情况只应当出现在：**返回的内容不属于函数局部**（可以独立于当前函数的生命周期）
    * 如`v[1]`
    * **不能返回局部变量的引用**：局部变量在函数返回时消失，尝试用引用将此类变量传递出函数之外会导致未定义行为
* 对于大对象的返回：
    * 现代C++中通常**直接返回大对象**，而不是返回引用：
        ```cpp
        Matrix operator+(const Matrix& a, const Matrix& b){
            Matrix res;
            // 计算res[i, j]=x[i, j] + y[i, j]
            return res; // 直接返回
        }
        ```
    * 不是直接复制，而是给大对象提供**移动构造方法**（能以非常小的cost将大对象传递到函数之外）
        * or: 编译器自动优化此类复制：**省略复制优化**
* **返回类型推导**：(自动推导返回类型) 
    ```cpp
    auto mul(int i, double d){return i*d;} 
    ```
    * **不要过度使用**
* **返回类型后置**：
    ```cpp
    auto mul(int i, double d) -> double {return i*d;}
    ```
* **结构化绑定**：（返回多个值）
    ```cpp
    struct Entry {
        string name;
        double value;
    };

    Entry read_entry(istream& is){
        string s;
        int i;
        is >> s >> i; // 从输入流读取
        return {s, i}; // 相当于Entry(s, i)
    }

    // 旧写法（.操作符）
    auto e = read_entry(cin);
    cout << e.name << " " << e.value;

    // 结构化绑定：拆包
    auto [n, v] = read_entry(is);
    cout << n << " " << v;
    ```
    * 遍历map:
        ```cpp
        for (const auto [key, value] : m) {
        cout << key << " " << value; 
        }
        ```