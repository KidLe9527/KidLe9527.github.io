## EventLoopThread 类成员变量与成员函数作用总结

### 一、成员变量

| 成员变量  |              类型              |                             作用                             |                      初始化 / 赋值时机                       |                       访问 / 修改条件                        |
| :-------: | :----------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
|   loop_   |           EventLoop*           | 指向与当前线程绑定的 EventLoop 对象，是线程内 EventLoop 的核心引用 | 1. 构造函数初始化为 nullptr；2. threadFunc () 中创建 EventLoop 对象后，通过互斥锁保护赋值为该对象地址；3. loop.loop () 退出后（线程退出前）重新赋值为 nullptr | 1. 读操作：startLoop () 中等待条件变量唤醒后读取、析构函数中判空；2. 写操作：仅在 threadFunc () 中，且需持有 mutex_互斥锁；3. 所有访问均需保证线程安全，写操作必须加锁 |
| exiting_  |              bool              |        标记线程是否需要退出，用于析构函数控制退出逻辑        |     1. 构造函数初始化为 false；2. 析构函数中赋值为 true      | 1. 读操作：析构函数中判断是否需要退出 EventLoop；2. 写操作：仅析构函数中赋值，无并发写场景（析构仅调用一次） |
|  thread_  |       Thread（自定义类）       |     封装底层线程对象，绑定线程执行函数并管理线程生命周期     | 构造函数中初始化，绑定 threadFunc () 为线程执行函数，传入线程名称 | 1. startLoop () 中调用 start () 启动线程；2. 析构函数中调用 join () 等待线程退出；3. 仅在 EventLoopThread 的成员函数中调用其接口 |
|  mutex_   |           std::mutex           | 保护 loop_的读写和条件变量 cond_的等待 / 唤醒，保证线程安全  |                      构造函数默认初始化                      | 1. 所有访问 loop_的场景（startLoop ()、threadFunc ()）均需通过 unique_lock 加锁；2. 条件变量 cond_的 wait/notify 操作必须配合该互斥锁 |
|   cond_   |    std::condition_variable     | 用于线程间同步，等待 loop_初始化完成后唤醒 startLoop () 所在线程 |                      构造函数默认初始化                      | 1. wait 操作：startLoop () 中持有 mutex_锁后等待，直到 loop_非空；2. notify 操作：threadFunc () 中初始化 loop_后，持有 mutex_锁并唤醒等待线程 |
| callback_ | ThreadInitCallback（函数对象） | 线程初始化回调函数，在 EventLoop 创建后、loop 循环启动前执行自定义初始化逻辑 |           构造函数中接收外部传入的回调函数（可空）           | 仅在 threadFunc () 中，创建 EventLoop 对象后、loop_赋值前执行，若回调函数非空则调用 |

### 二、成员函数

#### 1. 构造函数 `EventLoopThread(const ThreadInitCallback &cb, const std::string &name)`

- **作用**：初始化所有成员变量，绑定线程执行函数，保存初始化回调和线程名称。

- **调用时机**：创建 EventLoopThread 对象时调用，仅执行一次。

- **执行逻辑**：

  - 将 loop_置为 nullptr、exiting_置为 false；
  - 初始化 thread_，绑定 threadFunc () 为线程执行函数，传入线程名称；
  - 初始化 mutex_、cond_（默认构造）；
  - 保存传入的初始化回调函数 callback_。

  

#### 2. 析构函数 `~EventLoopThread()`

- **作用**：安全退出线程和 EventLoop，释放资源，避免内存泄漏和线程残留。

- **调用时机**：EventLoopThread 对象销毁时调用，仅执行一次。

- **执行条件**：对象生命周期结束（如超出作用域、delete 调用）。

- **执行逻辑**：

  - 标记 exiting_为 true，告知线程需要退出；
  - 若 loop_非空（线程已启动且 EventLoop 创建完成），调用 loop_->quit () 退出 EventLoop 的循环；
  - 调用 thread_.join () 等待线程完全退出，避免僵尸线程。

  

#### 3. 启动线程并获取 EventLoop `EventLoop *startLoop()`

- **作用**：启动底层线程，等待线程内 EventLoop 初始化完成后，返回该 EventLoop 的指针。

- **调用时机**：外部需要获取与该线程绑定的 EventLoop 时调用（如创建 EventLoopThreadPool 时）。

- **执行条件**：线程未启动（thread_未调用 start ()），且对象未析构。

- **执行逻辑**：

  - 调用 thread_.start () 启动底层线程，线程开始执行 threadFunc ()；
  - 加锁 mutex_，调用 cond_.wait () 等待，直到 loop_非空（threadFunc () 完成 loop_赋值）；
  - 读取 loop_的值并返回，解锁 mutex_。

  

#### 4. 线程执行函数 `void threadFunc()`

- **作用**：作为 thread_绑定的执行函数，在新线程中创建 EventLoop、执行初始化回调、启动事件循环，是线程的核心逻辑。

- **调用时机**：thread_.start () 启动线程后，由操作系统调度执行，仅在新线程中运行。

- **执行条件**：startLoop () 调用 thread_.start () 后触发，且 exiting_为 false（未析构）。

- **执行逻辑**：

  - 创建局部 EventLoop 对象（栈上对象，与线程一一对应，one loop per thread）；
  - 若 callback_非空，执行回调函数，传入 EventLoop 指针做自定义初始化；
  - 加锁 mutex_，将 loop_赋值为当前 EventLoop 对象的地址，调用 cond_.notify_one () 唤醒 startLoop () 所在线程；
  - 调用 loop.loop () 启动事件循环（底层 Poller::poll () 循环），直到 loop.quit () 被调用；
  - 事件循环退出后，加锁 mutex_，将 loop_置为 nullptr，线程执行完毕。

  

### 三、核心逻辑时序（补充说明）

1. 外部创建 EventLoopThread 对象 → 构造函数初始化成员变量；
2. 外部调用 startLoop () → 启动 thread_线程，阻塞等待 loop_初始化；
3. 新线程执行 threadFunc () → 创建 EventLoop → 执行初始化回调 → 赋值 loop_并唤醒 startLoop ()；
4. startLoop () 被唤醒 → 返回 loop_指针给外部；
5. threadFunc () 继续执行 loop.loop () → 进入事件循环；
6. 外部析构 EventLoopThread → 标记 exiting_为 true → 调用 loop_->quit () 退出事件循环 → 等待线程 join () → 线程执行完毕，loop_置为 nullptr。

