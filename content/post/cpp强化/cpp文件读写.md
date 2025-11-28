### 文件操作

C++ 的文件操作主要通过标准库中的 `<fstream>` 头文件实现，它提供了三类核心流对象：`ifstream`（读文件）、`ofstream`（写文件）、`fstream`（读写文件）。文件操作涉及**打开文件**、**读写数据**、**关闭文件**三大步骤，还包括文件状态检查、定位指针等高级操作。以下是详细语法和示例：

#### 一、文件操作的基础准备

首先需要包含头文件

```cpp
#include <fstream>   // 核心文件操作头文件
#include <iostream>  // 用于控制台输出（可选）
using namespace std; // 简化命名空间（可选）
```

#### 二、文件的打开方式

文件打开时需指定**打开模式**，通过构造函数或 `open()` 方法设置，常见模式如下：

| 模式符号      | 含义                                                         |                              |
| ------------- | ------------------------------------------------------------ | ---------------------------- |
| `ios::in`     | 以读模式打开文件（`ifstream` 默认模式，文件不存在则失败）    |                              |
| `ios::out`    | 以写模式打开文件（`ofstream` 默认模式，文件不存在则创建，存在则清空） |                              |
| `ios::app`    | 追加模式（写入数据时追加到文件末尾，不覆盖原有内容）         |                              |
| `ios::ate`    | 打开文件后将指针定位到文件末尾（可读写）                     |                              |
| `ios::trunc`  | 若文件存在则清空内容（与 `ios::out` 配合使用，默认生效）     |                              |
| `ios::binary` | 二进制模式（默认是文本模式，用于处理图片、视频等二进制文件） |                              |
| `ios::in      | ios::out`                                                    | 读写模式（文件不存在则失败） |

###### 打开文件的两种方式

1. **通过构造函数打开**

   ```cpp
   ofstream outFile("test.txt", ios::out); // 写模式打开test.txt
   ifstream inFile("data.txt", ios::in);   // 读模式打开data.txt
   ```

2. **通过 `open()` 方法打开**

   ```cpp
   fstream fs;
   fs.open("file.bin", ios::in | ios::out | ios::binary); // 二进制读写模式
   ```

#### 三、文件的读写操作

###### 1. 文本文件的读写

文本文件以 ASCII 字符存储数据，支持 `<<`（写）、`>>`（读）、`getline()`（读整行）等操作。

###### （1）写文本文件

```cpp
#include <fstream>
using namespace std;

int main() {
    // 打开文件（写模式，不存在则创建，存在则清空）
    ofstream outFile("text.txt", ios::out);
    if (!outFile.is_open()) { // 检查文件是否成功打开
        cout << "文件打开失败！" << endl;
        return 1;
    }

    // 写入数据（使用<<运算符）
    outFile << "Hello, File!" << endl;
    outFile << "数字：" << 123 << " 浮点数：" << 3.14 << endl;

    outFile.close(); // 关闭文件（重要，否则可能数据丢失）
    return 0;
}
```

###### （2）读文本文件

- **使用 `>>` 读取**：以空格 / 换行分隔数据，适合结构化数据。

  ```cpp
  #include <fstream>
  #include <iostream>
  using namespace std;
  
  int main() {
      ifstream inFile("text.txt", ios::in);
      if (!inFile) { // 等价于!inFile.is_open()
          cout << "文件打开失败！" << endl;
          return 1;
      }
  
      string str;
      int num;
      double f;
      inFile >> str >> num >> f; // 按格式读取
      cout << str << " " << num << " " << f << endl;
  
      inFile.close();
      return 0;
  }
  ```

- **使用 `getline()` 读取整行**：适合读取带空格的文本。

  ```cpp
  #include <fstream>
  #include <string>
  using namespace std;
  
  int main() {
      ifstream inFile("text.txt");
      string line;
      while (getline(inFile, line)) { // 逐行读取直到文件末尾
          cout << line << endl;
      }
      inFile.close();
      return 0;
  }
  ```

###### 2. 二进制文件的读写

二进制文件直接存储字节数据，需使用 `write()`（写）和 `read()`（读）方法，配合 `ios::binary` 模式。

> `ostream& wirte(const char* buffer,int len);`
>
> `istream& read(char * buffer,int len);` // 参数解释:字符指针buffer指向内存中一段存储空间。

###### （1）写二进制文件

```cpp
#include <fstream>
using namespace std;

struct Student {
    char name[20];
    int age;
};

int main() {
    Student s = {"Tom", 18};
    ofstream outFile("student.bin", ios::out | ios::binary);
    outFile.write((char*)&s, sizeof(s)); // 写入结构体数据
    outFile.close();
    return 0;
}
```

###### （2）读二进制文件

```cpp
#include <fstream>
#include <iostream>
using namespace std;

struct Student {
    char name[20];
    int age;
};

int main() {
    Student s;
    ifstream inFile("student.bin", ios::in | ios::binary);
    inFile.read((char*)&s, sizeof(s)); // 读取结构体数据
    cout << "姓名：" << s.name << " 年龄：" << s.age << endl;
    inFile.close();
    return 0;
}
```

#### 四、文件指针的定位

文件指针用于标记当前读写位置，分为**读指针**（`istream`）和**写指针**（`ostream`），`fstream` 同时支持两者。

###### 1. 定位读指针：`seekg()`

```cpp
// seekg(偏移量, 起始位置)
inFile.seekg(0, ios::beg);   // 定位到文件开头（beg=begin）
inFile.seekg(-10, ios::end); // 定位到文件末尾前10字节（end=end）
inFile.seekg(5, ios::cur);   // 从当前位置向后移动5字节（cur=current）
```

###### 2. 定位写指针：`seekp()`

```cpp
outFile.seekp(0, ios::end); // 定位到文件末尾（用于追加）
```

###### 3. 获取当前指针位置

- `tellg()`：获取读指针位置（返回字节数）。
- `tellp()`：获取写指针位置。

```cpp
#include <fstream>
#include <iostream>
using namespace std;

int main() {
    fstream fs("text.txt", ios::in | ios::out);
    fs.seekg(0, ios::end); // 定位到末尾
    cout << "文件大小：" << fs.tellg() << " 字节" << endl;
    fs.seekg(0, ios::beg); // 回到开头
    return 0;
}
```

#### 五、文件状态检查

通过流对象的状态方法检查操作是否成功：

| 方法        | 含义                            |
| ----------- | ------------------------------- |
| `is_open()` | 文件是否成功打开                |
| `eof()`     | 是否到达文件末尾（end of file） |
| `fail()`    | 操作是否失败（如格式错误）      |
| `bad()`     | 是否发生严重错误（如文件损坏）  |
| `good()`    | 操作是否正常（无错误）          |

示例：

```cpp
if (!inFile.eof()) { // 未到文件末尾
    // 继续读取
}

if (inFile.fail()) { // 读取失败
    cout << "读取错误！" << endl;
}
```

#### 六、文件的关闭

使用 `close()` 方法关闭文件，释放资源：

```cpp
file.close();
```

**注意**：若未显式关闭，流对象析构时会自动关闭，但显式关闭更安全（避免缓冲区数据未写入）。

#### 七、总结

- 文本文件用 `<<`/`>>`/`getline()`，二进制文件用 `write()`/`read()`。
- 打开模式需根据需求组合（如 `ios::in | ios::binary`）。
- 文件指针定位用于随机读写，状态检查避免操作失败。

掌握这些语法即可实现 C++ 中常见的文件读写需求。



----

> 补充：黑马关于文本文件读取的操作
>
> ```cpp
> //4.读数据
> 	//第一种
> 	/*char buf[1024] = { 0 };
> 	while (ifs>>buf)
> 	{
> 		cout << buf << endl;
> 	}*/
> 
> 	//第二种
> 	/*char buf[1024] = { 0 };
> 	while (ifs.getline(buf,sizeof(buf)))
> 	{
> 		cout << buf << endl;
> 	}*/
> 
> 	//第三种
> 	/*string buf;
> 	while (getline(ifs,buf))
> 	{
> 		cout << buf << endl;
> 	}*/
> 
> 	//第四种-不推荐
> 	char c;
> 	while ((c = ifs.get()) != EOF)//EOF——end of file
> 	{
> 		cout << c;
> 	}
> ```
>
> 