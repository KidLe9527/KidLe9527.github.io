---
title: '算法学习过程心得体会'

date: 2025-08-10T22:01:00-08:00
lastmod: 2025-08-10T22:02:00-08:00
categories: ['笔记']
tags: ['算法']
cover: https://kidle9527.github.io/images/22.png
---

## KMP算法

1. 下标从零开始

```
//这个 KMP 函数的返回值是模式串 t 在主串 s 中第一次出现的起始位置（从 0 开始计数）
int kmp(const string& s, const string& t){
        int s_length = s.size(), t_length = t.size();
        if (t_length == 0) return 0;
        if (s_length < t_length) return -1;

        //求next数组(从下标0开始)
        vector<int> next(t_length + 1);//大小 + 1防止堆和缓冲区溢出
        //next[t_length]的值是t字符串的最大前后缀公共长度
        next[0] = -1;
        int i = 0, j = -1;//i是当前正在计算next值的位置；j是前缀指针，当前已经匹配的长度
        while(i < t_length){
            if(j == -1 || t[i] == t[j]){//j = -1表示没有匹配的前缀
                next[++i] = ++j;
            }else{
                j = next[j];
            }
        }

        //KMP匹配，求t字符串在s字符串中出现的第一次位置
        i = 0, j = 0;//i是主串指针，j是匹配串指针
        while(i < s_length && j < t_length){
            if(j == -1 || s[i] == t[j]){
                ++i, ++j;
            }else{
                j = next[j];
            }
        }
        
        return j == t_length ? i - j : -1;
    }
};
```

* 这个 KMP 函数的返回值是模式串 t 在主串 s 中第一次出现的起始位置（从 0 开始计数）。具体来说

  

  * 如果 t 是空字符串（t_length == 0），返回 0，因为空字符串被认为是任何字符串的子串，且出现在位置 0。 
  * 如果 s 的长度小于 t 的长度（s_length < t_length），返回 -1，表示 t 不可能是 s 的子串。 
  * 如果 t 在 s 中找到匹配，返回匹配的起始位置（即 i - j，其中 i 是 s 中匹配结束的位置，j 是 t 的长度）。 
  * 如果 t 不在 s 中，返回 -1。 

* 例如：
  *  kmp("hello", "ll") 返回 2 
  * kmp("aaaaa", "bba") 返回 -1 
  * kmp("any", "") 返回 0 
* 注意：这个实现是从 0 开始计数的，与 C++ 的 std::string::find() 的返回值语义相同。

2. 下标从1开始的求next数组的kmp算法

```
int GetNext(char ch[], int length, int next[]){
    next[1] = 0;
    int i = 1, j = 0;
    while(i <= length){
        if(j==0 || ch[i] == ch[j]){
            next[++i] = ++j;
        }else{
            j = next[j];
        }
    }
}
```

[459. 重复的子字符串 - 力扣（LeetCode）](https://leetcode.cn/problems/repeated-substring-pattern/description/)



## 回溯问题

[17. 电话号码的字母组合 - 力扣（LeetCode）](https://leetcode.cn/problems/letter-combinations-of-a-phone-number/)

```
代码如下：

class Solution {
private:
    vector<string> dic = {"", "", "abc", "def", "ghi", "jkl", "mno", "pqrs", "tuv", "wxyz"};
public:
    vector<string> letterCombinations(string digits) {
        int n = digits.size();
        if(n == 0)  return {};

        vector<string> ans;
        string path;
        auto dfs = [&](this auto&& dfs, int i)->void{
            if(i == n){
                ans.emplace_back(path);
                return;
            }
            int idx = digits[i] - '0';
			for(int j = 0; j < dic[idx].size(); ++j){
				path += dic[idx][j];
				dfs(i + 1);
                path.pop_back();// 回溯，移除最后一个字母！ 必须要回溯！
			}
        };

        dfs(0);//回溯的入口！
        return ans;
    }
};
```

* 这里`path.pop_back()`用于递归后回溯的恢复数据，如果没有这一步path的路径会随着递归深入一直变长~



## 奇巧淫技

### 1. 求数组中的最大值

* `int max_val = std::ranges::max(nums)`

### 2. 字符串中查找某个字符的位置并返回序号

```cpp
size_t pos = chars.find(x);
            if(pos != std::string::npos){
                val = vals[pos];
            }else{
                val = x - 'a' + 1;  // 'a' 是 1
            }
```

* 函数原型：`size_t find(char c, size_t pos = 0) const;`
  - 第一个参数是要查找的字符
  - 第二个参数是起始查找位置（默认从 0 开始）
  - 返回值是找到的字符位置索引，如果未找到则返回`std::string::npos`

### 3. 用`max`函数求三个数的最大值

* `ans = max({ans, f_max, -f_min})`
  * 这行代码的作用是从三个值（`ans`、`f_max`、`-f_min`）中取最大值，并赋值给 `ans`
  * 这种语法依赖于 C++11 及以上标准引入的**初始化列表（initializer_list）** 特性：
    - `{ans, f_max, -f_min}` 会被隐式转换为 `initializer_list<int>` 类型
    - 标准库提供了针对 `initializer_list` 的 `max` 函数重载，可以接受任意数量的同类型参数并返回最大值

### 4. 二维数组的初始化

* 4.1 静态二维数组（编译时确定大小）

​	静态二维数组的大小在编译时固定，内存分配在栈上，适用于尺寸已知且固定的场景。

1.完全初始化（显示执行所有元素）

* 内层大括号可以省略，元素按照行优先顺序填充

```cpp
// 格式：类型 数组名[行数][列数] = {{行1元素}, {行2元素}, ...};
int arr[2][3] = {
    {1, 2, 3},   // 第0行
    {4, 5, 6}    // 第1行
};
```

2. 部分初始化（未指定的元素自动为0）

```cpp
// 只初始化部分元素，剩余元素默认值为0
int arr[3][3] = {
    {1},         // 第0行：[1, 0, 0]
    {4, 5},      // 第1行：[4, 5, 0]
    {}           // 第2行：[0, 0, 0]
};
```

3. 省略行数（由编译器自动推断行，列不能不可以不准省略～）

```cpp
// 列数必须指定，行数由初始化元素数量决定
int arr[][3] = {
    {1, 2, 3},
    {4, 5, 6}    // 编译器自动推断行数为2
};
```

* 4.2 动态二维数组（vector<vector<int>>, STL容器)

​	`vector<vector<int>>` 是动态二维数组，大小可在运行时调整，内存分配在堆上，是刷题和实际开发中最常用的方式（如 LeetCode 中的输入通常为此类型）。

​	不搞那么多操作，最重要的就是一点，已知行和列，初始化二维数组：

```cpp
int m = grid.size(), n = grid[0].size();
vector memo(m, vector<int>(n, -1)); // -1 表示没有计算过
```

### 5. 三维数组的初始化

1. 完全显示：` vector<vector<vector<int>>> memo(m, vector<vector<int>>(n, vector<int>(u, -1))); `
2. 简式：`vector memo(m, vector(n, vector<int>(u, -1)));`

### 6. Std:: move() 函数

​	`f = std::move(g);` 是 C++ 中利用移动语义（Move Semantics）进行的操作，主要作用是**高效地将对象 `g` 的资源转移给 `f`，而不是进行耗时的拷贝**。


## 计时模块