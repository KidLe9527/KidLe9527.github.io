# TcpServer 类 成员变量 & 成员函数 作用总结

### 一、核心设计背景

`TcpServer` 是 muduo 网络库对外暴露的**服务器核心入口类**，封装了 TCP 服务器的启动、监听、连接管理、线程池调度等核心能力，是用户编写服务器程序的顶层抽象。其核心设计遵循 **One Loop Per Thread + Reactor** 模型：

- 主线程（mainloop/baseloop）：由用户传入，负责监听新连接（`Acceptor` 绑定）；
- 子线程（subloop）：由 `EventLoopThreadPool` 管理，负责处理已建立连接的读写事件；
- 核心协作链路：用户初始化 `TcpServer` → 调用 `start()` 启动监听 → `Acceptor` 接收新连接 → `TcpServer` 创建 `TcpConnection` → 将连接分配到子线程 `subloop` → 由 `TcpConnection` 处理具体的连接生命周期和数据读写。
- `TcpServer` 是 muduo 服务器编程的 “总控中心”，屏蔽了底层 `Acceptor`、`EventLoopThreadPool`、`TcpConnection` 的细节，为用户提供简洁、高层的服务器开发接口。

### 二、成员变量

#### 1. 核心事件循环与标识

| 变量名     | 作用                                                         | 调用 / 生效时机                                              |
| ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `loop_`    | 服务器主线程的事件循环（`EventLoop*`，即 baseloop），仅负责监听新连接 | 构造函数初始化；`Acceptor` 绑定到该 `loop_`；`start()`、`newConnection()` 等核心操作确保在 `loop_` 线程执行 |
| `ipPort_`  | 服务器监听的 IP + 端口（字符串形式，如 `127.0.0.1:8080`），用于标识服务器实例 | 构造函数初始化；日志输出、连接命名（如 `TcpConnection` 的 `name_`）时使用 |
| `name_`    | 服务器名称（用户自定义），用于日志 / 调试区分不同服务器实例  | 构造函数初始化；日志输出、`TcpConnection` 命名前缀（如 `name_ + "-connection-1"`）时使用 |
| `started_` | 服务器启动状态标识（原子类型 `std::atomic_int`），线程安全的启动标记 | `start()` 中置为 1；多次调用 `start()` 时校验状态，避免重复启动；线程安全保证多线程调用 `start()` 无副作用 |

#### 2. 连接监听与管理

| 变量名         | 作用                                                         | 调用 / 生效时机                                              |
| -------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `acceptor_`    | 封装监听套接字（`listenfd`）和新连接接收逻辑，仅运行在 `loop_`（baseloop） | 构造函数初始化；`start()` 中调用 `acceptor_->listen()` 启动监听；`newConnection()` 作为 `Acceptor` 的回调接收新连接的 `sockfd` |
| `connections_` | 保存所有已建立的 TCP 连接（`unordered_map<string, TcpConnectionPtr>`），键为 `TcpConnection` 的唯一名称 | `newConnection()` 中添加新连接；`removeConnection()` 中移除断开的连接；支持遍历 / 管理所有连接（如批量关闭） |
| `nextConnId_`  | 连接 ID 生成器（自增整数），用于生成 `TcpConnection` 的唯一名称 | `newConnection()` 中为每个新连接分配唯一 ID（如 `connection-${nextConnId_}-${timestamp}`）；分配后自增，保证唯一性 |

#### 3. 线程池与回调函数

| 变量名                   | 作用                                                         | 调用 / 生效时机                                              |
| ------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `threadPool_`            | 事件循环线程池（`EventLoopThreadPool`），管理子线程的 `subloop` | 构造函数初始化；`setThreadNum()` 设置子线程数量；`start()` 中启动线程池；`newConnection()` 中从线程池获取 `subloop` 分配给新连接 |
| `numThreads_`            | 线程池中子线程的数量（默认 0，即单线程模式）                 | `setThreadNum()` 赋值；`start()` 中传递给 `threadPool_->setThreadNum()`；用于控制 Reactor 子线程数量 |
| `threadInitCallback_`    | 子线程 `subloop` 初始化时的回调（用户自定义子线程初始化逻辑） | `start()` 启动线程池时，传递给 `EventLoopThreadPool`；子线程创建 `EventLoop` 后触发该回调 |
| `connectionCallback_`    | 连接建立 / 断开时的回调（用户自定义），透传给 `TcpConnection` | `newConnection()` 中设置到 `TcpConnection`；由 `TcpConnection` 在连接建立 / 断开时触发 |
| `messageCallback_`       | 连接读取到数据时的回调（用户自定义），透传给 `TcpConnection` | 同上；由 `TcpConnection` 在 `handleRead()` 中触发            |
| `writeCompleteCallback_` | 数据发送完成后的回调（用户自定义），透传给 `TcpConnection`   | 同上；由 `TcpConnection` 在 `handleWrite()` 中触发           |

### 三、成员函数

#### 1. 构造 / 析构函数

| 函数名                                                       | 作用                 | 调用时机                                          | 内部逻辑 / 核心设计                                          |
| ------------------------------------------------------------ | -------------------- | ------------------------------------------------- | ------------------------------------------------------------ |
| `TcpServer(EventLoop *loop, const InetAddress &listenAddr, const std::string &nameArg, Option option = kNoReusePort)` | 初始化服务器核心资源 | 用户编写服务器程序时，创建 `TcpServer` 实例时调用 | 1. 绑定主线程 `loop_`、监听地址 `listenAddr`、服务器名称 `nameArg`；2. 初始化 `ipPort_`（由 `listenAddr` 转换为字符串）、`name_`；3. 创建 `Acceptor` 实例，绑定监听地址和端口重用选项（`option`）；4. 设置 `Acceptor` 的新连接回调为 `TcpServer::newConnection`；5. 初始化 `threadPool_`（绑定 `loop_` 和 `name_`）；6. 初始化 `started_ = 0`、`nextConnId_ = 1`、`numThreads_ = 0`；7. 初始化各类回调函数为空。 |
| `~TcpServer()`                                               | 释放服务器资源       | `TcpServer` 实例析构时调用（如服务器退出）        | 1. 无需手动释放 `loop_`（由用户管理生命周期）；2. `acceptor_`/`threadPool_` 由 `unique_ptr`/`shared_ptr` 自动析构，关闭监听 fd、停止线程池；3. `connections_` 中的 `TcpConnection` 由 `shared_ptr` 管理，析构时自动清理；4. 依赖 RAII 保证资源无泄漏。 |

> #### 核心设计：
>
> - **noncopyable 继承**：隐含继承 `noncopyable`（muduo 核心组件通用设计），禁止拷贝，确保服务器实例唯一；
> - **端口重用控制**：通过 `Option` 枚举控制 `listenfd` 的 `SO_REUSEPORT` 选项，适配多进程 / 多线程监听同一端口的场景；
> - **回调绑定**：构造时将 `Acceptor` 的新连接回调绑定到 `TcpServer::newConnection`，实现 “监听 - 接收连接” 的自动联动。

#### 2. 配置类接口（设置服务器参数）

| 函数名                                                      | 作用                            | 调用时机                                                     | 内部逻辑                                                     |
| ----------------------------------------------------------- | ------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `setThreadInitCallback(const ThreadInitCallback &cb)`       | 设置子线程 `subloop` 初始化回调 | 用户需要初始化子线程（如设置日志、加载配置）时调用           | 赋值 `threadInitCallback_ = cb`，并传递给 `threadPool_`      |
| `setConnectionCallback(const ConnectionCallback &cb)`       | 设置连接建立 / 断开回调         | 用户编写业务逻辑时，注册连接生命周期回调                     | 赋值 `connectionCallback_ = cb`，后续透传给 `TcpConnection`  |
| `setMessageCallback(const MessageCallback &cb)`             | 设置数据读取回调                | 用户编写业务逻辑时，注册数据处理回调                         | 赋值 `messageCallback_ = cb`，后续透传给 `TcpConnection`     |
| `setWriteCompleteCallback(const WriteCompleteCallback &cb)` | 设置数据发送完成回调            | 用户需要感知数据发送完成（如流量控制）时调用                 | 赋值 `writeCompleteCallback_ = cb`，后续透传给 `TcpConnection` |
| `setThreadNum(int numThreads)`                              | 设置线程池中子线程数量          | 用户初始化服务器时，配置 Reactor 子线程数（如 `setThreadNum(4)`） | 赋值 `numThreads_ = numThreads`；`start()` 时传递给 `threadPool_->setThreadNum()` |

> #### 配置生效规则：
>
> - **所有回调和线程数配置需在 `start()` 前调用，否则可能不生效；**
> - 回调函数采用 “覆盖式赋值”，多次调用以最后一次为准；
> - 线程数设置为 0 时，默认单线程模式（所有逻辑在 `loop_` 执行）。

#### 3. 服务器启动 / 核心逻辑

| 函数名                                                   | 作用                                | 调用时机                                                     | 内部逻辑 / 触发后续操作                                      |
| -------------------------------------------------------- | ----------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `start()`                                                | 启动服务器（监听端口 + 启动线程池） | 用户初始化配置后，启动服务器时调用                           | 1. 原子校验 `started_`，确保仅启动一次（线程安全）；2. 启动线程池 `threadPool_->start(threadInitCallback_)`，创建子线程和 `subloop`；3. 调用 `acceptor_->listen()`，启动 `listenfd` 的监听（`listen()` 系统调用）；4. 将 `started_` 置为 1；5. 所有操作确保在 `loop_` 线程执行（线程安全）。 |
| `newConnection(int sockfd, const InetAddress &peerAddr)` | 处理新连接（`Acceptor` 的回调函数） | `Acceptor` 检测到新连接，调用 `accept()` 拿到 `sockfd` 后触发 | 1. 从 `threadPool_` 获取一个 `subloop`（单线程时为 `loop_`）；2. 生成新连接的唯一名称（`name_ + "-connection-" + nextConnId_`），`nextConnId_` 自增；3. 创建 `TcpConnection` 实例，传入 `subloop`、连接名称、`sockfd`、本地地址（`listenAddr`）、对端地址（`peerAddr`）；4. 将 `TcpServer` 的回调（`connectionCallback_`/`messageCallback_` 等）设置到 `TcpConnection`；5. 设置 `TcpConnection` 的 `closeCallback` 为 `TcpServer::removeConnection`；6. 调用 `TcpConnection::connectEstablished()`，标记连接建立并注册事件到 `subloop`；7. 将新连接加入 `connections_` 容器管理。 |

> #### 新连接分配规则：
>
> - 线程池采用 “轮询 / 哈希” 策略（muduo 默认轮询）分配 `subloop`，均衡各子线程的连接数；
> - `TcpConnection` 的生命周期由 `shared_ptr` 管理，加入 `connections_` 后引用计数至少为 1，避免提前析构。

#### 4. 连接清理逻辑

| 函数名                                                 | 作用                                | 调用时机                                    | 内部逻辑 / 触发后续操作                                      |
| ------------------------------------------------------ | ----------------------------------- | ------------------------------------------- | ------------------------------------------------------------ |
| `removeConnection(const TcpConnectionPtr &conn)`       | 移除连接（跨线程安全接口）          | `TcpConnection` 触发 `closeCallback` 时调用 | 1. 调用 `loop_->runInLoop()`，将 `removeConnectionInLoop` 投递到 `loop_` 线程执行（避免跨线程修改 `connections_`）；2. 线程安全保证：无论调用线程是 `subloop` 还是其他线程，最终操作在 `baseloop` 执行。 |
| `removeConnectionInLoop(const TcpConnectionPtr &conn)` | 实际移除连接（仅 `loop_` 线程执行） | `removeConnection` 投递任务后触发           | 1. 从 `connections_` 容器中移除该连接（根据 `conn->name()` 查找）；2. 调用 `TcpConnection::connectDestroyed()`，清理连接的事件监听和状态；3. 释放 `TcpConnection` 的引用（`connections_` 移除后，引用计数减 1，无其他引用时析构）；4. 无需手动关闭 `sockfd`（`TcpConnection` 析构时由 `socket_` 自动关闭）。 |

> #### 连接清理核心规则：
>
> - `connections_` 容器的修改仅在 `loop_` 线程执行，避免多线程竞争；
> - `removeConnectionInLoop` 是连接销毁的 “最后一步”，确保 `TcpConnection` 资源被正确清理；
> - 清理顺序：移除容器引用 → 清理连接事件 → 释放资源，避免 `Poller` 持有无效连接的事件。

### 四、补充说明

#### 1. 核心设计逻辑

- **分层抽象**：`TcpServer` 作为顶层抽象，屏蔽底层 `Acceptor`（监听）、`EventLoopThreadPool`（线程管理）、`TcpConnection`（连接处理）的细节，用户仅需关注 “配置回调 + 启动服务器”；
- **线程模型解耦**：主线程（`loop_`）负责监听，子线程（`subloop`）负责处理连接，通过线程池实现 “监听 - 处理” 解耦，提升并发能力；
- **线程安全**：核心状态（`started_`）用原子类型，关键操作（`start()`、`removeConnection()`）通过 `loop_->runInLoop()` 保证在 `loop_` 线程执行，避免多线程竞争；
- **生命周期管理**：`TcpConnection` 的生命周期由 `shared_ptr` 管理，`TcpServer` 通过 `connections_` 容器持有引用，确保连接未处理完时不被析构。

#### 2. 异常处理细节

- **重复启动防护**：`start()` 中通过 `started_` 原子校验，避免多线程重复调用 `listen()` 导致的错误；
- **连接清理安全**：`removeConnection` 跨线程调用时，通过 `runInLoop()` 投递到 `loop_` 线程，避免 `connections_` 容器的并发修改；
- **资源泄漏防护**：`Acceptor` 的 `listenfd`、`TcpConnection` 的 `connfd` 均由 RAII 封装，析构时自动关闭，避免文件描述符泄漏；
- **线程池异常**：`threadPool_->start()` 若创建线程失败，会直接终止进程（muduo 设计），避免半启动状态导致的逻辑异常。

#### 3. 与其他组件的协作

| 组件                  | 协作方式                                                     |
| --------------------- | ------------------------------------------------------------ |
| `EventLoop`           | `loop_` 作为主线程 `baseloop`，管理 `Acceptor` 的事件；`threadPool_` 管理子线程 `subloop`，分配给 `TcpConnection`；`runInLoop()` 保证跨线程操作的线程安全 |
| `Acceptor`            | `TcpServer` 持有 `Acceptor` 实例，设置其新连接回调；`Acceptor` 负责监听 `listenfd`，触发 `newConnection()` 传递 `sockfd` |
| `EventLoopThreadPool` | `TcpServer` 持有线程池实例，`start()` 时启动线程池；`newConnection()` 从线程池获取 `subloop` 分配给新连接；线程池管理子线程的生命周期 |
| `TcpConnection`       | `TcpServer` 创建 `TcpConnection` 并传递回调；`TcpConnection` 触发 `closeCallback` 通知 `TcpServer` 清理连接；`TcpServer` 通过 `connections_` 管理所有连接 |
| `InetAddress`         | 封装监听地址和对端地址，为 `TcpServer`/`TcpConnection` 提供地址解析、转换能力 |

#### 4. 性能优化点

- **连接分配均衡**：线程池采用轮询策略分配 `subloop`，避免单个子线程处理过多连接导致的负载不均；
- **减少锁竞争**：`connections_` 仅在 `loop_` 线程修改，无需加锁；原子类型 `started_` 避免启动逻辑的锁开销；
- **延迟初始化**：线程池在 `start()` 时才启动，避免构造时的无效开销；
- **端口重用**：支持 `SO_REUSEPORT` 选项，适配多进程监听同一端口的场景（多核优化）；
- **RAII 资源管理**：`Acceptor`/`EventLoopThreadPool` 用智能指针管理，避免手动释放资源导致的泄漏。

### 五、典型业务流程示例

```plaintext
1. 服务器初始化与启动：
用户 → 构造 TcpServer（传入 loop_、监听地址、服务器名）→ 设置回调（connection/message）→ 设置线程数 → 调用 start() → 启动线程池 → Acceptor::listen() 监听端口

2. 接收新连接：
Poller（loop_）检测到 listenfd 可读 → Acceptor::handleRead() → 调用 accept() 获取 sockfd → TcpServer::newConnection() → 从线程池获取 subloop → 创建 TcpConnection → 设置回调 → TcpConnection::connectEstablished() → 加入 connections_ 管理

3. 处理连接数据：
Poller（subloop）检测到 connfd 可读 → TcpConnection::handleRead() → 触发 messageCallback_ → 执行业务逻辑 → 调用 TcpConnection::send() 发送响应

4. 连接断开清理：
TcpConnection 检测到连接关闭 → 触发 closeCallback_ → TcpServer::removeConnection() → 投递到 loop_ 线程 → removeConnectionInLoop() → 从 connections_ 移除 → TcpConnection::connectDestroyed() → 释放连接资源
```

### 六、关键注意事项

1. **线程安全**：`start()` 是线程安全的，可多线程调用，但仅第一次生效；`setThreadNum()`/ 回调设置非线程安全，需在 `start()` 前单线程调用；
2. **生命周期管理**：`loop_` 由用户管理，需保证 `TcpServer` 析构前 `loop_` 未销毁；`TcpConnection` 的生命周期由 `shared_ptr` 自动管理，用户无需手动释放；
3. **端口占用**：启动前需确保监听端口未被占用，否则 `acceptor_->listen()` 会失败；开启 `kReusePort` 可缓解端口占用问题，但需操作系统支持；
4. **回调执行线程**：`connectionCallback_`/`messageCallback_` 等在 `subloop` 线程执行，用户回调需保证线程安全（或仅在回调内操作连接局部数据）；
5. **优雅退出**：服务器退出时，需先停止 `loop_` 的事件循环，再析构 `TcpServer`，避免 `connections_` 中存在未清理的连接导致资源泄漏。

### 补充： 所有回调函数的整体流程

```cpp
1. 服务器启动阶段
   TcpServer::start() → Acceptor::listen() → 注册监听socket的读事件到MainLoop

2. 新连接建立阶段
   客户端连接 → MainLoop触发监听socket读事件 → Acceptor::handleRead() → TcpServer::newConnection()
   → 创建TcpConnection → 分配SubLoop → TcpConnection::connectEstablished() → 触发ConnectionCallback（kConnected）

3. 消息收发阶段
   客户端发数据 → SubLoop触发连接socket读事件 → TcpConnection::handleRead() → 触发MessageCallback
   用户调用send() → 数据入输出缓冲区 → 注册写事件 → 内核发送数据 → SubLoop触发写事件 → TcpConnection::handleWrite() → 缓冲区清空 → 触发WriteCompleteCallback

4. 连接断开阶段
   客户端断开 → SubLoop触发连接socket读事件（读字节数为0） → TcpConnection::handleClose()
   → 触发ConnectionCallback（kDisconnected） → TcpServer::removeConnection() → 清理连接资源
```

一个是操作系统内部的回调函数自动触发，触发的是用户设置的回调函数