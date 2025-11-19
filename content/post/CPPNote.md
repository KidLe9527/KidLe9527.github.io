---
title: 'CPP学习过程心得体会'

date: 2025-08-10T22:01:00-08:00
lastmod: 2025-08-10T22:02:00-08:00
categories: ['笔记']
tags: ['C++']
cover: https://kidle9527.github.io/images/33.png
---

# 学习C++新特性

## C++11

### std:move()

​	`std::move` 是 C++11 引入的一个标准库函数（定义在 `<utility>` 头文件中），它的核心作用是**将一个对象转换为右值引用（rvalue reference）**，从而触发移动语义（Move Semantics），实现资源的高效转移。

### 关键作用：

1. **转换值类型，不直接移动数据**
   `std::move` 本身并不 “移动” 数据，它只是给编译器一个信号：**这个对象可以被 “窃取” 资源**（即允许后续操作转移其内部资源）。

```cpp
std::vector<int> a = {1, 2, 3};
std::vector<int> b = std::move(a); // a 被转换为右值引用
```

这里 `b` 会直接接管 `a` 原来的内存（无需复制数据），而 `a` 会变成一个 “空壳”（有效但未定义的状态，通常不应再使用）。

2. **触发移动构造 / 赋值，优化性能**
   对于包含动态资源（如堆内存、文件句柄）的对象（如 `vector`、`string` 等），默认的拷贝操作会复制整个资源（耗时且占内存），而移动操作则直接 “接管” 资源，几乎不消耗额外成本。
   `std::move` 的作用就是让对象符合移动构造函数或移动赋值运算符的参数类型（右值引用），从而优先调用这些高效的移动接口。
3. **适用场景**
   - 当对象不再被使用时，通过 `std::move` 转移其资源（如临时变量、即将销毁的对象）。
   - 在函数参数传递或返回时，避免不必要的拷贝（例如返回大型容器时）。
   - 实现高效的容器元素操作（如 `std::swap` 的优化实现）。



### sto类函数（字符串转数字）
  * std::stoi (字符串转整数)--------------->"string to integer"​​
  * std::stol (字符串转长整数)
  * std::stoll (字符串转长长整数)
  * std::stof (字符串转浮点数)
  * std::stod (字符串转双精度浮点数)
```
#include <string>

std::string str = "123";
int num = std::stoi(str);  // num = 123
```

* 对于简单转换，使用 std::stoi/std::stod 系列函数

```
int stoi(const std::string& str, size_t* pos = 0, int base = 10);
// 其他函数参数类似

参数说明：

str：要转换的字符串
pos：可选参数，存储第一个未转换字符的位置
base：可选参数，指定数字的基数（2-36）
```

* 其他用法

```\
指定进制转换
std::string hex_str = "FF";
int hex_num = std::stoi(hex_str, nullptr, 16);  // 255 (十六进制)

std::string bin_str = "1010";
int bin_num = std::stoi(bin_str, nullptr, 2);   // 10 (二进制)

获取未转换部分位置
std::string str = "123abc";
size_t pos;
int num = std::stoi(str, &pos);  // num=123, pos=3
```



### istringstream(流输入)

* `std::istringstream`是 C++ 标准库中的一个类，属于 **字符串流（String Streams）** 的一部分，定义在 `<sstream>`头文件中。它允许将字符串（`std::string`）当作输入流（类似 `std::cin`）来处理，方便进行格式化的数据提取和解析。

```
std::string text = "C++ is awesome";
std::istringstream iss(text);
std::string word;

while (iss >> word) {  // 按空格分割
    std::cout << word << std::endl;
}
```



## C++17

### reduce函数（序列元素归约）

1. reduce函数是 C++17 引入的一个算法，用于对序列中的元素进行归约（reduction）操作，类似于 accumulate 但具有并行计算的潜力

```
语法
#include <numeric>

auto result = reduce(st.begin(), st.end());
```

2. 默认行为

* 当不提供初始值和操作时：
  1. 默认使用 std::plus<>() 作为二元操作（即加法）
  2. 默认初始值为 typename iterator_traits<It>::value_type{}（即该类型的默认构造值）

```
*int/float 等：初始值为 0，执行加法
vector<int> v{1, 2, 3};
int sum = reduce(v.begin(), v.end());  // 0+1+2+3 = 6
	
*需要类型支持 operator+ 和默认构造
vector<string> strs{"a", "b", "c"};
string concat = reduce(strs.begin(), strs.end());  // ""+"a"+"b"+"c" = "abc"
```



### count_if() 函数

* `std::count_if`是 C++ 标准库中的一个算法函数，用于**统计满足特定条件的元素个数**。它定义在 `<algorithm>`头文件中，是 STL 算法的重要组成部分。

```c++
函数原型：
template< class InputIt, class UnaryPredicate >
typename iterator_traits<InputIt>::difference_type
    count_if( InputIt first, InputIt last, UnaryPredicate p );
    
参数说明：
	1. first, last: 输入范围的迭代器（前闭后开区间）
	2. p: 一元谓词（返回 bool的可调用对象），用于测试元素是否满足条件
返回值：
	返回满足谓词条件的元素数量（类型为 difference_type，通常是 ptrdiff_t）
```

* 用法举例：

```
 // 统计偶数个数
    int even_count = std::count_if(nums.begin(), nums.end(), 
        [](int n) { return n % 2 == 0; });
```



## C++98

### accumulate

* `std::accumulate`是 C++ **标准库**中的算法函数，属于 `<numeric>`头文件，用于对序列中的元素进行累积计算（如求和、求积等）。

```
#include <numeric>

// 形式1：使用默认加法操作
T accumulate(InputIt first, InputIt last, T init);

// 形式2：使用自定义二元操作
T accumulate(InputIt first, InputIt last, T init, BinaryOp op);

参数：
1. first, last：输入范围的迭代器。
2. init：初始累积值（类型 T必须兼容操作结果）。
3. op：二元操作函数（可选，默认为 std::plus<T>()）。

std::vector<int> nums{1, 2, 3, 4, 5};
	
// 求和，初始值为0
int sum = std::accumulate(nums.begin(), nums.end(), 0);
// 结果：0 + 1 + 2 + 3 + 4 + 5 = 15
	
	
// 求乘积，初始值为1
int product = std::accumulate(nums.begin(), nums.end(), 1, 
    [](int a, int b) { return a * b; });
// 结果：1 * 1 * 2 * 3 * 4 * 5 = 120
	
	
字符串连接
std::vector<std::string> words{"Hello", " ", "World", "!"};

// 字符串连接，初始值为空字符串
std::string sentence = std::accumulate(words.begin(), words.end(), std::string());
// 结果："" + "Hello" + " " + "World" + "!" = "Hello World!"
	
	
自定义操作
// 计算向量中元素的平方和
int sum_of_squares = std::accumulate(nums.begin(), nums.end(), 0,
    [](int acc, int x) { return acc + x * x; });
// 结果：0 + 1² + 2² + 3² + 4² + 5² = 55
```



## C++20

### erase() 和 erase_if()

* std::erase() 和 std::erase_if() 是 `C++20`引入的两个新函数，用于简化从容器中删除元素的操作。它们提供了一种更安全、更简洁的方式来删除满足特定条件的元素，而无需手动处理迭代器或使用 erase-remove 惯用法

```
// 移除所有值为2的元素
    auto count = std::erase(nums, 2);  

// 移除单个元素(通过迭代器)
vec.erase(vec.begin() + 2); // 移除第三个元素(3)

// 移除一个范围内的元素
vec.erase(vec.begin(), vec.begin() + 2); // 移除前两个元素

 // 移除所有偶数
    auto count = std::erase_if(nums, [](int n) { return n % 2 == 0; });
    
// 通过键值移除元素
set.erase(3); // 移除值为3的元素

// 通过迭代器移除
set.erase(s.begin()); // 移除第一个元素

特点：
1.直接操作容器，不需要手动指定迭代器范围
2.返回被移除的元素数量
3.保持容器中剩余元素的相对顺序(序列容器)
4.谓词可以是任何可调用对象(函数指针、lambda、函数对象等)
```

---



### `std::format()`函数 和  ` { : b }`

1. `std::format`是 C++20 新增的 **字符串格式化库**，类似于 Python 的 `str.format()`或 C 的 `printf`，但更安全、更现代化。

基本语法：

```
#include <format>
std::string formatted_str = std::format("格式化字符串", 参数1, 参数2, ...);
```

2. `{:b}`是一个 **格式规范**，表示将整数格式化为 **二进制字符串**。

* 例如：`std::format("{:b}", 8)`→ `"1000"`。

* 其他常见格式：
  * `{:d}`：十进制（默认）。
  * `{:x}`：十六进制。
  * `{:o}`：八进制。

3. Eg ：

```
int year = 2025, month = 8, day = 12;
std::string binary_date = std::format("{:b}-{:b}-{:b}", year, month, day);
std::cout << binary_date << std::endl;  // 输出 "11111100001-1000-1100"
```

---

## C++23

### bit_width()

* `bit_width((unsigned) mx)`的作用：
  - `bit_width`是 C23 标准中引入的一个新函数，用于计算表示一个无符号整数所需的最小位数
  - 它的原型是`int bit_width(unsigned int x)`（对于不同的无符号类型有对应的版本）
  - 例如：`bit_width(0)`返回 0，`bit_width(1)`返回 1，`bit_width(4)`返回 3（因为 4 是 100，需要 3 位）



---

## 函数学习

### 字母和数字篇

1. isalnum(c)， 判断字符是字母或者数字
2. isalpha(c)，判断字符是字母
3. islower(c) 和 isupper(), 判断字符是大小写字母
4. isspace(c)，检查是否为空白字符
5. toupper() 和 tolower()，字母大小写转换，其他字符返回本身
6. isdigit(c)，用于**检查字符 `c`是否为十进制数字（0-9）**，检查 `'0'-'9'`。
7. isxdight(c)，检查 `'0'-'9'`、`'A'-'F'`、`'a'-'f'`（十六进制数字）。





---

* popcount() 函数

* `__builtin__popcount()`函数

---

### 函数模板的基本概念

* 函数模板是C++中的一种特性，它允许您编写通用的函数代码，这些函数可以处理多种数据类型而不需要为每种类型重写函数。

~~~cpp
// 没有模板时，需要为不同类型写多个重载函数
int max(int a, int b) { return a > b ? a : b; }
float max(float a, float b) { return a > b ? a : b; }
double max(double a, double b) { return a > b ? a : b; }

// 使用模板，只需写一次
template <typename T>
T max(T a, T b) { return a > b ? a : b; }
~~~

* 函数模板的基本语法结构

~~~cpp
template <typename T>  // 或者 template <class T>
返回类型 函数名(参数列表) {
    // 函数体
}
~~~

* 模板参数

~~~cpp
// 单个模板参数
template <typename T>
void print(T value) {
    cout << value << endl;
}

// 多个模板参数
template <typename T, typename U>
void printPair(T first, U second) {
    cout << first << ", " << second << endl;
}

// 非类型模板参数
template <typename T, int size>
class Array {
    T data[size];
};
~~~

* 函数模板的显示实例化

~~~cpp
template <typename T>	// 隐式实例化就是一般使用方法
T multiply(T a, T b) {
    return a * b;
}

int main() {
    cout << multiply<int>(5, 3.2);    // 显式指定T为int，3.2被转换为3
    cout << multiply<double>(2, 3.5); // 显式指定T为double，2被转换为2.0
    return 0;
}
~~~

* C++20引入concepts来约束模板参数类型
  * Concepts 是 **C++20** 引入的一种机制，用于对模板参数施加约束，明确指定模板参数必须满足的要求。它解决了传统模板编程中类型约束不明确、错误信息晦涩难懂的问题。

~~~cpp
//定义 Concepts
template <typename T>
concept Addable = requires(T a, T b) {
    { a + b } -> std::same_as<T>;  // 要求a+b的结果类型必须与T相同
};

//使用  concepts约束模板
template <Addable T>
T add(T a, T b) { return a + b; }

// 或者使用  requires子句
template <typename T>
requires Addable<T>
T add(T a, T b) { return a + b; }
~~~

* 举例说明

~~~cpp
template <typename T>
concept Arithmetic = requires(T a, T b) {
    a + b;
    a - b;
    a * b;
    a / b;
};

template <Arithmetic T>
T calculate(T a, T b) {
    return a * b + a / b;
}
~~~

1. `template <typename T> concept Arithmetic`定义了一个名为 `Arithmetic`的概念
2. `requires(T a, T b)`表示这个概念的约束条件将通过两个类型为 T 的参数 a 和 b 来验证
3. 约束条件是：类型T必须支持加减乘除运算，这些表达式必须合法（能够编译通过），但不关心返回类型
4. **受约束的模板函数**：
   1. `template <Arithmetic T>`表示这个模板只接受满足 `Arithmetic`概念的类型 T
   2. 函数实现中 `a * b + a / b`正好使用了概念中要求的所有四种运算
5. **编译时类型检查**：
   1. 当尝试用不支持这些操作的类型调用时，编译会在模板实例化时直接报错
   2. 错误信息会明确指出类型不满足 `Arithmetic`概念

---

## cpp中与upper、lower相关的函数

在 C++ 标准库中，与 `upper` 和 `lower` 相关的函数主要集中在**算法库（`<algorithm>`）** 和**范围库（C++20 起的 `<ranges>`）** 中，用于在有序序列中进行二分查找，核心是 `lower_bound` 和 `upper_bound` 系列函数。此外，字符串处理中也有少量相关函数（如大小写转换）。

### 一、二分查找相关函数（最核心）

这些函数用于在**已排序的序列**中高效查找元素，时间复杂度为 `O(log n)`，要求序列按升序排列（默认用 `<` 比较）。

#### 1. `std::lower_bound`（传统算法）

- **功能**：查找第一个**大于等于（`>=`）** 目标值 `x` 的元素。

- **参数**：`first, last`（迭代器范围）、`value`（目标值）。

- **返回值**：指向找到的元素的迭代器；若所有元素都小于 `x`，返回 `last`。

  ```cpp
  #include <algorithm>
  #include <vector>
  std::vector<int> arr = {1, 3, 5, 7};
  auto it = std::lower_bound(arr.begin(), arr.end(), 5);  // 指向 5
  ```

#### 2. `std::upper_bound`（传统算法）

- **功能**：查找第一个**严格大于（`>`）** 目标值 `x` 的元素。

- **参数**：同 `lower_bound`。

- **返回值**：指向找到的元素的迭代器；若所有元素都小于等于 `x`，返回 `last`。

  ```cpp
  auto it = std::upper_bound(arr.begin(), arr.end(), 5);  // 指向 7
  ```

#### 3. `std::ranges::lower_bound`（C++20 范围版）

- **功能**：同 `std::lower_bound`，但直接接受范围（如容器）作为参数，无需显式传递 `begin()` 和 `end()`。

- **示例**：

  ```cpp
  #include <ranges>
  auto it = std::ranges::lower_bound(arr, 5);  // 指向 5
  ```

#### 4. `std::ranges::upper_bound`（C++20 范围版）

- **功能**：同 `std::upper_bound`，范围版简化调用。

- **示例**：

  ```cpp
  auto it = std::ranges::upper_bound(arr, 5);  // 指向 7
  ```

#### 5. `std::equal_range`（传统算法）

- **功能**：同时返回 `lower_bound` 和 `upper_bound` 的结果，即返回一个迭代器对 `[first, last)`，表示序列中**所有等于 `x` 的元素范围**（因为序列有序，相等元素会连续）。

- **示例**：

  ```cpp
  auto [left, right] = std::equal_range(arr.begin(), arr.end(), 5);
  // left 是 lower_bound 结果（>=5），right 是 upper_bound 结果（>5）
  ```

#### 6. `std::ranges::equal_range`（C++20 范围版）

- **功能**：同 `std::equal_range`，范围版简化调用。

- **示例**：

  ```cpp
  auto [left, right] = std::ranges::equal_range(arr, 5);
  ```

### 二、大小写转换相关函数（字符处理）

在 `<cctype>` 库中，用于单个字符的大小写转换，名称包含 `lower` 和 `upper`：

#### 1. `tolower(int c)`

- **功能**：将大写字母转换为小写字母（非字母字符不变）。
- **示例**：`tolower('A')` 返回 `'a'`，`tolower('3')` 返回 `'3'`。

#### 2. `toupper(int c)`

- **功能**：将小写字母转换为大写字母（非字母字符不变）。
- **示例**：`toupper('b')` 返回 `'B'`，`toupper('!')` 返回 `'!'`。

### 三、其他相关函数（字符串 / 容器）

- **`std::string` 扩展**：部分实现（如微软 STL）可能提供非标准的 `_MakeLower` 或 `_MakeUpper`，但不建议使用，推荐用 `tolower`/`toupper` 遍历字符串实现。

- **自定义比较**：上述二分查找函数均可通过传入自定义比较器（如 `greater<>`）处理降序序列，例如：

  ```cpp
  // 在降序序列中查找第一个小于等于 x 的元素（类似 lower_bound 的反向逻辑）
  auto it = std::lower_bound(arr.begin(), arr.end(), 5, std::greater<int>());
  ```

### 总结💦

| 类别     | 函数（传统版）        | 函数（范围版，C++20）      | 核心功能                                         |
| -------- | --------------------- | -------------------------- | ------------------------------------------------ |
| 二分查找 | `std::lower_bound`    | `std::ranges::lower_bound` | 找第一个 `>= x` 的元素                           |
| 二分查找 | `std::upper_bound`    | `std::ranges::upper_bound` | 找第一个 `> x` 的元素                            |
| 二分查找 | `std::equal_range`    | `std::ranges::equal_range` | 找所有等于 `x` 的元素范围（返回 [lower, upper)） |
| 字符转换 | `tolower` / `toupper` | 无（直接使用）             | 单个字符大小写转换                               |
