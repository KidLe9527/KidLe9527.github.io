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

```
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



## 函数学习

### 字母和数字篇

1. isalnum(c)， 判断字符是字母或者数字
2. isalpha(c)，判断字符是字母
3. islower(c) 和 isupper(), 判断字符是大小写字母
4. isspace(c)，检查是否为空白字符
5. toupper() 和 tolower()，字母大小写转换，其他字符返回本身
6. isdigit(c)，用于**检查字符 `c`是否为十进制数字（0-9）**，检查 `'0'-'9'`。
7. isxdight(c)，检查 `'0'-'9'`、`'A'-'F'`、`'a'-'f'`（十六进制数字）。
