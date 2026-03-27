### `InetAddress` 组件代码解析

该组件是基于 IPv4 网络协议的地址封装类，核心功能是管理网络地址（IP + 端口），提供字节序转换、地址格式转换（二进制↔字符串）等基础能力，适用于网络编程场景（如 Socket 通信）。

#### 1. 头文件依赖

```cpp
#pragma once
#include <arpa/inet.h>  // inet_addr, inet_ntop （网络地址转换）
#include <netinet/in.h> // sockaddr_in （IPv4地址结构体）
#include <string>       // std::string （字符串类型）
```

```cpp
#include <strings.h>   // 提供字符串操作（如 strlen）
#include <string.h>    // 提供内存操作（如 memset）
#include "InetAddress.h" // 类的声明头文件
```

依赖系统库完成内存 / 字符串操作，依赖自定义头文件完成类的声明。头文件引入顺序：系统库 → 自定义库，提升代码可读性。

> ### 补充：
>
> * 唯一成员变量：`sockaddr_in addr_`，这是封装的底层的IPV4的地址结构体

#### 2. 构造函数

```cpp
InetAddress::InetAddress(uint16_t port, std::string ip)
{
    ::memset(&addr_, 0, sizeof(addr_)); // 初始化地址结构体为0
    addr_.sin_family = AF_INET;         // 指定地址族为IPv4
    addr_.sin_port = ::htons(port);     // 端口：本地字节序 → 网络字节序
    addr_.sin_addr.s_addr = ::inet_addr(ip.c_str()); // IP字符串 → 网络字节序二进制
  	// 更好的做法是inet_pton，支持IPV6  ::inet_pton(AF_INET, ip.c_str(), &addr_.sin_addr);
}
```

- **功能**：初始化 `sockaddr_in` 类型的成员变量 `addr_`（IPv4 地址结构体）。
- **关键操作**：
  - `memset`：清空地址结构体，避免脏数据；
  - `AF_INET`：明确使用 IPv4 协议；
  - `htons`：将主机字节序（小端）的端口转为网络字节序（大端）；
  - `inet_addr`：将点分十进制 IP 字符串（如 "127.0.0.1"）转为网络字节序的 32 位整数。

> `c_str()`将`string`转为`const char*`，因为网络函数只支持c风格的字符串

#### 3. toIp () 方法

```cpp
std::string InetAddress::toIp() const
{
    char buf[64] = {0}; // 缓冲区存储IP字符串
    ::inet_ntop(AF_INET, &addr_.sin_addr, buf, sizeof buf); // 二进制→字符串
    return buf;
}
```

- **功能**：将二进制的 IPv4 地址转为点分十进制字符串。
- **关键函数**：`inet_ntop`（网络 to 表示）：
  - 入参：地址族（AF_INET）、二进制地址指针、输出缓冲区、缓冲区大小；
  - 出参：将二进制地址写入缓冲区，返回缓冲区首地址。

#### 4. toIpPort () 方法

```cpp
std::string InetAddress::toIpPort() const
{
    char buf[64] = {0};
    ::inet_ntop(AF_INET, &addr_.sin_addr, buf, sizeof buf); // 先写入IP字符串
    size_t end = ::strlen(buf); // 获取IP字符串长度
    uint16_t port = ::ntohs(addr_.sin_port); // 端口：网络字节序→本地字节序
    sprintf(buf+end, ":%u", port); // 拼接端口（格式：IP:Port）buf+end表示从IP地址字符串的末尾开始写入端口号字符串
    return buf;
}
```

- **功能**：返回 "IP: 端口" 格式的字符串（如 "127.0.0.1:8080"）。
- **关键操作**：
  - `strlen`：获取 IP 字符串长度，确定端口拼接的起始位置；
  - `ntohs`：将网络字节序的端口转回主机字节序；
  - `sprintf`：在 IP 字符串后拼接端口（如 ":8080"）。

#### 5. toPort () 方法

```cpp
uint16_t InetAddress::toPort() const
{
    return ::ntohs(addr_.sin_port); // 网络字节序→本地字节序
}
```

- **功能**：返回主机字节序的端口号。
- **核心逻辑**：仅封装 `ntohs` 操作，简化端口的字节序转换。

#### 补充两个函数

```cpp
// 这个函数的作用是返回一个指向 InetAddress 类的成员变量 addr_ 的常量指针，允许外部代码访问 InetAddress 对象所封装的 socket 地址信息，但不允许修改它。
const sockaddr_in *getSockAddr() const { return &addr_; }

// 这个函数的作用是将一个 sockaddr_in 结构体对象作为参数传入，并将其赋值给 InetAddress 类的成员变量 addr_，从而更新 InetAddress 对象所封装的 socket 地址信息。
void setSockAddr(const sockaddr_in &addr) { addr_ = addr; } 
```



#### 6. 测试代码（注释掉）

```cpp
#if 0
#include <iostream>
int main()
{
    InetAddress addr(8080); // 构造默认IP（0.0.0.0）+ 8080端口的地址
    std::cout << addr.toIpPort() << std::endl; // 输出：0.0.0.0:8080（其实是127.0.0.1，问题不大～）
}
#endif
```

- **作用**：示例代码，验证组件功能；通过 `#if 0` 注释，编译时不会生效。

- **测试逻辑**：构造端口为 8080 的地址（IP 未指定时 `inet_addr` 默认为 0 → 0.0.0.0），输出 "0.0.0.0:8080"。

- #### <!-- #if 预处理指令，条件编译--> 🔥🤫

### 核心设计思路

1. **字节序适配**：网络协议要求使用大端字节序，通过 `htons/ntohs` 完成主机↔网络字节序的转换；
2. **格式封装**：将 `sockaddr_in` 结构体封装为类，对外提供简洁的 IP / 端口操作接口；
3. **易用性**：提供 `toIp/toIpPort/toPort` 等方法，屏蔽底层二进制地址的操作细节。

### 适用场景

该组件是网络编程的基础模块，可用于 TCP/UDP Socket 编程中，快速构建 / 解析网络地址（如服务端绑定地址、客户端指定连接地址）。



# --------------------------------------

## Socket 类 成员变量 & 成员函数 作用总结

该 `Socket` 类是对操作系统底层文件描述符（socket fd）的面向对象封装，遵循**不可拷贝**设计原则（继承 `noncopyable`），核心目标是简化 TCP 套接字的常用操作，屏蔽底层系统调用的细节，提升代码的可维护性和安全性。

### 核心设计原则

> #### 1. 不可拷贝特性
>
> 继承 `noncopyable` 意味着该类的**拷贝构造函数和赋值运算符被禁用**，避免 socket 文件描述符被重复拷贝导致的：
>
> - 重复关闭同一个 fd（双重释放）；
> - 多个对象管理同一个 fd 引发的状态混乱。
>
> #### 2. 资源管理
>
> 通过**析构函数**自动释放 socket fd 资源，遵循 RAII（资源获取即初始化）原则，避免手动管理 fd 导致的内存 / 文件描述符泄漏。

### 一、成员变量

| 变量名    | 作用                                                         | 调用 / 生效时机                                              |
| --------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `sockfd_` | 封装的操作系统底层 socket 文件描述符（fd），是 Socket 类的核心标识 | 构造函数初始化（由外部传入已创建的 sockfd）；所有 socket 操作（bind/listen/accept 等）均基于该 fd 执行；析构函数会关闭该 fd |

> 补充：
>
> 1. `sockfd_` 被声明为 `const int`，意味着一旦初始化后不可修改，保证 Socket 实例与 fd 一一绑定，避免 fd 被意外替换；
> 2. 继承 `noncopyable` 类，禁止拷贝构造和赋值构造，防止同一个 fd 被多个 Socket 实例持有（避免重复关闭、操作冲突）。

### 二、成员函数

#### 1. 构造 / 析构函数

| 函数名                        | 作用                                                         | 调用时机                                                     |
| ----------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `explicit Socket(int sockfd)` | 初始化 Socket 实例：1. 将传入的 sockfd 赋值给成员变量`sockfd_`；2. 因`explicit`修饰，禁止隐式类型转换（避免意外将 int 转为 Socket 实例） | 创建 Socket 实例时调用（通常是 Acceptor 接受连接后、或创建监听 socket 时） |
| `~Socket()`                   | 释放 socket 资源：关闭`sockfd_`对应的文件描述符，释放操作系统分配的 socket 资源 | Socket 实例生命周期结束时调用（如连接关闭、监听 socket 销毁时） |

> 析构函数核心逻辑：调用`::close(sockfd_)`关闭 fd，操作系统会清理该 socket 对应的内核资源（如 TCP 连接状态、文件描述符表项等）。

#### 2. fd 获取（基础接口）

| 函数名           | 作用                          | 调用时机                                                     | 内部逻辑 / 触发后续操作 |
| ---------------- | ----------------------------- | ------------------------------------------------------------ | ----------------------- |
| `int fd() const` | 返回当前 Socket 封装的 sockfd | 外部需要直接操作 fd 时调用（如将 fd 注册到 EventLoop 的 Poller、设置 socket 选项时） | 直接返回`sockfd_`的值   |

> 典型场景：`Channel` 封装 fd 时，通过`Socket::fd()`获取 fd 并关联到 Channel 实例，实现 “事件 + fd” 的绑定。

#### 3. socket 核心操作（TCP 通信）

| 函数名                                           | 作用                                                     | 调用时机                                                     | 内部逻辑 / 触发后续操作                                      |
| ------------------------------------------------ | -------------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `void bindAddress(const InetAddress &localaddr)` | 将 socket 绑定到指定的本地地址（IP + 端口）              | 创建监听 socket 后、调用`listen()`前调用（服务器端）；或客户端绑定固定端口时调用 | 1. 调用`::bind()`系统调用，传入`sockfd_`和`localaddr`解析后的 sockaddr 结构体；2. 处理绑定失败的错误（如端口被占用、权限不足） |
| `void listen()`                                  | 将 socket 置为监听状态，开始接受客户端连接（仅服务器端） | `bindAddress()`之后、`accept()`之前调用（服务器端启动监听时） | 1. 调用`::listen()`系统调用，传入`sockfd_`和监听队列长度（如 SOMAXCONN）；2. 使 socket 从 “主动连接” 状态转为 “被动监听” 状态，内核开始维护连接请求队列 |
| `int accept(InetAddress *peeraddr)`              | 接受一个客户端连接，获取新的连接 fd                      | 服务器端监听到可读事件（有新连接）时调用（Acceptor 处理读事件时） | 1. 调用`::accept4()`系统调用（相比`accept()`可设置 SOCK_CLOEXEC 等标志），获取新连接的 fd；2. 将客户端的地址信息（IP + 端口）填充到`peeraddr`中；3. 返回新连接的 fd（外部会用该 fd 创建新的 Socket 实例） |
| `void shutdownWrite()`                           | 关闭 socket 的写端（TCP 半关闭）                         | 主动关闭连接时调用（如服务器端发送完数据后、告知客户端不再写数据） | 1. 调用`::shutdown()`系统调用，传入`SHUT_WR`参数；2. TCP 协议会发送 FIN 包，触发半关闭状态（对方仍可读，本方不可写） |

> 关键补充：
>
> 1. `accept4()`的优势：支持设置`SOCK_NONBLOCK | SOCK_CLOEXEC`，新连接 fd 默认为非阻塞且执行 exec 时自动关闭，避免 fd 泄漏；在Reactor模式下，必须要设置成非阻塞fd～
> 2. `shutdownWrite()` vs `close()`：`shutdownWrite()`仅关闭写端，保留读端；`close()`会关闭读写两端，且若有多个进程 / 线程持有该 fd，需所有引用关闭后才真正释放连接。
>    * `shutdown`是 “连接层面” 的精细化关闭，用于控制套接字的读写方向，不释放文件描述符；
>    * `close` 是 “描述符层面” 的释放操作，核心是回收文件描述符，仅在引用计数为 0 时才关闭连接；

#### 4. socket 选项设置（属性配置）

| 函数名                        | 作用                                            | 调用时机                                                     | 内部逻辑 / 触发后续操作                                      |
| ----------------------------- | ----------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `void setTcpNoDelay(bool on)` | 开启 / 关闭 TCP_NODELAY 选项（禁用 Nagle 算法） | 创建连接后、传输数据前调用（如低延迟场景：即时通讯、游戏）   | 1. 调用`::setsockopt()`设置 TCP 层选项；2. `on=true`时禁用 Nagle 算法（立即发送小数据包），`on=false`时启用（合并小数据包） |
| `void setReuseAddr(bool on)`  | 开启 / 关闭 SO_REUSEADDR 选项                   | 服务器启动、绑定端口前调用（避免端口释放后 TIME_WAIT 状态导致绑定失败） | 1. 调用`::setsockopt()`设置 socket 层选项；2. `on=true`时允许复用本地地址和端口（解决 TIME_WAIT 占用端口问题） |
| `void setReusePort(bool on)`  | 开启 / 关闭 SO_REUSEPORT 选项                   | 多进程 / 线程监听同一端口时调用（如多核服务器负载均衡）      | 1. 调用`::setsockopt()`设置 socket 层选项；2. `on=true`时允许多个 socket 绑定到同一端口（不同进程 / 线程），内核会将连接请求均衡分发到各 socket |
| `void setKeepAlive(bool on)`  | 开启 / 关闭 SO_KEEPALIVE 选项（TCP 保活机制）   | 长连接场景下调用（如客户端断线后，服务器能及时检测）         | 1. 调用`::setsockopt()`设置 socket 层选项；2. `on=true`时内核会定期发送保活探测包，检测对方是否在线；无响应则断开连接 |

> 所有 socket 选项均通过`::setsockopt()`系统调用实现，通用流程为：
>
> 1. 定义`int optval = on ? 1 : 0;`（1 开启，0 关闭）；
> 2. 调用`setsockopt(sockfd_, 协议层（SOL_SOCKET/TCP）, 选项名, &optval, sizeof(optval))`；
> 3. 处理设置失败的错误（如不支持该选项的系统）。

> ### 套接字选项总结表
>
> | 选项名称     | 类型         | 核心作用                                                     | 适用场景                                                     |
> | ------------ | ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
> | TCP_NODELAY  | TCP 层选项   | 禁用 Nagle 算法，允许小数据包立即发送（默认 Nagle 算法会合并小数据包减少传输） | 对实时性要求高的场景（如游戏、实时通信、高频小数据包交互），避免小数据包因 Nagle 算法延迟发送 |
> | SO_REUSEADDR | 套接字层选项 | 允许套接字强制绑定到已被其他套接字使用的端口（即使端口处于 TIME_WAIT 状态） | 服务器重启时，快速重新绑定到原有端口，避免 “地址已被使用” 错误 |
> | SO_REUSEPORT | 套接字层选项 | 允许同一主机上多个套接字绑定到相同端口号                     | 多线程 / 多进程服务器场景，实现连接的负载均衡（多个进程 / 线程同时监听同一端口） |
> | SO_KEEPALIVE | 套接字层选项 | 启用 TCP 保活机制，定期发送探测包检测对等方是否存活          | 长连接场景（如长连接服务器），及时检测无效连接（如客户端异常断连未通知服务端） |
>
> ### 补充说明
>
> 1. **类型区分**：
>    - `TCP_NODELAY` 属于 TCP 协议层（`IPPROTO_TCP`）选项，仅对 TCP 套接字有效；
>    - `SO_REUSEADDR`/`SO_REUSEPORT`/`SO_KEEPALIVE` 属于套接字基础层（`SOL_SOCKET`）选项，是通用套接字属性。
> 2. **使用注意**：
>    - `SO_REUSEPORT` 需要操作系统内核支持（如 Linux 3.9+）；
>    - `TCP_NODELAY` 禁用 Nagle 算法可能增加网络小包数量，需在 “实时性” 和 “网络开销” 间权衡；
>    - `SO_KEEPALIVE` 的探测间隔由系统参数控制，无法通过该选项自定义。

### 补充说明

1. **核心设计逻辑**：
   - Socket 类是对操作系统 socket fd 的**轻量级封装**，仅聚焦 socket 本身的操作（bind/listen/accept/ 选项设置），不涉及事件循环、线程调度等逻辑；
   - 遵循 “单一职责”：只管理 fd 的生命周期和 socket 操作，<u>事件监听、读写处理由 EventLoop/Channel 负责</u>，解耦 “fd 操作” 和 “事件驱动”。
2. **非拷贝设计的必要性**：
   - 若允许拷贝，会导致多个 Socket 实例持有同一个 fd，析构时重复 close (fd)，引发未定义行为；
   - 继承`noncopyable`禁用拷贝构造和赋值运算符，强制通过指针 / 引用传递 Socket 实例，保证 fd 唯一归属。
3. **与其他组件的协作**：
   - 与`InetAddress`：通过`InetAddress`封装地址信息（IP / 端口），避免直接操作 sockaddr 结构体，简化地址相关逻辑；
   - 与`Acceptor`：Acceptor 持有监听 Socket 实例，调用`listen()`启动监听，在新连接到来时调用`accept()`获取新 fd；
   - 与`TcpConnection`：TcpConnection 持有连接对应的 Socket 实例，负责连接的读写、关闭（`shutdownWrite()`）等操作。

### 问题解释⬇︎

> 为什么Socket类没有封装`socket()`创建套接字的函数，`connect()`客户端发起连接的函数以及操作read、write等函数呢？

这个设计是**面向 “封装已有套接字 fd”** 的职责划分，而非 “包揽套接字全生命周期”，核心原因是**单一职责原则** + 网络编程架构分层，具体拆解：

#### 1. 为什么没有`socket()`（创建套接字）？

- **职责分离**：`Socket`类的定位是 “封装一个已存在的套接字 fd 的操作”，而非 “创建套接字”。创建套接字的逻辑（`socket()`系统调用）通常会放在**套接字创建器 / 工厂类** 或 **服务器初始化逻辑** 中（比如`TcpServer`类），而非`Socket`类。

  举例：服务器创建监听套接字的流程可能是：

  ```cpp
  // 伪代码：TcpServer初始化时创建监听fd
  int listenfd = ::socket(AF_INET, SOCK_STREAM, 0); // 单独的创建逻辑
  Socket listenSocket(listenfd); // 用Socket类封装已创建的fd
  listenSocket.bindAddress(localAddr);
  listenSocket.listen();
  ```

- **灵活性**：`socket()`的参数（协议族、类型、协议）可能因场景不同而变化（比如 IPv4/IPv6、TCP/UDP），如果把`socket()`塞进`Socket`类，会让类变得臃肿，且无法适配多场景。

#### 2. 为什么没有`connect()`（客户端发起连接）？

- **客户端 / 服务器职责分离**：`connect()`是**客户端专属操作**，而当前`Socket`类主要服务于**服务器端**（监听、接受连接）。

- **架构分层**：客户端发起连接的逻辑通常封装在`TcpClient`类中，`TcpClient`会先创建 fd、调用`connect()`，再把 fd 交给`Socket`类封装后续操作。

  举例：客户端流程：

  ```cpp
  // 伪代码：TcpClient发起连接
  int sockfd = ::socket(AF_INET, SOCK_STREAM, 0);
  Socket clientSocket(sockfd);
  ::connect(sockfd, (sockaddr*)&serverAddr, sizeof(serverAddr)); // connect在Client类中
  clientSocket.setTcpNoDelay(true); // 用Socket类设置fd选项
  ```

- **`connect()`的特殊性**：`connect()`是 “阻塞 / 非阻塞” 行为差异极大的系统调用（非阻塞`connect`需要配合`poll/epoll`检测可写事件），把它塞进基础的`Socket`类会增加复杂度，不如让上层业务（`TcpClient`）处理。

#### 3. 为什么没有`read()`/`write()`（读写数据）？

- **IO 模型适配**：该代码库的注释中提到`Reactor模型 one loop per thread` + `poller + non-blocking IO`，说明这是**非阻塞 IO + 事件驱动** 架构：

  - 传统的`read()`/`write()`是 “同步阻塞” 接口，无法适配 Reactor 模型；
  - 事件驱动架构中，数据读写依赖`epoll/poll/select`检测 fd 的 “可读 / 可写” 事件，再通过`recv()`/`send()`（或更底层的`readv()`/`writev()`）执行读写，且读写逻辑会封装在**IO 缓冲区类（如`Buffer`）** 或 **事件处理类（如`Channel`）** 中，而非基础的`Socket`类。

- **职责边界**：`Socket`类只负责 “套接字 fd 的属性配置、生命周期管理（关闭 /shutdown）”，而 “数据传输” 属于更高层的 IO 逻辑，交给`Channel`/`Buffer`/`TcpConnection`等类处理。

  举例：Reactor 模型中，`Channel`类关联`Socket`的 fd，监听`EPOLLIN`（可读）事件，触发后调用`TcpConnection`的`handleRead()`，最终通过`Buffer`的`readFd()`读取数据：

  ```cpp
  // 伪代码：事件驱动的读数据
  void Channel::handleEvent() {
      if (revents_ & EPOLLIN) {
          tcpConnection_->handleRead(); // 处理读事件
      }
  }
  
  void TcpConnection::handleRead() {
      int savedErrno = 0;
      ssize_t n = inputBuffer_.readFd(socket_->fd(), &savedErrno); // 从fd读数据到缓冲区
      // ...
  }
  ```

### 三、总结

这个`Socket`类是 **“轻量级 fd 封装器”**，只聚焦 “已创建的套接字 fd” 的核心操作（绑定、监听、接受连接、属性配置、优雅关闭），而把 “创建 fd（socket ()）”“客户端连接（connect ()）”“数据读写（read/write）” 等职责拆分到上层的`TcpServer`/`TcpClient`/`Channel`/`Buffer`等类中，符合网络编程中 “分层、单一职责” 的设计思想，也适配 Reactor 事件驱动模型的架构需求。

这种设计的优势是：类的职责清晰、可复用性高（比如`Socket`类既可以封装服务器的监听 fd，也可以封装客户端的连接 fd），且上层可以灵活适配不同的 IO 模型、客户端 / 服务器场景。