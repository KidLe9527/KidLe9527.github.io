---
title: '算法学习过程心得体会'

date: 2025-08-10T22:01:00-08:00
lastmod: 2025-08-10T22:02:00-08:00
categories: ['笔记']
tags: ['算法']
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



