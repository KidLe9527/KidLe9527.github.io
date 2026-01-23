### 一、C++ 常见标准异常类型

C++ 标准库定义了一组继承自 `std::exception` 的异常类型（`<exception>` 头文件），覆盖绝大多数通用异常场景，也是业务代码中最常使用的基础异常类型。

| 异常类型                 | 所属头文件       | 核心用途                                        | 典型触发场景                                                 |
| ------------------------ | ---------------- | ----------------------------------------------- | ------------------------------------------------------------ |
| `std::exception`         | `<exception>`    | 所有标准异常的基类（可自定义子类）              | 作为自定义异常的父类，捕获所有标准异常（兜底）               |
| `std::runtime_error`     | `<stdexcept>`    | 运行时错误（程序逻辑 / 状态合法，但执行中出错） | 线程池已停止仍提交任务、文件读写失败、网络连接异常           |
| `std::logic_error`       | `<stdexcept>`    | 逻辑错误（程序设计 / 参数非法，可提前避免）     | 传入非法参数（如线程数为 0）、数组越界（逻辑层面）、函数调用时机错误 |
| `std::invalid_argument`  | `<stdexcept>`    | 无效参数（`logic_error` 子类）                  | 向函数传入不符合要求的参数（如 `atoi("abc")`、线程池线程数为 0） |
| `std::out_of_range`      | `<stdexcept>`    | 越界访问（`logic_error` 子类）                  | `std::vector::at(i)` 越界、`std::string::substr` 越界        |
| `std::bad_alloc`         | `<new>`          | 内存分配失败（`exception` 直接子类）            | `new`/`std::make_shared`/`std::make_unique` 分配内存时堆不足 |
| `std::bad_cast`          | `<typeinfo>`     | 动态类型转换失败（`exception` 直接子类）        | `dynamic_cast<Derived*>(base_ptr)` 转换失败（指针版返回 null，引用版抛此异常） |
| `std::bad_function_call` | `<functional>`   | 调用空的 `std::function`                        | `std::function<void()> f; f();`（f 未绑定任何可调用对象）    |
| `std::system_error`      | `<system_error>` | 系统调用 / 库函数错误（关联错误码）             | 线程 `join()` 非法、文件打开失败、socket 操作错误（可获取 `errno`/ 系统错误码） |

#### 自定义异常（基于标准异常扩展）

业务场景中常自定义异常，继承 `std::runtime_error`/`std::logic_error`，便于分类处理：

```cpp
// 自定义任务提交异常（继承运行时异常）
class TaskSubmitError : public std::runtime_error {
public:
    using std::runtime_error::runtime_error; // 复用父类构造函数
};

// 使用：抛出自定义异常
throw TaskSubmitError("任务依赖资源未初始化，提交失败");
```

### 二、异常的核心使用场景

异常的核心价值是**处理 “非正常但可预见” 的错误**（而非流程控制），以下是最常见的使用场景：

#### 1. 参数 / 状态合法性检查（最基础）

在函数入口检查参数 / 对象状态，非法时抛异常（替代返回错误码，更直观）：

```cpp
// 线程池构造函数：参数非法抛逻辑异常
explicit ThreadPool(size_t thread_num) {
    if (thread_num == 0) {
        throw std::invalid_argument("线程池线程数不能为0"); // 逻辑错误：参数非法
    }
}

// submit函数：状态非法抛运行时异常
if (stop_flag_) {
    throw std::runtime_error("线程池已停止，无法提交任务"); // 运行时错误：状态非法
}
```

#### 2. 资源操作失败（文件 / 网络 / 内存）

资源操作依赖外部环境，失败时抛异常（避免层层返回错误码）：

```cpp
#include <fstream>
#include <stdexcept>

std::string read_file(const std::string& path) {
    std::ifstream file(path);
    if (!file.is_open()) {
        throw std::runtime_error("文件打开失败：" + path); // 运行时错误：外部资源失败
    }
    // 读取逻辑...
}
```

#### 3. 异步任务异常传递（线程池 / 多线程）

通过 `std::future` 将异步任务的异常传递给调用者（线程池 `submit` 核心逻辑）：

```cpp
// 提交一个可能抛异常的异步任务
auto fut = pool.submit([]() {
    if (/* 业务错误 */) {
        throw std::logic_error("任务执行逻辑错误");
    }
    return 1 + 2;
});

// 调用者捕获异常
try {
    int res = fut.get(); // 任务异常会在此处重抛
} catch (const std::logic_error& e) {
    std::cerr << "任务异常：" << e.what() << std::endl;
}
```

#### 4. 内存分配失败

`new`/`make_shared` 等分配内存失败时会抛 `std::bad_alloc`，需捕获以优雅处理：

```cpp
try {
    int* arr = new int[1000000000000]; // 超大数组，分配失败
} catch (const std::bad_alloc& e) {
    std::cerr << "内存分配失败：" << e.what() << std::endl;
    // 降级处理：如减小内存占用、释放其他资源后重试
}
```

#### 5. 类型转换 / 函数调用错误

处理动态类型转换、`std::function` 调用等场景的异常：

```cpp
#include <functional>
#include <typeinfo>

// 1. 动态转换异常
class Base {};
class Derived : public Base {};
void cast_test(Base* ptr) {
    try {
        Derived& d = dynamic_cast<Derived&>(*ptr); // 指针版返回null，引用版抛异常
    } catch (const std::bad_cast& e) {
        std::cerr << "类型转换失败：" << e.what() << std::endl;
    }
}

// 2. 空function调用异常
void func_test() {
    std::function<void()> f; // 未绑定任何函数
    try {
        f();
    } catch (const std::bad_function_call& e) {
        std::cerr << "函数调用失败：" << e.what() << std::endl;
    }
}
```

### 三、异常使用的核心语法与最佳实践

#### 1. 基础语法：抛异常（throw）+ 捕获（try-catch）

```cpp
try {
    // 可能抛异常的代码块
    if (/* 错误条件 */) {
        throw std::runtime_error("错误描述"); // 抛出异常，终止当前逻辑
    }
} 
// 按“子类→父类”顺序捕获（先捕获具体异常，再捕获通用异常）
catch (const std::invalid_argument& e) { // 捕获特定异常
    std::cerr << "参数错误：" << e.what() << std::endl;
}
catch (const std::runtime_error& e) { // 捕获运行时异常
    std::cerr << "运行时错误：" << e.what() << std::endl;
}
catch (const std::exception& e) { // 兜底：捕获所有标准异常
    std::cerr << "标准异常：" << e.what() << std::endl;
}
catch (...) { // 捕获所有异常（包括非标准异常），最后兜底
    std::cerr << "未知异常" << std::endl;
    throw; // 重新抛出，让上层处理（可选）
}
```

#### 2. 最佳实践（避坑指南）

##### （1）异常只用于 “错误处理”，不用于流程控制

❌ 错误示例：用异常控制循环

```cpp
// 反例：用异常代替break，可读性差、性能低
try {
    for (int i = 0; ; i++) {
        if (i == 10) throw std::runtime_error("结束循环");
    }
} catch (...) {}
```

✅ 正确示例：异常仅处理意外错误

```cpp
for (int i = 0; i < 100; i++) {
    try {
        submit_task(i); // 仅当任务提交失败时抛异常
    } catch (const std::runtime_error& e) {
        std::cerr << "第" << i << "个任务提交失败：" << e.what() << std::endl;
        break; // 异常处理后，用正常逻辑控制流程
    }
}
```

##### （2）捕获异常时使用引用（避免切片）

捕获异常时必须用 `const &`（或 `&`），否则会触发 “对象切片”（仅保留父类部分，丢失子类信息）：

❌ 错误：值捕获导致切片

```cpp
catch (std::runtime_error e) { // 拷贝异常对象，子类信息丢失
    std::cerr << e.what() << std::endl; // 无法获取自定义子类的扩展信息
}
```

✅ 正确：引用捕获

```cpp
catch (const std::runtime_error& e) { // 无拷贝，保留完整子类信息
    std::cerr << e.what() << std::endl;
}
```

##### （3）RAII 保证异常安全（避免资源泄露）

异常抛出时，局部对象会析构（RAII 特性），因此需用 RAII 管理资源（如锁、文件句柄、智能指针）：

```cpp
void safe_func() {
    std::unique_lock<std::mutex> lock(mtx_); // RAII锁：异常时自动解锁
    std::ofstream file("data.txt"); // RAII文件：异常时自动关闭
    if (!file) throw std::runtime_error("文件打开失败");
    // 即使后续抛异常，锁和文件都会被正确释放
}
```

##### （4）明确异常类型，避免 “一刀切” 捕获

❌ 错误：直接捕获所有异常，无法区分错误类型

```cpp
try {
    submit_task();
} catch (...) { // 无法知道是参数错误还是线程池停止
    std::cerr << "任务提交失败" << std::endl;
}
```

✅ 正确：按类型捕获，精细化处理

```cpp
try {
    submit_task();
} catch (const std::invalid_argument& e) {
    std::cerr << "参数错误：" << e.what() << "，请检查参数" << std::endl;
} catch (const std::runtime_error& e) {
    std::cerr << "运行时错误：" << e.what() << "，请重启线程池" << std::endl;
}
```

##### （5）异常说明（C++17 后弱化，了解即可）

早期用 `noexcept` 标记 “不抛异常” 的函数，帮助编译器优化：

```cpp
// 标记该函数不会抛出任何异常
void safe_func() noexcept {
    // 若内部抛异常，程序直接终止（std::terminate）
}
```

### 四、异常 vs 错误码（选型建议）

| 维度     | 异常                               | 错误码                         |
| -------- | ---------------------------------- | ------------------------------ |
| 可读性   | 高（错误逻辑与业务逻辑分离）       | 低（需层层判断返回值）         |
| 性能     | 异常抛出时开销高（正常路径无开销） | 无额外开销（但需频繁判断）     |
| 适用场景 | 复杂业务、资源操作、异步任务       | 底层函数、性能敏感场景、C 兼容 |
| 资源安全 | 易保证（RAII）                     | 易泄露（需手动释放）           |

✅ 优先用异常的场景：

- 业务逻辑复杂（如线程池、框架层）；
- 资源操作（文件、网络、内存）；
- 异步任务（通过 `future` 传递异常）。

✅ 优先用错误码的场景：

- 底层工具函数（如内存拷贝、数值计算）；
- 性能敏感场景（如高频循环）；
- 需兼容 C 代码的场景。

### 五、总结

1. **常见异常**：核心用 `std::invalid_argument`（参数错）、`std::runtime_error`（运行时错）、`std::bad_alloc`（内存错），自定义异常继承标准异常；
2. **核心用法**：参数 / 状态检查、资源操作失败、异步任务异常传递；
3. **最佳实践**：异常仅处理错误、引用捕获、RAII 保证资源安全、按类型精细化捕获；
4. **选型原则**：复杂业务用异常，底层 / 高性能场景用错误码。

异常的本质是 “让错误处理与正常逻辑分离”，合理使用能大幅提升代码可读性和健壮性，而滥用（如流程控制）则会导致代码混乱，需把握好边界。





## 代码示例

简单错误--除0异常

```cpp
#include <iostream>
#include <exception>

int devide(int a, int b) {
    if(b == 0) {
        throw std::invalid_argument("除数不能为零");    // throw语句抛出异常
    }
    return a / b;
}

int main() {
    try {
        int a, b;
        std::cout << "请输入两个整数（被除数和除数）：";
        std::cin >> a >> b;
        int result = devide(a, b);
        std::cout << "结果是: " << result << std::endl;
    } catch (const std::invalid_argument& e) { // 捕获除零异常,这里引用是为了避免拷贝开销
        std::cerr << "错误: " << e.what() << std::endl; // cerr用于输出错误信息
    }
}
```



```cpp
#include <iostream>
#include <vector>
#include <string>
#include <exception>    // std::exception
#include <stdexcept>    // std::runtime_error, std::invalid_argument, std::out_of_range
#include <functional>
#include <memory>
#include <thread>
#include <future>
#include <fstream> // 文件操作
#include <system_error> // std::system_error
#include <mutex>
#include <condition_variable>

// -------------------------- 1. 自定义异常（继承标准异常） --------------------------
// 业务场景自定义异常：任务执行异常
class TaskExecuteError : public std::runtime_error {
public:
    // 复用父类构造函数，支持自定义错误信息
    using std::runtime_error::runtime_error;

    // 扩展：自定义错误码（业务场景常用）
    int get_error_code() const { return error_code_; }
    TaskExecuteError(const std::string& msg, int code) 
        : std::runtime_error(msg), error_code_(code) {}

private:
    int error_code_ = 0;
};

// -------------------------- 2. 简易线程池（复用之前的逻辑，用于异步异常演示） --------------------------
class ThreadPool {
public:
    explicit ThreadPool(size_t thread_num) : stop_flag_(false) {
        // 参数检查：抛invalid_argument（逻辑错误）
        if (thread_num == 0) {
            throw std::invalid_argument("ThreadPool: 线程数不能为0！");
        }

        // 内存分配失败可能抛bad_alloc
        try {
            for (size_t i = 0; i < thread_num; ++i) {
                workers_.emplace_back([this]() {
                    while (true) {
                        std::unique_ptr<std::function<void()>> task;
                        {
                            std::unique_lock<std::mutex> lock(mtx_);
                            cv_.wait(lock, [this]() {
                                return stop_flag_ || !tasks_.empty();
                            });

                            if (stop_flag_ && tasks_.empty()) return;

                            task = std::move(tasks_.front());
                            tasks_.pop();
                        }
                        // 执行任务：异常会被packaged_task捕获，通过future传递
                        (*task)();
                    }
                });
            }
        } catch (const std::bad_alloc& e) {
            // 捕获内存分配异常，增强错误信息后重新抛出
            throw std::runtime_error("ThreadPool: 创建线程时内存分配失败 - " + std::string(e.what()));
        }
    }

    // 提交任务：泛型+future异常传递
    template<class F, class... Args>
    auto submit(F&& f, Args&&... args) -> std::future<typename std::result_of<F(Args...)>::type> {
        using ReturnType = typename std::result_of<F(Args...)>::type;

        // 包装任务：packaged_task不可拷贝，用shared_ptr
        auto task = std::make_shared<std::packaged_task<ReturnType()>>(
            std::bind(std::forward<F>(f), std::forward<Args>(args)...)
        );

        std::future<ReturnType> res = task->get_future();

        {
            std::unique_lock<std::mutex> lock(mtx_);
            // 状态检查：抛runtime_error（运行时错误）
            if (stop_flag_) {
                throw std::runtime_error("ThreadPool: 线程池已停止，无法提交任务！");
            }

            // 任务入队
            tasks_.emplace(std::make_unique<std::function<void()>>([task]() {
                (*task)();
            }));
        }

        cv_.notify_one();
        return res;
    }

    // 停止线程池
    void stop() {
        {
            std::unique_lock<std::mutex> lock(mtx_);
            stop_flag_ = true;
        }
        cv_.notify_all();

        // 等待线程退出：处理system_error（线程join失败）
        for (std::thread& worker : workers_) {
            try {
                if (worker.joinable()) {
                    worker.join();
                }
            } catch (const std::system_error& e) {
                throw std::runtime_error("ThreadPool: 线程join失败 - " + std::string(e.what()));
            }
        }
    }

    ~ThreadPool() {
        try {
            stop();
        } catch (const std::exception& e) {
            std::cerr << "ThreadPool析构异常：" << e.what() << std::endl;
        }
    }

private:
    std::vector<std::thread> workers_;
    std::queue<std::unique_ptr<std::function<void()>>> tasks_;
    std::mutex mtx_;
    std::condition_variable cv_;
    bool stop_flag_;
};

// -------------------------- 3. 工具函数：演示各类异常场景 --------------------------
// 函数1：文件操作（抛runtime_error）
std::string read_file(const std::string& path) {
    std::ifstream file(path);
    // 资源操作失败：抛运行时异常
    if (!file.is_open()) {
        throw std::runtime_error("read_file: 无法打开文件 - " + path);
    }

    std::string content, line;
    while (std::getline(file, line)) {
        content += line + "\n";
    }
    return content;
}

// 函数2：数组越界检查（抛out_of_range）
int get_vector_elem(const std::vector<int>& vec, size_t idx) {
    // 逻辑错误：越界访问抛out_of_range
    if (idx >= vec.size()) {
        throw std::out_of_range("get_vector_elem: 索引越界！idx=" + std::to_string(idx) + ", size=" + std::to_string(vec.size()));
    }
    return vec[idx];
}

// 函数3：空function调用（抛bad_function_call）
void call_empty_function() {
    std::function<void()> empty_func;
    try {
        empty_func(); // 空function调用抛异常
    } catch (const std::bad_function_call& e) {
        throw std::runtime_error("call_empty_function: 调用空函数 - " + std::string(e.what()));
    }
}

// 函数4：动态类型转换（抛bad_cast）
class Base { virtual void dummy() {} };
class Derived : public Base { public: void func() { std::cout << "Derived::func" << std::endl; } };
void test_dynamic_cast(Base* ptr) {
    try {
        // 引用版dynamic_cast失败抛bad_cast
        Derived& d = dynamic_cast<Derived&>(*ptr);
        d.func();
    } catch (const std::bad_cast& e) {
        throw std::runtime_error("test_dynamic_cast: 类型转换失败 - " + std::string(e.what()));
    }
}

// -------------------------- 4. 主函数：异常捕获与处理演示 --------------------------
int main() {
    // 核心：按「子类→父类→兜底」顺序捕获异常
    try {
        // ========== 场景1：参数错误（invalid_argument） ==========
        ThreadPool pool(0); // 线程数为0，抛invalid_argument

    } catch (const std::invalid_argument& e) {
        std::cerr << "\n【捕获invalid_argument】: " << e.what() << std::endl;

        // 修复参数后重新尝试
        try {
            ThreadPool pool(2); // 创建2个线程的线程池
            std::cout << "\n线程池创建成功（2个工作线程）" << std::endl;

            // ========== 场景2：异步任务异常（自定义异常+future传递） ==========
            auto fut1 = pool.submit([]() {
                // 任务内部抛自定义异常
                throw TaskExecuteError("任务执行失败：除数为0", 1001);
                return 10 / 0;
            });

            // ========== 场景3：文件操作异常（runtime_error） ==========
            try {
                std::string content = read_file("non_exist_file.txt"); // 文件不存在
                std::cout << "文件内容：" << content << std::endl;
            } catch (const std::runtime_error& e) {
                std::cerr << "\n【捕获runtime_error（文件操作）】: " << e.what() << std::endl;
            }

            // ========== 场景4：数组越界（out_of_range） ==========
            std::vector<int> vec = {1, 2, 3};
            try {
                int elem = get_vector_elem(vec, 5); // 索引5越界
                std::cout << "数组元素：" << elem << std::endl;
            } catch (const std::out_of_range& e) {
                std::cerr << "\n【捕获out_of_range】: " << e.what() << std::endl;
            }

            // ========== 场景5：空function调用（bad_function_call） ==========
            try {
                call_empty_function();
            } catch (const std::runtime_error& e) {
                std::cerr << "\n【捕获bad_function_call（包装后）】: " << e.what() << std::endl;
            }

            // ========== 场景6：动态类型转换（bad_cast） ==========
            Base base_obj;
            try {
                test_dynamic_cast(&base_obj); // Base转Derived失败
            } catch (const std::runtime_error& e) {
                std::cerr << "\n【捕获bad_cast（包装后）】: " << e.what() << std::endl;
            }

            // ========== 场景7：获取异步任务异常（自定义异常） ==========
            try {
                fut1.get(); // future.get()重抛任务内部的异常
            } catch (const TaskExecuteError& e) {
                std::cerr << "\n【捕获自定义异常TaskExecuteError】: " << e.what() 
                          << " | 错误码：" << e.get_error_code() << std::endl;
            }

            // ========== 场景8：线程池停止后提交任务（runtime_error） ==========
            pool.stop(); // 停止线程池
            try {
                pool.submit([]() { std::cout << "新任务" << std::endl; });
            } catch (const std::runtime_error& e) {
                std::cerr << "\n【捕获runtime_error（线程池停止）】: " << e.what() << std::endl;
            }

        } catch (const std::bad_alloc& e) {
            // ========== 场景9：内存分配失败（bad_alloc） ==========
            std::cerr << "\n【捕获bad_alloc】: " << e.what() << std::endl;
        } catch (const std::system_error& e) {
            // ========== 场景10：系统调用异常（system_error） ==========
            std::cerr << "\n【捕获system_error】: " << e.what() << std::endl;
        }

    } catch (const std::exception& e) {
        // ========== 兜底：捕获所有标准异常 ==========
        std::cerr << "\n【捕获std::exception（兜底）】: " << e.what() << std::endl;
    } catch (...) {
        // ========== 终极兜底：捕获所有未知异常 ==========
        std::cerr << "\n【捕获未知异常】: 发生未定义的异常类型" << std::endl;
        throw; // 可选：重新抛出，让上层处理
    }

    std::cout << "\n程序正常退出" << std::endl;
    return 0;
}
```

