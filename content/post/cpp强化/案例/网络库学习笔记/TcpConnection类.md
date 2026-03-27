# TcpConnection 类 成员变量 & 成员函数 作用总结

### 一、核心设计背景

`TcpConnection` 是高性能网络库中**连接层核心组件**，承接 `TcpServer/Acceptor` 建立的 TCP 连接，封装连接的生命周期管理、数据读写、事件回调等核心逻辑，遵循 **One Loop Per Thread + Reactor** 模型：

- 单 Reactor 模式下：`loop_` 指向主线程的 `baseloop`；
- 多 Reactor 模式下：`loop_` 指向子线程的 `subloop`（由 `TcpServer` 的线程池分配）；
- 核心协作链路：`TcpServer => Acceptor` 接收新连接 → 创建 `TcpConnection` → 绑定 `Channel` 到 `Poller` → 监听读写事件 → 触发回调处理业务。
- TcpConnection 就是 muduo 里对一条 “已建立的 TCP 连接” 的完整封装，负责管理这条连接从建立到销毁的全过程、收发数据、事件回调，是网络库和用户业务之间的桥梁。

### 二、成员变量

#### 1. 连接状态与核心上下文

| 变量名     | 作用                                                         | 调用 / 生效时机                                              |
| ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `loop_`    | 绑定当前连接的事件循环（`EventLoop*`），决定连接的事件处理线程 | 构造函数初始化；所有事件回调（`handleRead/Write/Close/Error`）、数据发送（`sendInLoop`）、连接销毁（`connectDestroyed`）时，确保操作在 `loop_` 线程执行 |
| `name_`    | 连接唯一标识（如 `connection-1-1690000000`），用于日志 / 调试 | 构造函数初始化；日志输出、连接管理（如 `TcpServer` 的 `connections_` 容器索引）时使用 |
| `state_`   | 连接状态（原子类型 `std::atomic_int`），线程安全的状态标识   | 构造函数初始化为 `kDisconnected`；`connectEstablished()` 置为 `kConnected`；`shutdown()` 置为 `kDisconnecting`；`handleClose()` 置为 `kDisconnected`；`connected()` 接口查询状态 |
| `reading_` | 标记连接是否监听「读事件」（默认开启）                       | `handleRead()` 处理异常时关闭读事件；`connectEstablished()` 初始化时开启读事件 |

#### 2. 网络资源封装

| 变量名       | 作用                                                         | 调用 / 生效时机                                              |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `socket_`    | 封装 TCP 连接的文件描述符（`fd`），管理 `fd` 的生命周期（如关闭 `fd`） | 构造函数初始化（接管 `Acceptor` 传入的 `connfd`）；`shutdownInLoop()` 中调用 `socket_->shutdownWrite()` 关闭写端；析构时自动释放 `fd` |
| `channel_`   | 封装 `fd` 的事件监听（读 / 写 / 错误 / 关闭），绑定事件回调到 `Poller` | 构造函数初始化；`connectEstablished()` 中注册到 `loop_` 的 `Poller`；`handleRead/Write/Close/Error` 作为事件回调；`connectDestroyed()` 中从 `Poller` 移除 |
| `localAddr_` | 本地地址（IP + 端口），记录连接的服务端地址                  | 构造函数初始化；上层业务获取本地地址（如日志、统计）时调用 `localAddress()` |
| `peerAddr_`  | 对端地址（IP + 端口），记录客户端地址                        | 构造函数初始化；上层业务获取客户端地址（如权限校验、日志）时调用 `peerAddress()` |

#### 3. 回调函数（业务扩展点）

| 变量名                   | 作用                                                         | 调用 / 生效时机                                              |
| ------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `connectionCallback_`    | 连接建立 / 断开时的回调（由 `TcpServer` 传递，用户自定义业务逻辑） | `connectEstablished()`（连接建立）、`handleClose()`（连接断开）时触发 |
| `messageCallback_`       | 读取到数据时的回调（用户自定义数据解析 / 处理逻辑）          | `handleRead()` 中读取到数据后触发，传入 `inputBuffer_` 供上层读取 |
| `writeCompleteCallback_` | 数据写入 `fd` 完成后的回调（如批量发送数据时的流量控制）     | `handleWrite()` 中 `outputBuffer_` 数据全部发送完成后触发    |
| `highWaterMarkCallback_` | 写缓冲区超过高水位阈值时的回调（流量控制 / 背压处理）        | `sendInLoop()` 中 `outputBuffer_` 可写字节数超过 `highWaterMark_` 时触发 |
| `closeCallback_`         | 连接关闭的最终回调（由 `TcpServer` 注册，用于清理连接资源）  | `handleClose()` 中触发，通知 `TcpServer` 移除当前连接        |
| `highWaterMark_`         | 高水位阈值（默认 0，用户通过 `setHighWaterMarkCallback` 设置） | `sendInLoop()` 中校验 `outputBuffer_` 长度时生效             |

#### 4. 数据缓冲区

| 变量名          | 作用                                                       | 调用 / 生效时机                                              |
| --------------- | ---------------------------------------------------------- | ------------------------------------------------------------ |
| `inputBuffer_`  | 读缓冲区，存储从 `fd` 读取的待处理数据                     | `handleRead()` 中调用 `readFd()` 读取数据到缓冲区；`messageCallback_` 中上层消费数据；`handleClose()` 前清空 |
| `outputBuffer_` | 写缓冲区，存储待发送到 `fd` 的数据（用户 `send` 接口写入） | `send()`/`sendInLoop()` 中写入数据；`handleWrite()` 中调用 `writeFd()` 发送数据；高水位校验基于该缓冲区长度 |

### 三、成员函数

#### 1. 构造 / 析构函数

| 函数名                                                       | 作用                    | 调用时机                                                     | 内部逻辑 / 核心设计                                          |
| ------------------------------------------------------------ | ----------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `TcpConnection(EventLoop *loop, const std::string &nameArg, int sockfd, const InetAddress &localAddr, const InetAddress &peerAddr)` | 初始化 TCP 连接核心资源 | `Acceptor` 接收新连接（`accept` 拿到 `connfd`）后，由 `TcpServer` 调用创建 | 1. 绑定 `loop_`、`name_`、`localAddr_`、`peerAddr_`；2. 封装 `sockfd` 为 `socket_`；3. 创建 `channel_` 并绑定 `fd`，注册 `handleRead/Write/Close/Error` 回调；4. 初始化 `state_ = kConnecting`、`reading_ = true`；5. 初始化缓冲区（`inputBuffer_`/`outputBuffer_`）、回调函数（默认空） |
| `~TcpConnection()`                                           | 释放连接资源            | 连接销毁（`connectDestroyed()`）后，`shared_ptr` 引用计数归 0 时调用 | 1. `socket_`/`channel_` 由 `unique_ptr` 自动析构，关闭 `fd`、移除 `Poller` 事件；2. 缓冲区（`inputBuffer_`/`outputBuffer_`）自动析构释放内存；3. 无额外手动资源管理（依赖 RAII） |

> #### 核心设计：
>
> - **RAII 封装**：`socket_`/`channel_` 用 `unique_ptr` 管理，确保析构时资源自动释放，避免内存泄漏 / 文件描述符泄漏；
>
> - **enable_shared_from_this**：继承 `std::enable_shared_from_this<TcpConnection>`，允许在回调中安全获取 `shared_ptr`（避免野指针），如 `handleRead()` 中用 `shared_from_this()` 延长对象生命周期；
>
>   > `std::enable_shared_from_this` 是 C++ 标准库提供的模板类，**允许对象自身安全地生成指向自己的 `shared_ptr`**，且这个 `shared_ptr` 会和外部已有的 `shared_ptr` 共享同一个引用计数。
>
> - **noncopyable**：继承 `noncopyable` 禁止拷贝，确保连接对象唯一（TCP 连接是一对一的，拷贝无意义且易导致资源冲突）。

#### 2. 基础查询接口（只读）

| 函数名                 | 作用                                | 调用时机                                                     | 内部逻辑                          |
| ---------------------- | ----------------------------------- | ------------------------------------------------------------ | --------------------------------- |
| `getLoop() const`      | 返回当前连接绑定的 `EventLoop` 指针 | 上层业务 / 回调中确保操作在 `loop_` 线程执行时调用（如 `sendInLoop()`） | 直接返回 `loop_`，无额外逻辑      |
| `name() const`         | 返回连接唯一标识 `name_`            | 日志输出、`TcpServer` 管理连接（如 `connections_` 容器）时调用 | 直接返回 `name_`，无额外逻辑      |
| `localAddress() const` | 返回本地地址 `localAddr_`           | 业务获取服务端地址（如日志、统计）时调用                     | 直接返回 `localAddr_`，无额外逻辑 |
| `peerAddress() const`  | 返回对端地址 `peerAddr_`            | 业务获取客户端地址（如权限校验、限流）时调用                 | 直接返回 `peerAddr_`，无额外逻辑  |
| `connected() const`    | 判断连接是否处于「已连接」状态      | 发送数据前校验连接状态（如 `send()`）、业务判断连接有效性时调用 | 直接返回 `state_ == kConnected`   |

#### 3. 回调函数注册（设置业务扩展点）

| 函数名                                                       | 作用                         | 调用时机                                                     | 内部逻辑                                                     |
| ------------------------------------------------------------ | ---------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `setConnectionCallback(const ConnectionCallback &cb)`        | 注册「连接建立 / 断开」回调  | `TcpServer` 初始化时，将用户注册的回调传递给 `TcpConnection` | 赋值 `connectionCallback_ = cb`                              |
| `setMessageCallback(const MessageCallback &cb)`              | 注册「读取到数据」回调       | 同上                                                         | 赋值 `messageCallback_ = cb`                                 |
| `setWriteCompleteCallback(const WriteCompleteCallback &cb)`  | 注册「数据发送完成」回调     | 同上                                                         | 赋值 `writeCompleteCallback_ = cb`                           |
| `setCloseCallback(const CloseCallback &cb)`                  | 注册「连接关闭」回调         | `TcpServer` 初始化时注册，用于清理连接资源                   | 赋值 `closeCallback_ = cb`                                   |
| `setHighWaterMarkCallback(const HighWaterMarkCallback &cb, size_t highWaterMark)` | 注册「高水位」回调并设置阈值 | 用户需要流量控制时调用（如限制写缓冲区大小）                 | 赋值 `highWaterMarkCallback_ = cb` + `highWaterMark_ = highWaterMark` |

> #### 回调传递链路：
>
> 用户 → `TcpServer` 注册回调 → `TcpConnection` 接收回调 → `Channel` 事件触发时执行回调 **handleXXX函数**（底层IO事件回调）→ 回到用户业务逻辑 **XXXcallback回调**

#### 4. 连接生命周期管理

| 函数名                 | 作用                                      | 调用时机                                    | 内部逻辑 / 触发后续操作                                      |
| ---------------------- | ----------------------------------------- | ------------------------------------------- | ------------------------------------------------------------ |
| `connectEstablished()` | 标记连接为「已建立」，注册事件到 `Poller` | `TcpServer` 创建 `TcpConnection` 后调用     | 1. 设置 `state_ = kConnected`；2. `channel_->enableReading()` 开启读事件；3. 将 `channel_` 注册到 `loop_->Poller`；4. 触发 `connectionCallback_`（通知上层连接建立）；5. 确保所有操作在 `loop_` 线程执行 |
| `connectDestroyed()`   | 标记连接为「已销毁」，清理事件监听        | `handleClose()` 触发后，由 `TcpServer` 调用 | 1. 校验 `state_ == kConnected`/`kDisconnecting`；2. `channel_->disableAll()` 禁用所有事件；3. `channel_->remove()` 从 `Poller` 移除；4. 设置 `state_ = kDisconnected`；5. 无需手动释放 `fd`（`socket_` 析构时自动关闭） |

#### 5. 数据发送（用户接口 + 内部实现）

| 函数名                                                       | 作用                                          | 调用时机                                 | 内部逻辑 / 触发后续操作                                      |
| ------------------------------------------------------------ | --------------------------------------------- | ---------------------------------------- | ------------------------------------------------------------ |
| `send(const std::string &buf)`                               | 上层用户发送数据的接口（线程安全）            | 业务层发送数据时调用（如响应客户端请求） | 1. 校验 `state_ == kConnected`；2. 调用 `loop_->runInLoop()`，将 `sendInLoop` 投递到 `loop_` 线程执行（避免跨线程操作缓冲区）；3. 若不在 `loop_` 线程，`runInLoop` 会将任务加入队列，由 `loop_` 线程执行 |
| `sendFile(int fileDescriptor, off_t offset, size_t count)`   | 上层用户发送文件的接口（线程安全）            | 业务层发送大文件时调用（如文件下载）     | 同 `send()` 逻辑，投递 `sendFileInLoop` 到 `loop_` 线程执行  |
| `sendInLoop(const void *data, size_t len)`                   | 实际发送数据的内部实现（仅 `loop_` 线程执行） | `send()` 投递任务后调用                  | 1. 若 `state_ != kConnected`，直接返回；2. 若 `channel_->isWriting() == false` + `outputBuffer_` 为空，尝试直接调用 `::write` 发送数据；3. 若直接发送失败 / 部分发送，将剩余数据写入 `outputBuffer_`；4. 若 `outputBuffer_` 长度超过 `highWaterMark_`，触发 `highWaterMarkCallback_`；5. `channel_->enableWriting()` 开启写事件（等待 `Poller` 触发可写）；6. 确保数据写入 `outputBuffer_` 后，由 `handleWrite()` 完成发送 |
| `sendFileInLoop(int fileDescriptor, off_t offset, size_t count)` | 实际发送文件的内部实现（仅 `loop_` 线程执行） | `sendFile()` 投递任务后调用              | 1. 利用 `sendfile()` 系统调用（零拷贝）直接发送文件数据；2. 处理部分发送场景，记录发送进度；3. 发送完成后触发 `writeCompleteCallback_`；4. 异常时触发 `handleError()` |

> #### 核心设计：
>
> - **线程安全**：`send()`/`sendFile()` 是线程安全的，通过 `loop_->runInLoop()` 确保所有缓冲区操作在 `loop_` 线程执行；
> - **零拷贝优化**：`sendFileInLoop` 用 `sendfile()` 避免用户态 / 内核态数据拷贝，适合大文件传输；
> - **写事件按需开启**：仅当 `outputBuffer_` 有未发送数据时，才开启写事件，减少 `Poller` 轮询开销。
> - 两种数据发送区别：
>
> | 核心维度   | sendInLoop                             | sendFileInLoop                  |
> | ---------- | -------------------------------------- | ------------------------------- |
> | 拷贝次数   | 1 次用户态→内核态拷贝                  | 0 次用户态→内核态拷贝（零拷贝） |
> | 数据载体   | 应用层内存数据                         | 磁盘文件（文件描述符）          |
> | 缓冲区依赖 | 强依赖 `outputBuffer_`（用户态）       | 不依赖 `outputBuffer_`          |
> | 流控机制   | 高水位回调（`highWaterMarkCallback_`） | 无                              |
> | 性能       | 通用场景够用，大文件效率低             | 大文件传输效率极高              |
> | 底层调用   | `write`                                | `sendfile`                      |

#### 6. 连接关闭（半关闭 / 全关闭）

| 函数名             | 作用                                             | 调用时机                                 | 内部逻辑 / 触发后续操作                                      |
| ------------------ | ------------------------------------------------ | ---------------------------------------- | ------------------------------------------------------------ |
| `shutdown()`       | 上层用户关闭连接写端（半关闭）的接口（线程安全） | 业务层主动关闭连接时调用（如响应完成后） | 1. 校验 `state_ == kConnected`；2. 调用 `loop_->runInLoop()` 投递 `shutdownInLoop` 到 `loop_` 线程执行 |
| `shutdownInLoop()` | 实际关闭写端的内部实现（仅 `loop_` 线程执行）    | `shutdown()` 投递任务后调用              | 1. 设置 `state_ = kDisconnecting`；2. 若 `channel_->isWriting() == false`，调用 `socket_->shutdownWrite()` 关闭 TCP 写端；3. 若 `outputBuffer_` 还有数据，等待 `handleWrite()` 发送完成后再关闭；4. 关闭后由 `Poller` 检测到读端 EOF，触发 `handleClose()` |

> #### 半关闭设计：
>
> TCP 半关闭（`shutdownWrite`）保证：
>
> - 本端无法再写数据，但仍可读取对端数据；
> - 对端收到 FIN 包后，读取到 EOF，触发 `handleRead()` 后关闭连接；
> - <u>避免直接 `close(fd)` 导致未发送的 `outputBuffer_` 数据丢失。</u>

#### 7. 事件回调（Channel 绑定的核心逻辑）

| 函数名                              | 作用                          | 调用时机                                                     | 内部逻辑 / 触发后续操作                                      |
| ----------------------------------- | ----------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `handleRead(Timestamp receiveTime)` | 处理读事件（`fd` 可读）       | `Poller` 检测到 `fd` 可读时，由 `Channel` 回调触发           | 1. 调用 `inputBuffer_.readFd(channel_->fd(), &saveErrno)` 读取数据到 `inputBuffer_`；2. 若读取到数据，触发 `messageCallback_`（传入 `shared_from_this()`、`inputBuffer_`、`receiveTime`）；3. 若读取到 0 字节（对端关闭），触发 `handleClose()`；4. 若读取失败（非 EAGAIN），触发 `handleError()`；5. 确保所有操作线程安全（`loop_` 线程执行） |
| `handleWrite()`                     | 处理写事件（`fd` 可写）       | `Poller` 检测到 `fd` 可写时，由 `Channel` 回调触发           | 1. 调用 `outputBuffer_.writeFd(channel_->fd(), &saveErrno)` 发送 `outputBuffer_` 数据；2. 若数据全部发送完成：- `channel_->disableWriting()` 关闭写事件；- 触发 `writeCompleteCallback_`；- 若 `state_ == kDisconnecting`，调用 `shutdownInLoop()` 完成关闭；3. 若仅发送部分数据，等待下一次写事件触发；4. 异常时触发 `handleError()` |
| `handleClose()`                     | 处理连接关闭事件（`fd` 关闭） | `Poller` 检测到 `fd` 关闭 / 读端 EOF 时，由 `Channel` 回调触发 | 1. `channel_->disableAll()` 禁用所有事件；2. 触发 `connectionCallback_`（通知上层连接断开）；3. 触发 `closeCallback_`（通知 `TcpServer` 清理连接）；4. 最终由 `TcpServer` 调用 `connectDestroyed()` 完成销毁；5. 注意：`handleClose()` 不会直接析构 `TcpConnection`（由 `shared_ptr` 管理生命周期） |
| `handleError()`                     | 处理错误事件（`fd` 错误）     | `Poller` 检测到 `fd` 错误时，由 `Channel` 回调触发           | 1. 调用 `getsockopt(channel_->fd(), SOL_SOCKET, SO_ERROR, &optval, &optlen)` 获取错误码；2. 日志输出错误码、连接信息（`name_`、`peerAddr_`）；3. 上层可通过日志 / 监控感知错误，无默认业务逻辑（由用户自定义） |

> #### 事件回调核心规则：
>
> - 所有事件回调仅在 `loop_` 线程执行，避免线程安全问题；
> - 回调触发顺序：`handleRead`（数据 / EOF）→ `handleClose` → `connectDestroyed`；
> - 错误处理：`handleRead/Write` 中检测到错误时，优先触发 `handleError()`，再触发 `handleClose()`❓

### 四、补充说明

#### 1. 核心设计逻辑

- **Reactor 模型适配**：`TcpConnection` 是 Reactor 模型的「事件处理对象」，`Channel` 是「事件分发器」，`Poller` 是「事件检测器」，三者协作完成 “事件检测 - 分发 - 处理”；
- **线程模型隔离**：多 Reactor 模式下，每个 `TcpConnection` 绑定独立的 `subloop`，实现连接级别的线程隔离，避免锁竞争；
- **缓冲区分层**：`inputBuffer_`（读）/`outputBuffer_`（写）分离，适配 TCP 流的 “读 / 写异步” 特性，避免数据交叉；
- **原子状态管理**：`state_` 用 `std::atomic_int` 保证多线程下状态读写的原子性（如 `shutdown()` 跨线程修改状态）。

#### 2. 异常处理细节

- **非阻塞 IO 适配**：`readFd`/`writeFd` 处理 `EAGAIN`/`EWOULDBLOCK`（非阻塞 IO 正常中断），仅返回已读 / 写字节数，不视为错误；
- **资源泄漏防护**：`connectDestroyed()` 确保 `Channel` 从 `Poller` 移除，避免 `Poller` 持有无效 `fd` 导致的资源泄漏；
- **生命周期安全**：依赖 `shared_from_this()` 避免回调中 `TcpConnection` 被提前析构（如 `handleRead` 中业务逻辑耗时较长时）。

#### 3. 与其他组件的协作

| 组件        | 协作方式                                                     |
| ----------- | ------------------------------------------------------------ |
| `TcpServer` | `TcpServer` 创建 `TcpConnection`，传递回调函数，管理连接生命周期（`connections_` 容器）；`TcpConnection` 触发 `closeCallback_` 通知 `TcpServer` 清理连接 |
| `Acceptor`  | `Acceptor` 接收新连接（`accept`）后，将 `connfd` 传递给 `TcpServer`，由 `TcpServer` 创建 `TcpConnection` |
| `EventLoop` | `TcpConnection` 绑定 `loop_`，所有事件处理、数据读写均在 `loop_` 线程执行；`loop_->runInLoop()` 保证跨线程操作的线程安全 |
| `Channel`   | `Channel` 封装 `fd` 事件，绑定 `TcpConnection` 的 `handleRead/Write/Close/Error` 回调，由 `Poller` 触发 |
| `Buffer`    | `inputBuffer_` 接收网络数据，`outputBuffer_` 缓存待发送数据；`readFd`/`writeFd` 完成内核态 / 用户态数据交互 |

#### 4. 性能优化点

- **减少系统调用**：`sendInLoop` 优先直接写 `fd`，仅当数据未发送完成时才写入缓冲区，减少 `write` 系统调用次数；
- **避免内存拷贝**：`sendFileInLoop` 用 `sendfile()` 实现零拷贝，大文件传输效率提升显著；
- **事件按需监听**：写事件仅在 `outputBuffer_` 有数据时开启，读事件默认开启（可关闭），减少 `Poller` 轮询开销；
- **RAII 资源管理**：`socket_`/`channel_` 用 `unique_ptr` 自动释放，避免手动管理资源导致的泄漏。

### 五、典型业务流程示例

```plaintext
1. 新连接建立：
Acceptor::handleRead() → TcpServer::newConnection() → TcpConnection 构造 → connectEstablished() → 触发 connectionCallback_（连接建立）

2. 接收数据：
Poller 检测到读事件 → Channel::handleEvent() → TcpConnection::handleRead() → inputBuffer_.readFd() → 触发 messageCallback_（业务处理数据）

3. 发送数据：
用户调用 TcpConnection::send() → sendInLoop() → 直接写 fd / 写入 outputBuffer_ → 开启写事件 → Poller 检测到写事件 → handleWrite() → outputBuffer_.writeFd() → 触发 writeCompleteCallback_

4. 关闭连接：
用户调用 TcpConnection::shutdown() → shutdownInLoop() → 关闭写端 → 对端发送 FIN → handleRead() 检测到 EOF → handleClose() → 触发 connectionCallback_（连接断开）→ closeCallback_ → TcpServer::removeConnection() → connectDestroyed() → TcpConnection 析构
```

