# EventLoopThreadPool 类 成员变量 & 成员函数 作用总结

### 一、成员变量

#### 1. 核心循环与标识相关

| 变量名        | 作用                                                         | 调用 / 生效时机                                              |
| ------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `baseLoop_`   | 指向用户创建的主 EventLoop（mainLoop），是线程池的基准循环   | 构造函数初始化；若线程池线程数为 1，所有任务直接复用该 loop；多线程模式下，作为 “分发者” 为新连接分配 subLoop |
| `started_`    | 标识线程池是否已启动（所有 subLoop 线程是否已创建并运行）    | 调用`start()`时置为`true`；线程池生命周期内，`started()`会返回该值供外部校验状态 |
| `numThreads_` | 线程池内要创建的 IO 线程（EventLoopThread）数量              | 构造后可通过`setThreadNum()`修改；`start()`执行时，根据该值创建对应数量的 EventLoopThread 实例 |
| `next_`       | 轮询索引，记录下一个新连接要分配的 subLoop 在`loops_`中的位置 | 构造时初始化为 0；每次调用`getNextLoop()`时，先返回当前`next_`对应的 loop，再对`next_`做自增取模（保证轮询分发） |

#### 2. 名称与容器相关

| 变量名     | 作用                                                         | 调用 / 生效时机                                              |
| ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `name_`    | 线程池名称，作为内部 EventLoopThread 命名的基准              | 构造函数初始化；`name()`方法返回该值，供外部标识线程池，也用于给每个 EventLoopThread 生成唯一名称（如 “pool-name-thread-0”） |
| `threads_` | 存储线程池内所有 EventLoopThread 实例的智能指针容器          | `start()`执行时，根据`numThreads_`创建 EventLoopThread 并添加到该容器；线程池析构时，智能指针自动释放所有 EventLoopThread |
| `loops_`   | 存储线程池内所有 subLoop（EventLoopThread 线程函数创建的EventLoop对象）指针 | `start()`中启动每个 EventLoopThread 后，从线程中获取其创建的 EventLoop 并添加到该容器；`getNextLoop()`/`getAllLoops()`从该容器读取 subLoop |

### 二、成员函数

#### 1. 构造 / 析构函数

| 函数名                                                       | 作用                                                         | 调用时机                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `EventLoopThreadPool(EventLoop *baseLoop, const std::string &nameArg)` | 初始化线程池核心变量：1. 绑定主循环`baseLoop_`；2. 赋值线程池名称`name_`；3. 初始化`started_`为`false`、`next_`为 0、`numThreads_`为默认值（通常 0）；4. 清空`threads_`和`loops_`容器 | 创建 EventLoopThreadPool 实例时调用（通常在 mainLoop 所在线程初始化） |
| `~EventLoopThreadPool()`                                     | 释放线程池资源：1. `threads_`中的智能指针自动析构，触发 EventLoopThread 析构，停止对应的 subLoop 线程；2. 清空`loops_`（仅存储指针，不负责释放，由 EventLoopThread 管理） | 线程池实例生命周期结束时调用（如程序退出、mainLoop 销毁时）  |

> 析构函数为什么是空：
>
> * thread_数组存储的是用独占智能指针管理的EventloopThread对象，不需要手动析构
> * eventloop是eventloopthread对象创建的新线程在栈上创建的局部变量，只能由新线程管理释放内存（绑定栈的生命周期），eventloopthread对象所在的主线程在析构时通过（修改quit_标志位 + 唤醒）提醒新线程退出事件循环；loops_存储的只是对应事件循环指针，不需要释放

#### 2. 线程池配置与状态查询

| 函数名                         | 作用                           | 调用时机                                                     | 内部逻辑 / 触发后续操作                                      |
| ------------------------------ | ------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `setThreadNum(int numThreads)` | 设置线程池要创建的 IO 线程数量 | 线程池启动（`start()`）前调用（启动后修改无效）；通常在初始化线程池后、启动前配置线程数｜ 🤫因为 `start()` 是实际创建线程 / EventLoop 的入口。 | 直接赋值`numThreads_`，无其他逻辑，仅为`start()`提供创建线程的数量依据 |
| `started() const`              | 返回线程池是否已启动的状态     | 外部需要校验线程池是否完成初始化时调用（如给新连接分配 subLoop 前） | 直接返回`started_`的值，无其他逻辑                           |
| `name() const`                 | 返回线程池的名称               | 外部需要标识线程池、或给日志 / 监控打标签时调用              | 直接返回`name_`的值；EventLoopThread 创建时，也会基于该名称生成子线程名称 |

> 设置线程池中IO线程的数量（需在start()之前调用）
>
> ​    1. numThreads = 0（默认值）：不创建任何IO线程，线程池无独立的EventLoop；
>
> ​       所有事件循环任务均复用用户传入的baseLoop_（主线程的EventLoop），getNextLoop()始终返回baseLoop_；
>
> ​    2. numThreads > 0：start()时会创建对应数量的EventLoopThread（IO线程），每个线程内部运行一个独立的EventLoop；
>
> ​       这些新创建的EventLoop会存入loops_列表，baseLoop_仅作为主线程的EventLoop，不会被分配给任何子线程；
>
> ​    3. 后续调用getNextLoop()时，会轮询返回loops_中的子线程EventLoop（numThreads>0时），实现多线程事件循环分发；

#### 3. 线程池启动（核心）

| 函数名                                                       | 作用                                                         | 调用时机                                                     | 内部逻辑 / 触发后续操作                                      |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `start(const ThreadInitCallback &cb = ThreadInitCallback())` | 启动线程池，创建并启动指定数量的 IO 线程：1. 校验`started_`，避免重复启动；2. 若`numThreads_`为 0，仅标记`started_`为`true`（复用 baseLoop_）；3. 若`numThreads_ > 0`，循环创建 EventLoopThread 实例：- 给每个线程分配基于`name_`的唯一名称；- 启动线程（EventLoopThread::startLoop ()），获取线程内创建的 subLoop；- 若传入初始化回调`cb`，在 subLoop 所属线程执行`cb(subLoop)`；- 将 EventLoopThread 加入`threads_`，subLoop 加入`loops_`；4. 置`started_`为`true` | 程序初始化阶段，mainLoop 准备好后调用（如 TcpServer 启动时）；必须在`setThreadNum()`之后调用 | 核心是完成 “线程创建 + subLoop 初始化 + 容器填充”，是线程池可用的前提；初始化回调`cb`用于给每个 subLoop 做定制化初始化（如设置定时器、初始化日志） |

#### 4. subLoop 分配与获取

| 函数名          | 作用                                                         | 调用时机                                                     | 内部逻辑 / 触发后续操作                                      |
| --------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `getNextLoop()` | 以轮询方式获取下一个可用的 subLoop（核心分发逻辑）           | 新连接到来时（Acceptor 处理完连接后），mainLoop 调用该方法为新连接分配 subLoop | 1. 若线程池未启动 / 线程数为 0，直接返回`baseLoop_`；2. 若有 subLoop，先取`next_`对应的`loops_[next_]`；3. `next_`自增并对`loops_.size()`取模（保证循环轮询）；4. 返回选中的 subLoop |
| `getAllLoops()` | 获取线程池内所有的 subLoop（包含 baseLoop_ 吗？不，仅返回`loops_`） | 外部需要遍历所有 subLoop 执行操作时调用（如批量设置定时器、广播任务） | 1. 校验`started_`，确保线程池已启动；2. 直接返回`loops_`容器的拷贝（或引用，取决于实现）；3. 若线程数为 0，返回的容器为空（需外部自行处理 baseLoop_） |

### 补充说明

1. **线程池核心设计逻辑**：
   - 单线程模式（`numThreads_ = 0/1`）：所有 IO 事件直接复用`baseLoop_`，无额外线程创建；
   - 多线程模式（`numThreads_ > 1`）：创建多个 EventLoopThread，每个线程持有一个 subLoop，mainLoop 仅负责 “接受连接” 并通过`getNextLoop()`轮询分发连接给 subLoop，实现 “主从分离”。
2. **线程安全注意**：
   - `next_`的自增取模操作需保证原子性（muduo 实现中通常加锁或用原子变量），避免多线程调用`getNextLoop()`时索引错乱；
   - `start()`仅能调用一次，`started_`作为状态锁，防止重复创建线程。
3. **回调的作用**：
   - `ThreadInitCallback`是给每个 subLoop 启动后提供的初始化入口，比如可以在回调中给 subLoop 设置默认的定时器、初始化日志上下文、注册全局事件等，保证每个 subLoop 启动后具备相同的初始状态。

