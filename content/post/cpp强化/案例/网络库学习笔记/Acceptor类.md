# Acceptor 类 成员变量 & 成员函数 作用总结

### 一、成员变量

#### 1. 核心循环与功能标识相关

| 变量名        | 作用                                                         | 调用 / 生效时机                                              |
| ------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `loop_`       | 指向 Acceptor 绑定的 EventLoop（即用户创建的 mainLoop/baseLoop），Acceptor 所有事件处理（如监听连接、处理新连接）均在该 loop 所属线程执行 | 构造函数初始化，生命周期内固定绑定；`handleRead()`中处理新连接时，依赖该 loop 完成回调派发等操作 |
| `listenning_` | 标识 Acceptor 是否处于 “监听端口” 状态                       | 调用`listen()`成功后置为`true`；析构时（或主动停止监听时）置为`false`；`listenning()`方法返回该值供外部校验监听状态 |

#### 2. 网络与事件处理相关

| 变量名                   | 作用                                                         | 调用 / 生效时机                                              |
| ------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `acceptSocket_`          | 专用于 “监听并接受新 TCP 连接” 的 Socket 对象，封装了监听套接字的创建、端口绑定、listen 系统调用等底层操作 | 构造函数中根据传入的`listenAddr`创建并初始化；`listen()`中调用其`listen()`方法启动端口监听；`handleRead()`中调用其`accept4()`方法接受新连接 |
| `acceptChannel_`         | 绑定`acceptSocket_`的 Channel 对象，用于监听 “套接字可读事件”（即有新连接到来），并将事件回调关联到`handleRead()` | 构造函数初始化：将`acceptSocket_`的文件描述符、`loop_`传入 Channel；`listen()`中设置 Channel 关注 “读事件” 并注册到 EventLoop；`handleRead()`是 Channel 的读事件回调函数；析构时 Channel 自动从 EventLoop 注销 |
| `NewConnectionCallback_` | 新连接建立后的回调函数，参数为 “新连接的套接字描述符” 和 “客户端地址”，由外部（如 TcpServer）设置，用于将新连接分发到 subLoop 处理 | 外部调用`setNewConnectionCallback()`设置；`handleRead()`中成功接受新连接后，执行该回调函数 |

### 二、成员函数

#### 1. 构造 / 析构函数

| 函数名                                                       | 作用                                                         | 调用时机                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `Acceptor(EventLoop *loop, const InetAddress &listenAddr, bool reuseport)` | 初始化 Acceptor 核心资源：1. 绑定所属的 EventLoop（`loop_`）；2. 创建`acceptSocket_`：基于`listenAddr`（监听地址 + 端口）创建套接字，开启`SO_REUSEADDR`/`SO_REUSEPORT`（根据`reuseport`参数），完成地址绑定；3. 创建`acceptChannel_`：绑定`acceptSocket_`的 fd 和`loop_`，并设置 Channel 的读事件回调为`handleRead()`；4. 初始化`listenning_`为`false`，`NewConnectionCallback_`为空 | 创建 Acceptor 实例时调用（通常在 mainLoop 所在线程初始化，与 TcpServer 绑定） |
| `~Acceptor()`                                                | 释放 Acceptor 资源：1. `acceptChannel_`析构：自动从`loop_`中注销事件监听，关闭 Channel 关联的 fd 事件；2. `acceptSocket_`析构：自动关闭监听套接字；3. 置`listenning_`为`false` | Acceptor 实例生命周期结束时调用（如 TcpServer 析构时）       |

> #### 关键初始化细节：
>
> - `SO_REUSEPORT`：多进程 / 线程监听同一端口时，内核会将新连接均匀分发到不同监听套接字，提升并发接受连接的性能；
> - Channel 回调绑定：构造时就将`handleRead()`设为读事件回调，保证新连接到来时能触发处理逻辑；
> - 套接字绑定：构造时已完成`bind()`，`listen()`仅执行`listen()`系统调用，减少`listen()`调用的耗时。

> #### 构造辅助函数：`static int createNonblocking()`
>
> 函数整体作用：
>
> - **`static`**：这个函数**只在当前 .cpp 文件内可见**，外部不能调用，是内部工具函数。
> - **功能**：创建一个**非阻塞、自动关闭、TCP 协议**的套接字（socket），并返回它的文件描述符 `sockfd`。
>
> ```cpp
> int sockfd = ::socket(AF_INET, SOCK_STREAM | SOCK_NONBLOCK | SOCK_CLOEXEC, IPPROTO_TCP); //核心代码
> ```

> ####  构造函数与后续代码的衔接
>
> 当用户调用 `TcpServer->start()` 时，最终会调用 `Acceptor->listen()`：
>
> - `listen()` 会调用 `acceptSocket_.listen()`（系统调用 `listen`）。
> - 然后调用 `acceptChannel_.enableReading()`，把 listenfd 真正注册到 Epoll Poller 中开始监听。
>
> 一旦有新连接进入，Epoll 检测到读事件，就会触发 `acceptChannel_` 的回调，也就是刚才注册的 **`Acceptor::handleRead`** 函数。

> ### 为什么构造函数不开启读事件监听？
>
> 构造函数的职责是 “初始化资源”，而不是 “启动服务”—— 这是**面向对象设计的 “开闭原则”**，把 “创建” 和 “启动” 分离，让代码更灵活。
>
> 1. 构造函数只**注册回调、初始化资源**，但不开启事件监听，保证 “创建” 和 “启动” 分离；
> 2. 真正开启 listenfd 读事件监听的是 `Acceptor::listen()` 中的 `enableReading()`，这是 Reactor 模式 “先注册回调，后启用事件” 的标准流程。

---

> #### 析构函数隐含操作
>
> * `acceptChannel_`是`Acceptor`的成员（栈上对象），`Acceptor`析构时`acceptChannel_`会自动析构，但 **`Channel`的析构函数本身不会主动调用`disableAll()`/`remove()`**（`Channel`的设计是 “被动管理”，需上层主动调用接口）。
>   * 但`Acceptor`析构函数中已经主动调用了`disableAll()`+`remove()`，等价于 “从`loop_`注销事件监听、关闭 fd 事件”
> * `acceptSocket_`是`Acceptor`的成员（栈上对象），`Acceptor`析构时会触发`acceptSocket_`的析构；
>   * `acceptSocket_`封装了监听套接字（`listenfd`），其析构函数会调用`::close(fd)`关闭该套接字 
> * （额外：`Channel`的析构不会主动关闭 fd，fd 的关闭由`acceptSocket_`的析构负责，这是分层设计的合理分工 ——`Channel`管 “事件监听”，`Socket`管 “fd 生命周期”）。

#### 2. 回调设置与状态查询

| 函数名                                                      | 作用                                                         | 调用时机                                                     | 内部逻辑 / 触发后续操作                                      |
| ----------------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `setNewConnectionCallback(const NewConnectionCallback &cb)` | 设置新连接的回调函数，供外部（如 TcpServer）注入 “新连接处理逻辑” | TcpServer 初始化 Acceptor 后调用，**通常在启动监听前完成设置**💥 | 直接将`NewConnectionCallback_`赋值为传入的`cb`；无其他逻辑，仅为`handleRead()`提供回调入口 |
| `listenning() const`                                        | 返回 Acceptor 当前是否处于监听状态                           | 外部需要校验监听端口是否正常启动时调用（如 TcpServer 启动后自检、日志打印） | 直接返回`listenning_`的值，无其他逻辑                        |

#### 3. 核心功能：启动监听

| 函数名     | 作用                                               | 调用时机                                                     | 内部逻辑 / 触发后续操作                                      |
| ---------- | -------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `listen()` | 启动端口监听，使 Acceptor 进入 “可接受新连接” 状态 | TcpServer 启动时调用（mainLoop 线程执行），**通常在设置完`NewConnectionCallback_`后** | 1. 调用`acceptSocket_.listen()`执行底层`listen()`系统调用，开启套接字监听；2. 将`acceptChannel_`设置为 “关注读事件”（`enableReading()`）；3. `acceptChannel_`注册到`loop_`的 Poller 中，等待新连接事件（`Channel::update()`）；4. 置`listenning_`为`true`；⚠️ 若`listen()`系统调用失败，通常会触发断言 / 日志，保证监听启动失败时能快速发现 |

> ​	先调用 `socket->listen()`让套接字进入监听状态以准备接收客户端连接，再调用 `channel->enableReading()` 将监听套接字的读事件注册到 Poller 中，是为了确保 Poller 监测到的读事件是有效的客户端连接请求，避免因先注册事件但套接字未监听导致 Poller 误触发无意义的事件（空监控 / 误触发，accept 可能返回错误）。

#### 4. 内部事件处理：处理新连接

| 函数名         | 作用（私有成员函数，仅内部调用）                             | 调用时机                                                     | 内部逻辑 / 触发后续操作                                      |
| -------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `handleRead()` | 处理 “监听套接字可读” 事件（即有新连接到来），核心逻辑是接受新连接并触发回调 | 当有新连接到来时，`loop_`的 Poller 检测到`acceptSocket_`的读事件，调用该回调（Channel 绑定的读事件处理函数） | 1. 调用`acceptSocket_.accept4()`接受新连接：- 若接受成功：返回新连接的`sockfd`和客户端`InetAddress`；- 若接受失败（如系统资源不足、中断）：记录日志并返回（不退出，保证后续连接仍能处理）；2. 若设置了`NewConnectionCallback_`，执行该回调：<!--将新连接的 sockfd 和客户端地址传入上层，交由外部（如 TcpServer）分发到 subLoop 处理-->；⚠️ `accept4()`通常会设置 `SOCK_NONBLOC | KSOCK_CLOEXEC`，保证新连接的套接字是非阻塞的，且 exec 时自动关闭 |

> * 新连接事件回调函数：接收新连接的fd和地址信息
>   * 把 accept 得到的**新连接 fd**和**客户端地址**，交给上层去做后续处理。（具体操作见下图）
> * 读事件回调函数：无参无返回值
>   * 当poller监听到acceptSocket_的可读事件（说明是新连接），接收新连接并触发新连接事件回调

> ```cpp
> 客户端发起连接
>      ↓
> 内核完成三次握手
>      ↓
> listenfd 可读（epoll 通知）
>      ↓
> Channel 调用 Acceptor::handleRead()
>      ↓
> accept() → 得到 connfd、peerAddr
>      ↓
> =====================================
> ★  这一行：NewConnectionCallback_(connfd, peerAddr);	// 新连接事件回调函数
> =====================================
>      ↓
> 上层 TcpServer 收到连接
>      ↓
> 1. 创建 TcpConnection
> 2. 选一个 subLoop（IO 线程）
> 3. 把 connfd 封装成 Channel 注册到 subLoop
> 4. 交给 subLoop 负责读写
> ```

### 补充说明

1. **核心设计逻辑**：
   - Acceptor 是 “主从 Reactor 模型” 中 mainLoop 的核心组件：仅负责 “监听端口 + 接受新连接”，不处理连接的 IO 读写；
   - 接受新连接后，通过`NewConnectionCallback_`将连接分发到 subLoop（EventLoopThreadPool 中的 IO 线程），实现 “主循环只做连接分发，子循环做 IO 处理” 的解耦。
2. **异常处理细节**：
   - `handleRead()`中`accept4()`失败时，不会退出或关闭监听：比如`EAGAIN`（无可用连接）、`ECONNABORTED`（连接被中断）等临时错误，仅日志记录；若为`EMFILE`（文件描述符耗尽）等致命错误，会触发告警并停止监听；
   - `acceptChannel_`仅关注 “读事件”：因为监听套接字只有 “可读” 事件（新连接到来），无需关注写事件。
3. **与其他组件的协作**：
   - Acceptor 依赖 EventLoop：所有事件注册、回调执行均在`loop_`所属线程（mainLoop 线程）完成；
   - Acceptor 与 TcpServer 联动：TcpServer 初始化 Acceptor，设置`NewConnectionCallback_`（回调逻辑为 “将新连接封装为 TcpConnection，分配到 subLoop”），调用`listen()`启动监听；
   - 与 Socket/Channel 解耦：Acceptor 仅封装 “监听 + 接受连接” 的业务逻辑，底层套接字操作由 Socket 封装，事件监听由 Channel 封装，符合 “单一职责” 设计原则。