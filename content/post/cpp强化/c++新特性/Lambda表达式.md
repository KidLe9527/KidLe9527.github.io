### C++ Lambda 表达式详解

Lambda 表达式是 C++11 引入的重要特性，允许在代码中定义匿名函数，极大简化了函数对象的使用，尤其适用于 STL 算法、回调函数等场景。

------

### 定义语法

Lambda 表达式的完整语法格式如下：

```cpp
[capture](parameters) mutable noexcept -> return_type {
    // 函数体
}
```

#### 各部分说明

1. **捕获子句（Capture Clause）**

   - `[]`：空捕获，不捕获外部变量
   - `[=]`：值捕获所有外部变量（拷贝）
   - `[&]`：引用捕获所有外部变量（引用）
   - `[var]`：值捕获特定变量`var`
   - `[&var]`：引用捕获特定变量`var`
   - `[this]`：捕获当前类的`this`指针（类成员函数中使用）

   

   - `[=, &a, &b]`表示以引用传递的方式捕捉变量`a`和`b`，以值传递方式捕捉其它所有变量。
   - `[&, a, this]`表示以值传递的方式捕捉变量`a`和`this`，引用传递方式捕捉其它所有变量。
   - 捕捉列表**不允许变量重复传递**
     - `[=,a]`这里已经以值传递方式捕捉了所有变量，但是重复捕捉`a`了，会报错的；

2. **参数列表（Parameters）**

   - 与普通函数参数列表类似，支持类型推导（C++14 起）
   - 无参数时可省略`()`

3. **可变说明符（Mutable）**🔥

   - 允许修改值捕获的变量（默认不可修改）

   ```cpp
   int a = 3;
   int b = 0;
   [&, b](int c) mutable { a = ++b + c;}(5); // 函数内部a = 1 + 5 = 6, b = 1调用lambda函数传参给c = 5
   cout << a << b << endl; // 函数外部a = 6, b = 0
   ```

   

4. **异常说明（Noexcept）**

   - 声明 Lambda 是否抛出异常
   - 在MSDN的异常规范中，明确指出异常规范是在 C++11 中弃用的 C++ 语言功能。因此这里不建议不建议大家使用。

5. **返回类型（Return Type）**

   - 可省略，编译器自动推导（单 return 语句时）

6. **函数体**

   - 执行逻辑代码



---

### 总结： lambda表达式就是匿名函数对象～

---



### Lambda表达式工作原理

编译器会把一个Lambda表达式生成一个匿名类的**匿名对象**，并在类中**重载函数调用运算符**，实现了一个`operator()`方法。

```cpp
auto print = [](){cout << "Hello World!" << endl; };
```

编译器会把上面这一句翻译为下面的代码：（补充上下文环境）

```cpp
#include <iostream>
using namespace std;

class print_class
{
public:
    // 显式定义默认构造函数，方便观察
    print_class() {
        cout << "调用了构造函数 print_class()" << endl;
    }

    // 重载函数调用运算符
    void operator()(void) const
    {
        cout << "调用了重载的 operator()，输出：Hello World!" << endl;
    }
};

int main() {
    // 这里的 () 是调用构造函数，创建对象 （类名 + ()）
    auto print = print_class();
    
    // 这里的 () 是调用重载的 operator()  （对象 + ()）
    print();
    
    return 0;
}
```

**核心区分点：`()` 跟在类名后 → 构造函数；`()` 跟在类对象后 → 重载的函数调用运算符。**