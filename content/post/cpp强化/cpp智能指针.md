```
C++ 中的智能指针是 RAII（资源获取即初始化） 机制的实现，用于自动管理动态分配的内存（或其他资源），避免内存泄漏和悬空指针问题。智能指针本质上是封装了原始指针的类模板，通过析构函数自动释放资源，核心原理是 利用栈对象的生命周期自动管理堆资源。
```

### 一、智能指针的分类与语法

C++ 标准库提供了三种主要的智能指针（`<memory>`头文件）：`std::unique_ptr`、`std::shared_ptr`、`std::weak_ptr`，还有已被废弃的`std::auto_ptr`。

#### 1. `std::unique_ptr`

- **特性**：**独占所有权**，同一时间只能有一个`unique_ptr`指向某一资源，不支持拷贝（但支持移动）。

- **语法**：

  ```cpp
  #include <memory>
  #include <iostream>
  
  int main() {
      // 创建unique_ptr，管理动态分配的int
      std::unique_ptr<int> ptr1(new int(10));
      // 或使用make_unique（C++14起推荐，更安全）
      auto ptr2 = std::make_unique<int>(20);
  
      // 访问资源：解引用或->
      std::cout << *ptr1 << std::endl; // 输出10
  
      // 移动语义（转移所有权）
      std::unique_ptr<int> ptr3 = std::move(ptr1); // ptr1变为空，ptr3拥有资源
  
      // 重置（释放当前资源，可指向新资源）
      ptr3.reset(new int(30)); // 原资源（值20）被释放，ptr3指向30
  
      // 释放资源（主动置空）
      ptr3.reset(); // 资源被释放，ptr3为空
  
      return 0; // 函数结束时，所有unique_ptr析构，自动释放资源
  }
  ```

- **原理**：

  - 析构函数中调用`delete`释放管理的资源。
  - 禁用拷贝构造函数和拷贝赋值运算符（C++11 通过`= delete`），仅提供移动构造和移动赋值，确保所有权唯一。

#### 2. `std::shared_ptr`

- **特性**：**共享所有权**，多个`shared_ptr`可指向同一资源，通过**引用计数**（reference count）管理资源生命周期：当最后一个`shared_ptr`销毁时，资源才被释放。

- **语法**：

  ```cpp
  #include <memory>
  #include <iostream>
  
  int main() {
      // 创建shared_ptr（推荐用make_shared，更高效）
      std::shared_ptr<int> ptr1 = std::make_shared<int>(10);
      std::shared_ptr<int> ptr2 = ptr1; // 拷贝，引用计数+1（此时计数为2）
  
      // 查看引用计数
      std::cout << "引用计数：" << ptr1.use_count() << std::endl; // 输出2
  
      // 重置ptr2，引用计数-1（变为1）
      ptr2.reset();
  
      // ptr1销毁时，引用计数变为0，资源释放
      return 0;
  }
  ```

  

- **原理**：

  - 内部维护两个指针：**指向资源的指针** + **指向控制块的指针**（控制块包含引用计数、弱引用计数、析构器等）。
  - 拷贝`shared_ptr`时，控制块的引用计数加 1；析构或重置时，引用计数减 1，当计数为 0 时，释放资源。
  - `make_shared`一次性分配资源和控制块的内存，比直接`new`更高效（减少内存分配次数）。

#### 3. `std::weak_ptr`

- **特性**：**弱引用**，指向`shared_ptr`管理的资源，但不增加引用计数，用于解决`shared_ptr`的**循环引用**问题。

- **语法**：

  ```cpp
  #include <memory>
  #include <iostream>
  
  struct Node {
      std::shared_ptr<Node> next;
      std::weak_ptr<Node> prev; // 用weak_ptr避免循环引用
  };
  
  int main() {
      std::shared_ptr<Node> node1 = std::make_shared<Node>();
      std::shared_ptr<Node> node2 = std::make_shared<Node>();
  
      node1->next = node2;
      node2->prev = node1; // weak_ptr不增加node1的引用计数
  
      // 检查weak_ptr是否指向有效资源（lock()返回shared_ptr，若过期则为空）
      if (auto prev = node2->prev.lock()) {
          std::cout << "prev有效" << std::endl;
      }
  
      return 0;
  }
  ```

  

- **原理**：

  - 指向`shared_ptr`的控制块，但不修改引用计数，因此不影响资源的释放。
  - 通过`lock()`方法可获取一个`shared_ptr`（若资源已释放则返回空），用于安全访问资源。

### 二、核心原理总结

1. **RAII 机制**：智能指针是栈上的对象，其析构函数会在生命周期结束时自动调用，从而释放管理的堆资源。
2. **所有权管理**：
   - `unique_ptr`：独占所有权，通过移动语义转移所有权。
   - `shared_ptr`：共享所有权，通过引用计数跟踪资源的所有者数量。
   - `weak_ptr`：弱引用，不参与所有权管理，解决循环引用。
3. **安全性**：避免手动调用`delete`，减少内存泄漏、双重释放等问题；`make_shared`/`make_unique`避免裸指针的直接使用，更安全。

### 三、注意事项

- 不要用同一个裸指针初始化多个`shared_ptr`（会导致重复释放）。
- 避免`shared_ptr`的循环引用（A 含`shared_ptr<B>`，B 含`shared_ptr<A>`），此时用`weak_ptr`打破循环。
- `unique_ptr`支持自定义删除器（用于管理非`new`分配的资源，如`malloc`或文件句柄），`shared_ptr`也支持。

例如，`unique_ptr`自定义删除器：

```cpp
#include <memory>
#include <cstdio>

// 自定义删除器：释放FILE*
struct FileDeleter {
    void operator()(FILE* fp) const {
        if (fp) std::fclose(fp);
    }
};

int main() {
    std::unique_ptr<FILE, FileDeleter> fp(std::fopen("test.txt", "w"));
    // 析构时自动调用FileDeleter，关闭文件
    return 0;
}
```

## ----------------------------------------------------

## 补充定义和实现💥

下面我们仿照你提供的简化版智能指针的定义风格，分别实现并介绍**独占型（类似`unique_ptr`）**、** 共享型（类似`shared_ptr`）**和**弱引用型（类似`weak_ptr`）**智能指针，帮助你直观理解它们的设计逻辑和使用方法。

### 一、独占型智能指针（MyUniquePtr）

模仿`std::unique_ptr`，核心是**独占资源所有权**，禁止拷贝、支持移动。

#### 定义实现

```cpp
template <class T>
class MyUniquePtr {
public:
    // 构造函数：接管裸指针
    MyUniquePtr(T* ptr = nullptr) : _ptr(ptr) {}

    // 禁用拷贝构造（独占所有权，不能拷贝）
  	//= delete是 C++11 引入的语法，称为“删除函数”（deleted function），表示显式禁用该函数 
  	//编译器会拒绝任何尝试调用该函数的代码。
    MyUniquePtr(const MyUniquePtr&) = delete;
    MyUniquePtr& operator=(const MyUniquePtr&) = delete;

    // 移动构造：转移所有权
    MyUniquePtr(MyUniquePtr&& other) noexcept {
        _ptr = other._ptr;
        other._ptr = nullptr; // 原对象放弃所有权
    }

    // 移动赋值：转移所有权
    MyUniquePtr& operator=(MyUniquePtr&& other) noexcept {
        if (this != &other) {
            delete _ptr; // 释放当前资源
            _ptr = other._ptr;
            other._ptr = nullptr;
        }
        return *this;
    }

    // 解引用
    T& operator*() const { return *_ptr; }
    T* operator->() const { return _ptr; }

    // 重置资源
    void reset(T* ptr = nullptr) {
        delete _ptr;
        _ptr = ptr;
    }

    // 获取裸指针
    T* get() const { return _ptr; }

    // 析构函数：释放资源
    ~MyUniquePtr() { delete _ptr; }

private:
    T* _ptr;
};
```

####  核心特点

- **独占所有权**：同一时间只有一个`MyUniquePtr`管理资源，禁用拷贝确保不出现多个所有者。
- **移动语义**：通过`std::move`转移所有权，原对象置空，避免资源被重复释放。

### 二、共享型智能指针（MySharedPtr）

模仿`std::shared_ptr`，核心是**引用计数**，多个指针共享资源，最后一个指针销毁时释放资源。

####  定义实现

```cpp
template <class T>
class MySharedPtr {
public:
    // 构造函数：初始化指针和引用计数
    MySharedPtr(T* ptr = nullptr) : _ptr(ptr), _refCount(new int(1)) {}

    // 拷贝构造：共享资源，引用计数+1
    MySharedPtr(const MySharedPtr& other) {
        _ptr = other._ptr;
        _refCount = other._refCount;
        (*_refCount)++; // 引用计数增加
    }

    // 拷贝赋值：释放当前资源，共享新资源
    MySharedPtr& operator=(const MySharedPtr& other) {
        if (this != &other) {
            // 释放当前资源（若引用计数为1则删除）
            release();
            // 共享新资源
            _ptr = other._ptr;
            _refCount = other._refCount;
            (*_refCount)++;
        }
        return *this;
    }

    // 解引用
    T& operator*() const { return *_ptr; }
    T* operator->() const { return _ptr; }

    // 获取引用计数
    int use_count() const { return *_refCount; }

    // 释放资源（内部调用）
    void release() {
        (*_refCount)--;
        if (*_refCount == 0) {
            delete _ptr;
            delete _refCount; // 释放引用计数本身
        }
      
       // 3. 关键：当前对象放弃所有权后，置空内部指针（避免野指针）
        _ptr = nullptr;
        _refCount = nullptr;
    }

    // 析构函数：释放资源
    ~MySharedPtr() { release(); }

private:
    T* _ptr;         // 指向资源的指针
    int* _refCount;  // 引用计数（动态分配，便于共享）
};
```

#### 核心特点

- **共享所有权**：多个`MySharedPtr`可指向同一资源，通过引用计数跟踪所有者数量。
- **自动释放**：当引用计数为 0 时（最后一个指针销毁），才释放资源，避免内存泄漏。

### 与`std::shared_ptr`的差异（简化版局限）

- 未实现**移动语义**（`std::shared_ptr`支持移动，减少计数操作）；
- 未处理**循环引用**（需配合`weak_ptr`解决）；
- 无**线程安全**（`std::shared_ptr`的计数操作是原子的，本实现多线程下可能计数错误）；
- 不支持**自定义删除器**（`std::shared_ptr`可自定义资源释放逻辑）

### 三、弱引用型智能指针（MyWeakPtr）

模仿`std::weak_ptr`，核心是**不增加引用计数**，解决`MySharedPtr`的循环引用问题，需配合共享指针使用。

#### 共享指针循环引用的问题🔥

```cpp
#include <iostream>
using namespace std;

// 前置声明（因为A和B互相引用）
class B;
class A {
public:
    MySharedPtr<B> b_ptr;  // A持有B的共享指针
    ~A() { cout << "A被销毁" << endl; }
};

class B {
public:
    MySharedPtr<A> a_ptr;  // B持有A的共享指针
    ~B() { cout << "B被销毁" << endl; }
};

int main() {
    {
        // 创建两个对象，建立双向关联
        MySharedPtr<A> a = MySharedPtr<A>(new A());
        MySharedPtr<B> b = MySharedPtr<B>(new B());

        cout << "关联前：a的计数=" << a.use_count()  // 1（只有a指向A）
             << ", b的计数=" << b.use_count() << endl;  // 1（只有b指向B）

        // 互相持有：形成循环引用
        a->b_ptr = b;  // A的b_ptr指向B，B的引用计数+1（变为2）
        b->a_ptr = a;  // B的a_ptr指向A，A的引用计数+1（变为2）

        cout << "关联后：a的计数=" << a.use_count()  // 2（a + b->a_ptr）
             << ", b的计数=" << b.use_count() << endl;  // 2（b + a->b_ptr）
    }
    // 代码块结束：a和b的局部变量被销毁

    cout << "代码块结束，检查是否释放资源" << endl;
    return 0;
}
```

```cpp
// 运行结果
关联前：a的计数=1, b的计数=1
关联后：a的计数=2, b的计数=2
代码块结束，检查是否释放资源
  
//关键问题：A和B的析构函数都没有被调用！资源（new A()和new B()）和引用计数都没有被释放，造成了内存泄漏。
步骤 1：创建对象并关联时的引用计数
初始化a（指向A）：A的引用计数 = 1（仅a持有）。
初始化b（指向B）：B的引用计数 = 1（仅b持有）。
a->b_ptr = b：B的引用计数 + 1 → 2（持有者：b + a->b_ptr）。
b->a_ptr = a：A的引用计数 + 1 → 2（持有者：a + b->a_ptr）。

步骤 2：局部变量a和b销毁时的变化
代码块结束后，局部变量a和b的析构函数被调用：
销毁a（局部变量）：A的引用计数 - 1 → 1（剩余持有者：b->a_ptr）。
	因为计数不为 0，A对象和A的引用计数不会被释放。
销毁b（局部变量）：B的引用计数 - 1 → 1（剩余持有者：a->b_ptr）。
	因为计数不为 0，B对象和B的引用计数不会被释放。
  
步骤 3：最终的 “死锁” 状态
此时内存中残留：
A对象的b_ptr仍持有B的共享指针（B的计数 = 1）。
B对象的a_ptr仍持有A的共享指针（A的计数 = 1）。
这两个指针互相 “牵制”，但没有任何外部指针能访问到它们，导致：
	引用计数永远无法减为 0。
A和B对象永远无法被销毁，造成内存泄漏。
```



#### 定义实现（需依赖 MySharedPtr）

```cpp
template <class T>
class MyWeakPtr {
public:
    // 构造函数：从MySharedPtr初始化
    MyWeakPtr(const MySharedPtr<T>& sharedPtr) {
        _ptr = sharedPtr.get();
        _refCount = sharedPtr.use_count_ptr(); // 假设MySharedPtr提供获取引用计数指针的接口
    }

    // 尝试获取共享指针（lock）：若资源有效则返回MySharedPtr，否则返回空
    MySharedPtr<T> lock() const {
        if (expired()) {
            return MySharedPtr<T>(nullptr);	// 资源过期，返回空的MySharedPtr
        }
        return MySharedPtr<T>(_ptr, _refCount); // 需为MySharedPtr增加构造函数支持，资源有效，返回共享指针
    }

    // 检查资源是否过期（引用计数为0）
    bool expired() const {
        return *_refCount == 0;	//true过期
    }

private:
    T* _ptr;         // 指向资源的指针（不管理生命周期）
    int* _refCount;  // 引用计数（仅观察，不修改）
};

// 注意：需为MySharedPtr补充获取引用计数指针的接口
template <class T>
int* MySharedPtr<T>::use_count_ptr() const {
    return _refCount;
}

// 为MySharedPtr补充构造函数（供MyWeakPtr::lock使用）
template <class T>
MySharedPtr<T>::MySharedPtr(T* ptr, int* refCount) : _ptr(ptr), _refCount(refCount) {
    (*_refCount)++;	//// 新共享指针加入，计数+1
}
```

#### 使用方法（解决循环引用）💧

```cpp
#include <iostream>
using namespace std;

// 先定义MySharedPtr（含补充接口），再定义MyWeakPtr

struct Node {
    MyWeakPtr<Node> next; // 用弱引用替代共享引用，打破循环
};

int main() {
    // 1. 创建共享指针
    MySharedPtr<Node> node1(new Node);
    MySharedPtr<Node> node2(new Node);
    cout << "node1计数：" << node1.use_count() << endl; // 1
    cout << "node2计数：" << node2.use_count() << endl; // 1

    // 2. 弱引用关联（不增加计数）
    node1->next = node2;
    node2->next = node1;
    cout << "node1计数：" << node1.use_count() << endl; // 仍为1
    cout << "node2计数：" << node2.use_count() << endl; // 仍为1

    // 3. 通过lock()安全访问弱引用资源
    MyWeakPtr<Node> wp = node1;
    if (!wp.expired()) { // 检查资源是否有效
        MySharedPtr<Node> sp = wp.lock(); // 获取共享指针，计数+1（node1计数→2）
        cout << "弱引用资源有效" << endl;
    } // sp析构，node1计数→1

    // 4. 析构时正常释放
    return 0; // node1、node2析构，计数→0，资源释放
}
```

#### 核心特点

- **弱引用**：不参与资源的所有权管理，不增加引用计数，不影响资源释放。
- **解决循环引用**：当两个`MySharedPtr`互相引用时，用`MyWeakPtr`打破循环，避免引用计数无法归 0 导致的内存泄漏。
- **安全访问**：通过`lock()`方法获取`MySharedPtr`，若资源已释放则返回空，避免悬空指针。

### 四、三种智能指针的对比总结

| 类型          | 核心特性               | 适用场景                 |
| ------------- | ---------------------- | ------------------------ |
| `MyUniquePtr` | 独占所有权，禁止拷贝   | 单一所有者管理资源       |
| `MySharedPtr` | 共享所有权，引用计数   | 多个所有者共享资源       |
| `MyWeakPtr`   | 弱引用，不影响引用计数 | 配合共享指针解决循环引用 |

这些简化实现省略了标准库智能指针的部分细节（如自定义删除器、空指针检查、线程安全等），但核心设计思想与`std::unique_ptr`、`std::shared_ptr`、`std::weak_ptr`完全一致。理解它们的实现逻辑，能更深入掌握 C++ 标准库智能指针的使用原理。



## --------------------------------------------------

# 详细解释 

## 🌞 独占指针

`std::unique_ptr`是 C++11 引入的**独占型智能指针**，核心特性是**独占资源所有权**—— 同一时间只能有一个`unique_ptr`指向某一资源，不支持拷贝（但支持移动），通过 RAII 机制自动释放资源。下面结合代码详细介绍其语法、用法及特性：

### 一、基本语法与创建方式

#### 1. 头文件依赖

使用`std::unique_ptr`必须包含头文件：

```cpp
#include <memory>
```

#### 2. 创建`unique_ptr`对象

- **方式 1：通过裸指针初始化**

  直接用`new`分配的裸指针初始化`unique_ptr`，接管资源所有权：

  ```cpp
  std::unique_ptr<int> ptr1(new int(10)); // 管理int类型资源，值为10
  std::unique_ptr<std::string> ptr2(new std::string("hello")); // 管理string类型资源
  ```

- **方式 2：使用`std::make_unique`（C++14 起推荐）**

  `make_unique`是标准库提供的工厂函数，直接创建资源并返回`unique_ptr`，更安全（避免裸指针暴露）、更高效：

  ```cpp
  auto ptr3 = std::make_unique<int>(20); // 等价于std::unique_ptr<int>(new int(20))
  auto ptr4 = std::make_unique<std::vector<int>>(3, 10); // 管理vector，含3个10
  ```

  ❗ 注意：`make_unique`不支持自定义删除器，若需自定义删除器，需用方式 1。

### 二、资源访问与操作

#### 1. 解引用与成员访问

`unique_ptr`重载了`operator*`（解引用）和`operator->`（成员访问），用法与裸指针一致：

```cpp
std::unique_ptr<int> ptr(new int(10));
std::cout << *ptr << std::endl; // 解引用：输出10

std::unique_ptr<std::string> str_ptr(new std::string("hello"));
std::cout << str_ptr->size() << std::endl; // 成员访问：输出5（string长度）
```

#### 2. 获取裸指针（`get()`）

通过`get()`方法可获取`unique_ptr`管理的裸指针（仅用于访问，不建议修改或释放）：

```cpp
std::unique_ptr<int> ptr(new int(10));
int* raw_ptr = ptr.get(); // raw_ptr指向10，但所有权仍归ptr
std::cout << *raw_ptr << std::endl; // 输出10
// 禁止delete raw_ptr; 否则ptr析构时会双重释放
```

#### 3. 释放资源（`reset()`）

- `reset()`：释放当前管理的资源，`unique_ptr`变为空；
- `reset(new T(...))`：释放当前资源，接管新资源的所有权。

```cpp
std::unique_ptr<int> ptr(new int(10));
ptr.reset(); // 释放资源，ptr变为空（指向nullptr）

ptr.reset(new int(20)); // ptr现在管理值为20的int资源
```

#### 4. 转移所有权（移动语义）

`unique_ptr`不支持拷贝（拷贝构造 / 赋值被禁用），但支持**移动语义**（通过`std::move`转移所有权）：

```cpp
std::unique_ptr<int> ptr1(new int(10));
std::unique_ptr<int> ptr2 = std::move(ptr1); // 所有权转移给ptr2，ptr1变为空

if (ptr1 == nullptr) {
    std::cout << "ptr1已空" << std::endl; // 输出
}
std::cout << *ptr2 << std::endl; // 输出10
```

#### 5. 释放并获取资源（`release()`）

`release()`：放弃资源所有权，返回裸指针（但不释放资源），`unique_ptr`变为空。常用于将资源转移给其他智能指针或手动管理：

```cpp
std::unique_ptr<int> ptr(new int(10));
int* raw_ptr = ptr.release(); // ptr放弃所有权，变为空
std::cout << *raw_ptr << std::endl; // 输出10
delete raw_ptr; // 需手动释放资源（否则内存泄漏）
```

### 三、特殊用法：管理数组

`unique_ptr`原生支持数组类型（C++11 起），会**自动用`delete[]`释放资源，无需手动处理**：

```cpp
// 管理int数组（长度为3）
std::unique_ptr<int[]> arr_ptr(new int[3]{1, 2, 3});
std::cout << arr_ptr[0] << std::endl; // 输出1
arr_ptr[1] = 10; // 修改数组元素

// 用make_unique创建数组（C++14）
auto arr_ptr2 = std::make_unique<int[]>(5); // 长度为5的int数组，默认初始化为0
```

❗ 注意：`std::shared_ptr`管理数组需手动指定删除器（`std::default_delete<T[]>`），而`unique_ptr`对数组更友好。

### 四、自定义删除器

`unique_ptr`支持自定义删除器，用于管理非`new`分配的资源（如`malloc`内存、文件句柄、网络连接等）。删除器是一个可调用对象（函数、lambda、函数对象），在`unique_ptr`析构时调用。

#### 示例 1：管理`malloc`分配的内存

```cpp
#include <memory>
#include <cstdlib> // malloc/free
#include <iostream>

int main() {
    // 自定义删除器：lambda表达式，释放malloc内存
    auto free_deleter = [](int* p) {
        std::free(p);
        std::cout << "malloc内存已释放" << std::endl;
    };

    // 创建unique_ptr，指定删除器类型为lambda的类型（decltype）
    std::unique_ptr<int, decltype(free_deleter)> ptr(
        static_cast<int*>(std::malloc(sizeof(int))), // malloc分配int内存
        free_deleter
    );

    *ptr = 100; // 使用资源
    std::cout << *ptr << std::endl; // 输出：100

    return 0; // 析构时调用free_deleter，释放内存
}
```

#### 示例 2：管理文件句柄

```cpp
#include <memory>
#include <cstdio> // fopen/fclose
#include <iostream>

// 自定义删除器：函数对象（结构体重载operator()）
struct FileDeleter {
    void operator()(FILE* fp) const {
        if (fp) {
            std::fclose(fp);
            std::cout << "文件已关闭" << std::endl;
        }
    }
};

int main() {
    // 创建unique_ptr，管理FILE*，删除器为FileDeleter
    std::unique_ptr<FILE, FileDeleter> fp(std::fopen("test.txt", "w"));

    if (fp) { // 检查文件是否成功打开
        std::fputs("Hello, unique_ptr!", fp.get()); // 写入文件
    }

    return 0; // 析构时调用FileDeleter，关闭文件
}
```

### 五、使用场景与注意事项

#### 1. 适用场景

- **独占资源所有权**：如函数返回动态分配的对象、管理局部动态资源、容器中存储独占对象（如`std::vector<std::unique_ptr<T>>`）。
- **替代`std::auto_ptr`**：`auto_ptr`是 C++98 的过时产物，存在拷贝语义缺陷，`unique_ptr`是更安全的替代品。
- **管理数组资源**：原生支持数组，无需额外处理`delete[]`。

#### 2. 注意事项

- ❌ 禁止拷贝`unique_ptr`：`std::unique_ptr<int> ptr2 = ptr1;`会编译错误（拷贝构造被`= delete`禁用）。
- ❌ 禁止用同一裸指针初始化多个`unique_ptr`：否则会导致双重释放。
- ✅ 优先使用`std::make_unique`：避免裸指针暴露，提升异常安全性。
- ✅ 移动后原`unique_ptr`为空：移动后的原对象不能再访问资源（否则会访问空指针）。

### 六、总结

`std::unique_ptr`是 C++ 中**轻量级、高效**的独占型智能指针，核心优势在于：

- 独占资源所有权，避免资源竞争；
- 自动释放资源，杜绝内存泄漏；
- 支持移动语义，灵活转移所有权；
- 原生支持数组和自定义删除器，适配多种资源类型。

掌握`unique_ptr`的用法，是 C++ 资源管理的基础，也是理解`shared_ptr`、`weak_ptr`的前提。