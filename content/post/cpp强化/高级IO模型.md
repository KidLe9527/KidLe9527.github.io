# 高级 I/O 模型与并发编程 全面解析

本文将从**核心概念、语法细节、知识点拆解、代码实战**四个维度，系统讲解高级 I/O 模型（阻塞 / 非阻塞 I/O、多路复用）和并发编程（多线程 / 多进程、线程池、异步 I/O），让你不仅理解原理，还能上手写代码。

## 一、基础铺垫：文件描述符（fd）

所有 I/O 模型的核心是**文件描述符（File Descriptor，fd）** —— 操作系统为每个打开的文件（包括网络套接字、管道、普通文件）分配的整数标识（如`0`= 标准输入、`1`= 标准输出、`2`= 标准错误）。

后续所有 I/O 操作都是围绕`fd`展开，这是理解所有 I/O 模型的前提。

## 二、阻塞 I/O（Blocking I/O）

### 1. 核心概念

程序发起 I/O 操作后，**若数据未准备好，线程会被内核挂起（阻塞）**，直到数据就绪并完成 I/O 操作，线程才被唤醒继续执行。

- 同步特性：I/O 未完成时，线程无法做任何其他工作
- 资源浪费：阻塞期间线程占用 CPU 调度资源，但无实际计算

### 2. 关键语法 / 函数

Linux 下默认的 I/O 函数都是阻塞模式：

| 函数                  | 用途                 | 阻塞场景         |
| --------------------- | -------------------- | ---------------- |
| `read(fd, buf, len)`  | 从 fd 读取数据到 buf | 无数据可读时阻塞 |
| `write(fd, buf, len)` | 从 buf 写入数据到 fd | 缓冲区满时阻塞   |
| `accept(sockfd, ...)` | 监听套接字等待连接   | 无新连接时阻塞   |
| `recv/send`           | 网络套接字读写       | 同 read/write    |

### 3. 代码示例（阻塞读取套接字）

```cpp
#include <stdio.h>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <errno.h>

int main() {
    // 1. 创建TCP套接字（fd）
    int sockfd = socket(AF_INET, SOCK_STREAM, 0);
    if (sockfd < 0) { perror("socket fail"); return -1; }

    // 2. 初始化地址，然后绑定地址和端口
    struct sockaddr_in addr{};
    addr.sin_family = AF_INET;
    addr.sin_port = htons(8080);
    addr.sin_addr.s_addr = INADDR_ANY;
    if (bind(sockfd, (struct sockaddr*)&addr, sizeof(addr)) < 0) {
        perror("bind fail"); close(sockfd); return -1;
    }

    // 3. 监听端口（阻塞核心：accept会等连接）
    listen(sockfd, 5);
    printf("等待客户端连接...\n");
    
    // 4. 阻塞等待客户端连接（无连接时，线程卡在这里）
    int client_fd = accept(sockfd, nullptr, nullptr);//主动放弃获取客户端地址信息以简化代码、减少不必要的资源开销。
    if (client_fd < 0) { perror("accept fail"); close(sockfd); return -1; }
    printf("客户端已连接\n");

    // 5. 阻塞读取客户端数据（无数据时，线程卡在这里）
    char buffer[1024] = {0};
    ssize_t n = read(client_fd, buffer, sizeof(buffer)-1);
    if (n < 0) { perror("read fail"); close(client_fd); close(sockfd); return -1; }
    printf("收到数据：%s\n", buffer);

    // 6. 关闭fd
    close(client_fd);
    close(sockfd);
    return 0;
}
```

### 4. 知识点拆解

- 阻塞的本质：内核将线程状态置为「睡眠」，直到 I/O 事件触发（如数据到达），再将线程唤醒
- 适用场景：简单小程序（如单客户端通信），无需高并发，编程成本低
- 缺点：单线程下只能处理一个 I/O 事件，多线程虽能解决但会带来线程开销

## 三、非阻塞 I/O（Non-blocking I/O）

### 1. 核心概念

通过`fcntl`将`fd`设置为非阻塞模式后，发起 I/O 操作时：---> file control

- 数据就绪：正常执行 I/O，返回结果
- 数据未就绪：立即返回错误（`errno = EWOULDBLOCK` 或 `EAGAIN`），线程不阻塞
  - 若「未决连接队列」为空，`accept()` 立即返回 `-1`，且 `errno = EAGAIN` 或 `EWOULDBLOCK`（无错误，仅表示 “暂无连接”）；

### 2. 关键语法 / 函数

#### （1）设置非阻塞模式

```cpp
#include <fcntl.h>
// 步骤：1. 获取当前fd的标志 2. 添加O_NONBLOCK标志 3. 设置新标志
int set_nonblock(int fd) {
    int flags = fcntl(fd, F_GETFL, 0); // 调用 fcntl 函数，以 F_GETFL（File GET FLags）为操作指令，获取 fd 当前的「文件状态标志」
    if (flags == -1) return -1;
    return fcntl(fd, F_SETFL, flags | O_NONBLOCK); // 添加非阻塞标志，File SET FLags
}
```

> 为什么要先获取现有标志？
>
> * 文件描述符的标志是**位掩码**（多个标志通过二进制位组合），比如 fd 可能同时有 `O_RDWR`（可读可写）、`O_APPEND`（追加写）等标志。如果直接设置 `O_NONBLOCK`，会覆盖原有标志，导致其他属性丢失。因此必须先获取现有标志，再「追加」非阻塞标志。
>
> * 错误处理：如果 `fcntl` 返回 - 1，说明获取标志失败（如 fd 无效、权限不足），直接返回 - 1 告知调用者。
>
> * `fcntl` 是 Linux/Unix 系统中操作文件描述符的「万能函数」，不仅能设置非阻塞标志，还能实现：
>
>   | 操作指令           | 作用                                              |
>   | ------------------ | ------------------------------------------------- |
>   | `F_GETFL`          | 获取文件状态标志（如 O_NONBLOCK、O_APPEND）       |
>   | `F_SETFL`          | 设置文件状态标志（仅部分标志可改，如 O_NONBLOCK） |
>   | `F_GETFD`          | 获取文件描述符标志（如 FD_CLOEXEC）               |
>   | `F_SETFD`          | 设置文件描述符标志                                |
>   | `F_SETLK/F_SETLKW` | 设置文件锁（共享锁 / 排他锁）                     |

#### （2）错误码判断

| 错误码             | 含义                 | 处理方式         |
| ------------------ | -------------------- | ---------------- |
| EWOULDBLOCK/EAGAIN | 数据未就绪（非错误） | 稍后重试         |
| EBADF              | fd 无效              | 检查 fd 是否正确 |

### 3. 代码示例（非阻塞读取套接字）

```cpp
#include <stdio.h>	// 仅处理一次连接 + 一次数据
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <fcntl.h>
#include <errno.h>
#include <string.h>

// 设置非阻塞模式
int set_nonblock(int fd) {
    int flags = fcntl(fd, F_GETFL, 0);
    if (flags == -1) return -1;
    return fcntl(fd, F_SETFL, flags | O_NONBLOCK);
}

int main() {
    int sockfd = socket(AF_INET, SOCK_STREAM, 0);
    if (sockfd < 0) { perror("socket fail"); return -1; }

    // 关键：将监听套接字设为非阻塞
    if (set_nonblock(sockfd) < 0) { perror("set_nonblock fail"); close(sockfd); return -1; }

    struct sockaddr_in addr{};
    addr.sin_family = AF_INET;
    addr.sin_port = htons(8080);
    addr.sin_addr.s_addr = INADDR_ANY;
    if (bind(sockfd, (struct sockaddr*)&addr, sizeof(addr)) < 0) {
        perror("bind fail"); close(sockfd); return -1;
    }
    listen(sockfd, 5);

    printf("非阻塞模式：轮询等待连接...\n");
    int client_fd = -1;	// 这里等于-1表示还没有连接客户端
    // 轮询检查是否有新连接（非阻塞核心：accept立即返回）
    while (client_fd < 0) {
        client_fd = accept(sockfd, nullptr, nullptr);
        if (client_fd < 0) {
            if (errno == EWOULDBLOCK || errno == EAGAIN) {	 // errorno全局变量保存最近一次系统调用错误码
                // 无连接，做其他工作（示例：打印提示）
                printf("暂无连接，执行其他任务...\n");
                sleep(1); // 线程会被内核挂起 1 秒，此时 CPU 会切换到其他进程 / 线程执行，完全不占用 CPU 资源；
                continue;
            } else {
                perror("accept fail"); close(sockfd); return -1;
            }
        }
    }
    printf("客户端已连接\n");

    // 客户端fd也设为非阻塞，读取数据
    set_nonblock(client_fd);
    char buffer[1024] = {0};
    ssize_t n = -1;
    while (n < 0) {
        n = read(client_fd, buffer, sizeof(buffer)-1);
        if (n < 0) {
            if (errno == EWOULDBLOCK || errno == EAGAIN) {
                printf("暂无数据，等待...\n");
                sleep(1);
                continue;
            } else {
                perror("read fail"); close(client_fd); close(sockfd); return -1;
            }
        }
    }
    printf("收到数据：%s\n", buffer);

    close(client_fd);
    close(sockfd);
    return 0;
}
```

### 4. 知识点拆解

- 非阻塞的本质：I/O 操作不挂起线程，无论结果如何立即返回
- 轮询问题：纯非阻塞 I/O 需要不断轮询`fd`状态，浪费 CPU（实际需配合多路复用）
- 适用场景：高并发场景（需结合多路复用），避免单线程阻塞

## 四、多路复用（select/poll/epoll）

### 1. 核心概念

多路复用允许**单个线程监控多个 fd**，内核会告知线程哪些 fd 已就绪（可读 / 可写 / 异常），线程只需处理就绪的 fd，避免轮询或阻塞单个 fd。

- 核心价值：用单线程管理多 fd，大幅提升并发能力
- 同步模型：线程仍会阻塞在多路复用函数上，但阻塞的是「等待任意 fd 就绪」，而非单个 fd

### 2. select：跨平台基础实现

> **定义**：内核提供的系统调用，能批量**监控**多个 fd 的可读 / 可写 / 异常事件，阻塞等待直到事件就绪或超时。
>
> **作用**：让单线程高效处理多 fd 事件，解决阻塞 I/O 串行、非阻塞轮询 CPU 空转的问题，最大化 CPU 利用率，实现多事件并发响应。

#### （1）关键语法

```c
#include <sys/select.h>
// 参数说明：
// nfds：监控的最大fd+1		-- 如果nfds = 6，则select会监控0,1,2,3,4,5共6个fd。
// readfds：待监控的可读fd集合
// writefds：待监控的可写fd集合
// exceptfds：待监控的异常fd集合
// timeout：超时时间（nullptr=永久阻塞，0=立即返回）
int select(int nfds, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);

// fd_set操作宏
FD_ZERO(fd_set *set);    // 清空集合
FD_SET(int fd, fd_set *set);  // 将fd加入集合
FD_CLR(int fd, fd_set *set);  // 将fd移出集合
FD_ISSET(int fd, fd_set *set); // 检查fd是否在就绪集合中 // 仅在 select 返回值 > 0 时调用才有意义（返回 0 表示超时，无 fd 就绪；返回 - 1 表示出错）。
```

> * `fd_set` 是系统定义的**文件描述符（fd）集合类型**
> * 注意：系统对 `fd_set` 能容纳的最大 fd 数量有限制（由 `FD_SETSIZE` 宏定义，通常是 1024），这也是 select 的经典缺陷之一。
> * `fd_set*` 是 `select` 机制的核心载体：
>   - 作为**输入**：告诉 `select` 需要监控哪些 fd 的可读 / 可写 / 异常事件；
>   - 作为**输出**：`select` 返回时，通过它告知调用者哪些 fd 已就绪；
>   - 指针传递：实现「输入 + 输出」的双向交互，同时减少拷贝开销。
>
> 其本质是位图的封装，通过 `FD_ZERO/FD_SET/FD_CLR/FD_ISSET` 四个宏操作，完成对 fd 集合的管理。

#### （2）代码示例（监控多个 fd）

```cpp
#include <stdio.h>
#include <unistd.h>
#include <sys/select.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <fcntl.h>

int main() {
    // 1. 创建监听套接字
    int sockfd = socket(AF_INET, SOCK_STREAM, 0);
    struct sockaddr_in addr{};
    addr.sin_family = AF_INET;
    addr.sin_port = htons(8080);
    addr.sin_addr.s_addr = INADDR_ANY;
    bind(sockfd, (struct sockaddr*)&addr, sizeof(addr));
    listen(sockfd, 5);

  	// 核心功能是同时监控「监听套接字（新连接事件）」和「标准输入（控制台输入事件）」，并在对应事件就绪时做出响应 🔥🔥
    fd_set read_fds;
    int max_fd = sockfd; // 监控的最大fd

    while (1) {
        // 每次调用select前必须重新初始化集合（select会修改集合）
      	/*
      	select 调用返回时会修改传入的 fd_set（只保留「就绪的 fd」对应的比特位为 1，其余置 0）。如果不重新初始化，下一次调用时，read_fds 中只会保留上一次就绪的 fd，导致漏监控其他 fd。*/ 🔥
        FD_ZERO(&read_fds);
        FD_SET(sockfd, &read_fds); // 监控监听套接字（新连接）
        FD_SET(STDIN_FILENO, &read_fds); // 将标准输入（fd=0）加入集合，监控控制台输入的可读事件。

        // 调用select，阻塞等待fd就绪
        struct timeval timeout{5, 0}; // 5秒超时，时间结构体，聚合初始化
        int ret = select(max_fd + 1, &read_fds, nullptr, nullptr, &timeout);
        
        if (ret < 0) { perror("select fail"); break; }
        if (ret == 0) { printf("select超时，无fd就绪\n"); continue; }

        // 检查哪个fd就绪
        // 1. 标准输入就绪
        if (FD_ISSET(STDIN_FILENO, &read_fds)) {
            char buf[1024] = {0};
            read(STDIN_FILENO, buf, sizeof(buf)-1);
            printf("控制台输入：%s", buf);
        }
        // 2. 监听套接字就绪（新连接）
        if (FD_ISSET(sockfd, &read_fds)) {
            int client_fd = accept(sockfd, nullptr, nullptr);
            printf("新客户端连接：%d\n", client_fd);
        }
    }

    close(sockfd);
    return 0;
}
```

#### （3）知识点拆解

- 缺点 1：fd 数量限制（默认`FD_SETSIZE=1024`），无法监控更多 fd
- 缺点 2：每次调用 select 需重新传入 fd 集合（内核会修改集合）
- 缺点 3：内核采用轮询方式检查 fd 状态，时间复杂度 O (n)
- 适用场景：跨平台小程序，fd 数量少

> | 维度             | 非阻塞 + 轮询版本                                        | select 多路复用版本                                          |
> | ---------------- | -------------------------------------------------------- | ------------------------------------------------------------ |
> | **等待方式**     | 主动轮询（while 循环 + sleep），CPU 空转                 | 被动等待（内核阻塞），CPU 仅在 fd 就绪时唤醒                 |
> | **并发能力**     | 单线程只能处理「一个客户端 + 一个监听 fd」，无法并行处理 | 单线程可同时监控「监听 fd + 多个客户端 fd + 标准输入」，真正并发 |
> | **资源利用率**   | 轮询时 sleep 仍会占用 CPU（低效率），且 sleep 时间难把控 | 无 fd 就绪时进程阻塞，CPU 完全释放，就绪后精准唤醒           |
> | **fd 管理方式**  | 逐个处理（处理完一个客户端才能接下一个）                 | 批量监控（同时监控所有 fd，哪个就绪处理哪个）                |
> | **非阻塞的作用** | 核心依赖非阻塞：避免 accept/read 阻塞导致程序卡死        | 非阻塞可选（select 本身是阻塞等待，就绪后读写可阻塞 / 非阻塞） |
> | **事件覆盖范围** | 仅处理「监听 fd（连接）+ 客户端 fd（读写）」             | 可同时处理「监听 fd + 客户端 fd + 标准输入 / 其他 fd」       |
>
> #### 1. 「等待 fd 就绪」的底层逻辑（最核心）
>
> - **非阻塞 + 轮询版本**：
>
>   程序主动「问」内核：“这个 fd 就绪了吗？”（调用 accept/read），如果没就绪（errno=EWOULDBLOCK/EAGAIN），就 sleep 1 秒再问 —— 本质是**CPU 主动轮询**，哪怕没有任何事件，也会每隔 1 秒唤醒一次，属于 “忙等”（低效）。
>
>   且因为是「嵌套循环」（外层等连接，内层处理单个客户端），处理客户端 A 时，无法接受客户端 B 的连接，完全是 “串行处理”。
>
> - **select 版本**：
>
>   * 程序把「要监控的 fd 列表」交给内核：“帮我盯着这些 fd，哪个就绪了再叫醒我”—— 内核在内核态阻塞等待，直到有 fd 就绪（或超时），再通知用户态程序。
>
>   * 这个过程中进程是「休眠」状态，CPU 完全释放给其他进程，只有事件发生时才被唤醒，属于 “闲等”（高效）。
>
>   * 且 select 能同时监控多个 fd，比如 “监听 fd 就绪（新连接）” 和 “标准输入就绪（控制台输入）” 可以并行响应。也就是说，只要其中一个fd就绪就可以响应，多个fd之间没有先后依赖
>
> #### 2. 并发能力的天壤之别
>
> - **非阻塞 + 轮询版本**：
>
>   假设客户端 A 连接后一直不发数据，程序会卡在「客户端 A 的 read 轮询循环」里，此时即使有客户端 B 发起连接，也无法处理 ——**单线程下完全无并发能力**，只能 “一对一” 处理。
>
> - **select 版本**：
>
>   只要把新 accept 的 client_fd 加入 read_fds，select 就能同时监控「监听 fd（新连接）+ 客户端 A fd（读数据）+ 客户端 B fd（读数据）+ 标准输入」—— 哪个 fd 就绪就处理哪个
>
>   > 若多个 fd 同时就绪（比如 fdA 和 fdB 都就绪），select 返回后，进程会按代码里的检查顺序（比如先检查 fdA、再检查 fdB）**依次处理**，而非 “同时处理”
>   >
>   > “并行响应” 是「事件等待阶段的并行」（内核同时监控所有 fd），而非「事件处理阶段的并行」（单线程只能串行处理）。
>
> 总结select的优点！
>
> 1. “一个 fd 正在执行，另一个 fd 转为就绪态，只能等当前 fd 执行完”→ **完全正确**（单线程的本质）；
> 2. “本质还是单线程，轮流处理看起来像并发，其实还是串行”→ **正确**（处理阶段串行，等待阶段并行，所以是 “伪并发”）；
> 3. “多个 fd 同时就绪，先处理谁取决于代码里 FD_ISSET 的检查顺序”→ **正确**（工人按自己的检查顺序干活，主任只负责告诉 “哪些活能做”，不决定顺序）。

### 3. poll：突破 fd 数量限制

> `poll` 是 Unix/Linux 系统中用于**I/O 多路复用**的核心系统调用，作用是监控多个文件描述符（fd）的状态变化（可读 / 可写 / 异常），并在其中任意 fd 就绪或超时后返回，避免了传统 `select` 函数的 fd 数量限制（`select` 受 `FD_SETSIZE` 限制），是高并发网络编程的基础工具。

#### （1）关键语法

```c
#include <poll.h>
// pollfd结构体：描述单个需要监控的文件描述符及事件
struct pollfd {
    int   fd;         // 要监控的文件描述符，若设为 -1，则该结构体被忽略（events/revents 无效）
    short events;     // 要监控的事件（POLLIN=可读，POLLOUT=可写）
    short revents;    // 实际发生的事件（由内核填充）--> 输出参数，初始化可设为0（内核会覆盖）
};
// 参数：
// fds：pollfd数组
// nfds：数组长度
// timeout：超时时间（毫秒，-1=永久阻塞，0=立即返回）
int poll(struct pollfd *fds, nfds_t nfds, int timeout);
```

> * `events` 是**输入参数**，由用户通过「事件掩码」（位运算组合）设置，告诉内核需要关注的事件类型。支持的核心事件如下（POSIX 标准，不同系统可能有扩展）
>
>   | 事件宏        | 含义                                                         |
>   | ------------- | ------------------------------------------------------------ |
>   | **`POLLIN`**  | 有数据可读（普通数据 / 优先数据），如 socket 收到数据、管道有输入、标准输入有按键 |
>   | `POLLPRI`     | 有高优先级数据可读（如带外数据 OOB，常见于 TCP 紧急模式）    |
>   | **`POLLOUT`** | fd 可写（无阻塞时不会阻塞写操作），如 socket 连接建立后可发数据、缓冲区有空闲 |
>   | `POLLRDHUP`   | 对端关闭连接（仅流式 socket，如 TCP），或关闭写半连接（对方 `shutdown(SHUT_WR)`） |
>   | **`POLLERR`** | fd 发生错误（无需用户设置，内核自动检测并返回在 `revents`）  |
>   | `POLLHUP`     | fd 挂起（如管道读端关闭、socket 连接异常断开）（无需用户设置，内核自动返回） |
>   | `POLLNVAL`    | fd 无效（如未打开、已关闭）（无需用户设置，内核自动返回）    |
>
>   * #### 用法：通过位或（`|`）组合多个监控事件
>
>     * 例如：同时监控 fd 的「可读」和「可写」事件：pfd.events = POLLIN | POLLOUT;    // 请求监控：可读 + 可写
>
> * `revents` 是**输出参数**，由内核填充，用于告知用户「监控的 fd 实际发生了哪些事件」，**绝对不能由用户手动设置**（设置了也会被内核覆盖）
>
>   * `revents` 的事件是 `events` 的「子集或扩展」：
>     - 内核只会返回用户在 `events` 中请求的事件（如用户只设 `POLLIN`，`revents` 不会出现 `POLLOUT`）；
>     - 但会额外返回「错误类事件」（`POLLERR`/`POLLHUP`/`POLLNVAL`），无论是否在 `events` 中设置（内核强制通知错误）。
>   * 通过「位与（`&`）」判断是否发生某个事件：
>     内核用「位掩码」存储结果，**需通过位与运算提取目标事件**（避免其他事件干扰）。

#### （2）代码示例

```cpp
#include <stdio.h>
#include <unistd.h>
#include <poll.h>
#include <sys/socket.h>
#include <netinet/in.h>

int main() {
    int sockfd = socket(AF_INET, SOCK_STREAM, 0);
    struct sockaddr_in addr{};
    addr.sin_family = AF_INET;
    addr.sin_port = htons(8080);
    addr.sin_addr.s_addr = INADDR_ANY;
    bind(sockfd, (struct sockaddr*)&addr, sizeof(addr));
    listen(sockfd, 5);

    // 定义要监控的fd集合
    struct pollfd fds[2];
    // 监控监听套接字（可读事件）
    fds[0].fd = sockfd;
    fds[0].events = POLLIN;
    // 监控标准输入（可读事件）
    fds[1].fd = STDIN_FILENO;
    fds[1].events = POLLIN;

    while (1) {
        // 调用poll，阻塞等待事件
        int ret = poll(fds, 2, 5000); // 5秒超时
        if (ret < 0) { perror("poll fail"); break; }
        if (ret == 0) { printf("poll超时\n"); continue; }

        // 检查监听套接字
      	// 🔥fds[0].revents & POLLIN：位运算判断 revents 中是否包含 POLLIN 事件（内核告知该 fd 可读）
        if (fds[0].revents & POLLIN) {
            int client_fd = accept(sockfd, nullptr, nullptr);
            printf("新客户端：%d\n", client_fd);
        }
        // 检查标准输入
        if (fds[1].revents & POLLIN) {
            char buf[1024] = {0};
            read(STDIN_FILENO, buf, sizeof(buf)-1); // STDIN_FILENO：系统宏定义（值为 0），等价于直接写 0，语义更清晰
            printf("控制台输入：%s", buf);
        }
    }

    close(sockfd);
    return 0;
}
```

> #### 1. 事件判断的位运算
>
> `revents` 是**位图**（多个事件可同时存在，如 `POLLIN | POLLHUP`），必须用 `&` 而非 `==` 判断：
>
> - 错误写法：`if (fds[0].revents == POLLIN)`（若同时有 `POLLIN | POLLHUP`，会判断失败）；
> - 正确写法：`if (fds[0].revents & POLLIN)`（只要包含 `POLLIN` 就触发）。
>
> #### 2. 非阻塞特性
>
> 调用 `poll` 后，只要 `revents` 包含 `POLLIN`，此时 `accept`/`read` 都是**无阻塞**的（不会卡在调用处），这是 I/O 多路复用的核心价值 —— 避免单个 fd 阻塞导致整个程序无法响应其他事件。
>
> #### 3. 多个事件可同时触发
>
> 若 “新客户端连接” 和 “控制台输入” 同时发生，`poll` 的返回值会是 2（两个 fd 有事件），此时两个 `if` 分支都会执行（而非互斥）。

#### （3）知识点拆解

- 优点：无 fd 数量限制（仅受系统资源限制）
- 缺点：仍采用轮询方式，时间复杂度 O (n)
- 适用场景：跨平台、fd 数量超过 1024 的场景

> * `select` VS `poll`
>
>   * 文件描述符数量限制
>
>   | 受系统宏 `FD_SETSIZE` 硬性限制（默认值为 1024），超出该数值的文件描述符无法被监控（需修改内核编译参数才能调整上限） | 无固定数量限制，可监控的文件描述符数量仅受系统最大文件描述符数（`RLIMIT_NOFILE`）和系统内存资源约束 |
>   | ------------------------------------------------------------ | ------------------------------------------------------------ |
>
>   * 事件集合初始化效率
>
>   | 采用读、写、异常三个独立的文件描述符集合，且集合为 “输入输出复用” 模式：内核调用后会修改集合内容（仅保留触发事件的 fd），因此每次调用前需重新初始化所有集合，初始化开销高 | 基于 `struct pollfd` 数组实现事件描述，`events`（待监控事件，输入型）与 `revents`（实际触发事件，输出型）分离，无需重复初始化事件集合，仅需读取内核返回的 `revents`，初始化效率显著提升 |
>   | ------------------------------------------------------------ | ------------------------------------------------------------ |
>
>   * 多事件并发处理能力
>
>   | 多事件触发时，需分别遍历读、写、异常三个集合逐一核对事件归属的文件描述符，事件关联逻辑分散，易因集合遍历疏漏导致事件漏处理 | 单个文件描述符的所有待监控 / 已触发事件均封装在 `pollfd` 结构体中，多事件并发触发时，仅需通过位运算（`&`）解析单个 `pollfd` 的 `revents` 字段即可完整获取所有事件，事件处理逻辑集中，无漏处理风险 |
>   | ------------------------------------------------------------ | ------------------------------------------------------------ |
>
>   * 非阻塞 I/O 体验
>
>   | 核心逻辑支持非阻塞 I/O（仅对触发事件的文件描述符执行 I/O 操作），但因集合重复初始化、多集合遍历等额外操作，整体流程繁琐，开发与运行效率均较低 | 非阻塞 I/O 核心逻辑与 `select` 一致，但得益于事件集合的输入输出分离设计，无需冗余的初始化与遍历操作，非阻塞 I/O 流程更简洁，开发维护成本更低，运行效率更优 |
>   | ------------------------------------------------------------ | ------------------------------------------------------------ |

### 4. epoll：Linux 高性能多路复用

#### （1）关键语法

```c
#include <sys/epoll.h>
// 1. 创建epoll实例（返回epoll_fd） -- 返回值：成功返回非负的 epoll 文件描述符（epfd），失败返回 -1 并设置 errno。
int epoll_create(int size); // size：历史参数，现在可传任意正数,建议1024、

// 2. 管理epoll监控的fd（添加/修改/删除）
// epfd：epoll_create() 返回的实例ep的有效句柄
// op：EPOLL_CTL_ADD（添加）、EPOLL_CTL_MOD（修改）、EPOLL_CTL_DEL（删除） --> 🔥不能重复添加、删除需已加，不然报错
// fd：要监控的fd
// event：要监控的事件
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event); // 返回值：0 / -1 + errno

// 3. 等待事件就绪
// events：输出参数，（events 数组）仅保存本次就绪的事件，内核不会清空未就绪的 fd 信息
// maxevents：最多返回的事件数
// timeout：超时时间（毫秒，-1=永久阻塞）
int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout);

// 事件结构体 ---> 内置类型
struct epoll_event {
    uint32_t     events; // 事件类型（EPOLLIN=可读，EPOLLOUT=可写）｜ 位掩码，可以组合使用
    epoll_data_t data;   // 自定义数据（通常存fd）
};
typedef union epoll_data {	// 联合体成员互斥：同一时间只能使用一个成员（如设置 fd 后，ptr 无意义）。
    void        *ptr;	// 指向自定义数据的指针（如连接信息）
    int          fd;	// 最常用：文件描述符，epoll：监听相关事件
    uint32_t     u32;	// 32位无符号整数
    uint64_t     u64;	// 64位无符号整数
} epoll_data_t;

// 触发模式
#define EPOLLLT 0x0     // 水平触发（默认）：只要fd就绪，每次epoll_wait都返回
#define EPOLLET 0x8     // 边缘触发：仅当fd状态从「未就绪」变「就绪」时返回
```

> * `events`：位掩码，支持**多事件组合**（用 `|` 拼接），核心取值：
>
> | 宏常量         | 语义               | 适用场景                                              |
> | -------------- | ------------------ | ----------------------------------------------------- |
> | `EPOLLIN`      | 可读事件           | 套接字有数据可读、管道读端就绪、fd 关闭（触发读事件） |
> | `EPOLLOUT`     | 可写事件           | 套接字发送缓冲区空闲、管道写端就绪                    |
> | `EPOLLERR`     | 错误事件           | fd 发生错误（无需手动设置，内核自动触发）             |
> | `EPOLLHUP`     | 挂起事件           | fd 被关闭（无需手动设置，内核自动触发）               |
> | `EPOLLET`      | 边缘触发（ET）模式 | 与 EPOLLIN/EPOLLOUT 组合使用（默认是水平触发 LT）     |
> | `EPOLLONESHOT` | 一次性事件         | 触发一次后，fd 自动从 epoll 表中移除（需重新 ADD）    |
>
> - `union`：C 语言的**共用体（联合）** 关键字，表示该类型的所有成员**共用同一块内存空间**（内存大小 = 最大成员的大小），同一时间只能有一个成员有效（写一个成员会覆盖其他成员的值）。

#### （2）代码示例（ET 模式 + 非阻塞）

```cpp
#include <stdio.h>
#include <unistd.h>
#include <sys/epoll.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <fcntl.h>
#include <errno.h>
#include <string.h>

// 设置非阻塞
int set_nonblock(int fd) {
    int flags = fcntl(fd, F_GETFL, 0);
    return fcntl(fd, F_SETFL, flags | O_NONBLOCK);
}

int main() {
    // 1. 创建监听套接字并设为非阻塞
    int sockfd = socket(AF_INET, SOCK_STREAM, 0);
    set_nonblock(sockfd);
  
   	// 地址复用（避免TIME_WAIT导致重启失败）------------------------------------------------注意⚠️！！
    int opt = 1;    // 地址复用选项
    if (setsockopt(sockfd, SOL_SOCKET, SO_REUSEADDR, &opt, sizeof(opt)) < 0) {
        perror("设置地址复用失败");
        close(sockfd);
        return -1;
    }
  	// 地址复用必须在绑定地址和端口之前～～～
    struct sockaddr_in addr{};
    addr.sin_family = AF_INET;
    addr.sin_port = htons(8080);
    addr.sin_addr.s_addr = INADDR_ANY;
    bind(sockfd, (struct sockaddr*)&addr, sizeof(addr));
    listen(sockfd, 5);

    // 2. 创建epoll实例
    int epfd = epoll_create(1); // 参数无意义，仅需>0
    if (epfd < 0) { perror("epoll_create fail"); return -1; }

    // 3. 将监听套接字添加到epoll（边缘触发+可读）
    struct epoll_event ev{};
    ev.events = EPOLLIN | EPOLLET; // 可读 + 边缘触发
    ev.data.fd = sockfd;
    epoll_ctl(epfd, EPOLL_CTL_ADD, sockfd, &ev);

    // 4. 事件循环
    struct epoll_event events[1024]; // 保存就绪事件
    while (1) {
        // 等待事件就绪
        int n = epoll_wait(epfd, events, 1024, -1);
        if (n < 0) { perror("epoll_wait fail"); break; }

        // 处理就绪事件
        for (int i = 0; i < n; i++) {
            int fd = events[i].data.fd;
            // 监听套接字就绪（新连接）
            if (fd == sockfd) {
                while (1) {
                    int client_fd = accept(sockfd, nullptr, nullptr);
                    if (client_fd < 0) {
                        // 非阻塞模式下，无新连接时退出循环
                        if (errno == EWOULDBLOCK || errno == EAGAIN) break;
                        else { perror("accept fail"); break; }
                    }
                    printf("新客户端：%d\n", client_fd);
                    // 客户端fd设为非阻塞，添加到epoll（ET模式）
                    set_nonblock(client_fd);
                    ev.events = EPOLLIN | EPOLLET;
                    ev.data.fd = client_fd;
                    epoll_ctl(epfd, EPOLL_CTL_ADD, client_fd, &ev);
                }
            }
            // 客户端fd就绪（可读）
            else if (events[i].events & EPOLLIN) {
                char buffer[1024] = {0};
                ssize_t ret = 0;
                // ET模式需一次性读完所有数据
                while (1) {
                    ret = read(fd, buffer, sizeof(buffer)-1);
                    if (ret < 0) {
                        if (errno == EWOULDBLOCK || errno == EAGAIN) break;
                        else {
                            perror("read fail");
                            epoll_ctl(epfd, EPOLL_CTL_DEL, fd, nullptr);
                            close(fd);
                            break;
                        }
                    } else if (ret == 0) {
                        // 客户端关闭连接
                        printf("客户端%d断开\n", fd);
                        epoll_ctl(epfd, EPOLL_CTL_DEL, fd, nullptr);
                        close(fd);
                        break;
                    } else {
                        printf("客户端%d数据：%s\n", fd, buffer);
                        memset(buffer, 0, sizeof(buffer));
                    }
                }
            }
        }
    }

    close(epfd);
    close(sockfd);
    return 0;
}
```

#### （3）知识点拆解

- 核心优势：
  1. 事件驱动：内核记录就绪 fd，无需轮询，时间复杂度 O (1)
  2. 无 fd 数量限制：仅受系统最大文件描述符限制
  3. 内存共享：fd 集合只需向内核注册一次，无需重复拷贝
- 触发模式：
  - 水平触发（LT）：适合新手，容错高（漏处理也会再次返回）
  - 边缘触发（ET）：效率更高，但需配合非阻塞 I/O，一次性读完 / 写完数据，不然没有读取的数据会被遗留在缓存区，直到fd状态变化才会被再次读取
- 适用场景：Linux 高并发服务器（如 Nginx、Redis）

### 5. select/poll/epoll 对比总结

| 特性       | select                  | poll                 | epoll              |
| ---------- | ----------------------- | -------------------- | ------------------ |
| 最大 fd 数 | 有限（FD_SETSIZE=1024） | 无限制（系统资源）   | 无限制（系统资源） |
| 时间复杂度 | O (n)（轮询）           | O (n)（轮询）        | O (1)（事件驱动）  |
| 触发方式   | 仅 LT                   | 仅 LT                | LT/ET              |
| 内存拷贝   | 每次调用拷贝 fd 集合    | 每次调用拷贝 fd 集合 | 仅注册时拷贝一次   |
| 跨平台     | 所有 Unix/Linux/Windows | 所有 Unix/Linux      | 仅 Linux           |
| 适用场景   | 简单跨平台、少 fd       | 跨平台、多 fd        | Linux 高并发服务器 |



> * <-----  select 、 poll、epoll 的完整比较 ----->
>
> | 对比维度                  | select                                                      | poll                                                         | epoll（Linux 特有）                                          |
> | ------------------------- | ----------------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
> | **底层数据结构**          | 位图（fd_set）                                              | pollfd 结构体数组（struct pollfd []）                        | 内核红黑树 + 就绪事件链表                                    |
> | **时间复杂度**            | O (n)（遍历全部监控 fd）                                    | O (n)（遍历全部监控 fd）                                     | O (1)（仅遍历就绪 fd，无就绪则不遍历）                       |
> | **FD 数量限制**           | 硬限制（默认 1024，由 FD_SETSIZE 定义），需重新编译内核修改 | 无硬限制（仅受进程最大打开文件数限制）                       | 无硬限制（仅受系统内存 / 文件描述符上限限制）                |
> | **内核 / 用户态数据拷贝** | 每次调用拷贝全部 fd_set 到内核，返回时拷贝就绪位图到用户态  | 每次调用拷贝全部 pollfd 数组到内核，返回时拷贝就绪状态到用户态 | 仅初始化时拷贝监控 fd 到内核，后续通过 epoll_ctl 增删，就绪事件仅拷贝 “就绪 fd” 到用户态 |
> | **触发模式**              | 仅水平触发（LT）--> 文件描述符集合                          | 仅水平触发（LT）                                             | 支持水平触发（LT）+ 边缘触发（ET），还支持 EPOLLONESHOT（一次性触发） |
> | **重复监听处理**          | 每次调用需重新设置 fd_set（位图会被内核修改）               | 无需重新设置，但需遍历数组判断就绪状态                       | 无需重新设置，就绪事件独立存储，可复用监控列表               |
> | **错误处理**              | 仅返回总就绪数，需逐个 FD 判断 FD_ISSET                     | 返回总就绪数，需逐个遍历 pollfd 的 revents                   | 返回就绪 FD 数量，直接遍历就绪事件数组即可                   |
> | **支持的事件类型**        | 仅支持读（POLLIN）、写（POLLOUT）、异常                     | 支持更多事件（POLLIN/POLLOUT/POLLERR/POLLHUP 等）            | 兼容 poll 所有事件，新增 EPOLLPRI（紧急数据）、EPOLLRDHUP（半关闭）等 |
> | **高并发表现**            | 并发 > 1000 时效率急剧下降（位图遍历 + 拷贝开销）           | 并发 > 10000 时效率显著下降（数组遍历 + 拷贝开销）           | 百万级并发仍保持高效（仅处理就绪 FD）                        |
> | **内存开销**              | 固定大小（FD_SETSIZE），fd 越少越浪费                       | 随监控 FD 数线性增长，fd 越多开销越大                        | 内存开销与就绪 FD 数正相关，监控 FD 多但就绪少则开销极低     |
> | **使用复杂度**            | 简单（位图操作），但需手动管理 FD_SET                       | 中等（数组操作），无需关心 FD 编号                           | 稍复杂（需创建 epoll 实例、管理事件结构体），但封装后易用    |
> | **跨平台性**              | 跨 Unix/Linux/Windows（最通用）                             | 跨 Unix/Linux（比 select 少）                                | 仅 Linux 支持（无 POSIX 标准）                               |
> | **典型应用场景**          | 低并发（fd<1000）、跨平台场景                               | 中低并发（fd<10000）、需更多事件类型                         | 高并发（如 Nginx/Redis/MySQL）、高性能服务端                 |



## 五、并发编程：多线程 / 多进程

### 1. 多线程模型

#### （1）核心概念

- 线程是进程内的执行流，共享进程的内存空间（全局变量、堆、fd），但有独立栈空间
- 轻量级：创建 / 切换开销远小于进程
- 同步问题：多线程操作共享资源需加锁（如`std::mutex`），避免数据竞争

#### （2）关键语法（C++11 std::thread）

#### （3）知识点拆解

- `std::thread`：创建线程，参数为函数 + 函数参数
- `join()`：等待线程执行完毕，主线程阻塞
- `detach()`：分离线程（主线程不等待，线程后台运行）
- `std::mutex`：互斥锁，`lock()`加锁，`unlock()`解锁（推荐用`std::lock_guard`自动管理）
- 数据竞争：多个线程同时读写共享资源，会导致结果不可预测（如上述代码不加锁，count 可能小于 200000）

### 2. 多进程模型 🔥

#### （1）核心概念

- 进程是独立的执行单元，有独立的地址空间
- 隔离性：一个进程崩溃不影响其他进程
- 通信：需通过 IPC（管道、共享内存、消息队列、socket）

#### （2）关键语法（Linux fork）

```cpp
#include <stdio.h>
#include <unistd.h>	// Unix 标准库（fork、getpid、getppid）
#include <sys/wait.h>	// 进程等待相关函数（wait）

int main() {
    int count = 0;
    // 创建子进程
    pid_t pid = fork();
    if (pid < 0) { perror("fork fail"); return -1; }

    // 子进程（pid=0）
    if (pid == 0) {
        count += 10;
        printf("子进程：count=%d, pid=%d\n", count, getpid());	// 10
        return 0; // 子进程退出
    }

    // 父进程（pid=子进程ID）
    wait(nullptr); // 等待子进程结束
    count += 20;
    printf("父进程：count=%d, pid=%d\n", count, getpid());	// 20
    return 0;
}
```

#### （3）知识点拆解

> - `fork()` 是 Unix 系统调用，**创建一个新的子进程**，核心特性：
>   - 调用一次，返回两次：父进程返回子进程的 PID（正整数），子进程返回 0；
>   - 如果返回 -1，表示创建子进程失败（如系统资源不足），`perror` 打印错误原因。
> - 父子进程的关系：子进程是父进程的**副本**（复制父进程的代码、数据、栈、文件描述符等），但父子进程拥有**独立的内存空间**（修改变量互不影响）。
>
> - `wait(nullptr)`：父进程**阻塞等待**所有子进程退出（释放子进程资源，避免僵尸进程）；
>   - `nullptr` 表示不关心子进程的退出状态；
>   - 如果不调用 `wait()`，子进程可能先退出，成为 “僵尸进程”（占用系统资源）。
>
> - `getpid()`：获取当前进程的 PID（进程唯一标识）；

- `fork()`：调用一次，返回两次（父进程返回子进程 PID，子进程返回 0）
- <u>写时复制：子进程共享父进程内存，但修改时会拷贝一份（上述代码中，子进程修改 count 不影响父进程）</u>
  - `fork()` 后父子进程的内存是**写时复制（Copy-On-Write）** 的，修改变量时会复制一份独立的内存，互不干扰。

- `wait()`：父进程等待子进程结束，避免子进程变成僵尸进程
- 适用场景：CPU 密集型任务、高稳定性要求（如守护进程）

### 3. 多线程 vs 多进程 对比

| 特性     | 多线程                        | 多进程               |
| -------- | ----------------------------- | -------------------- |
| 资源占用 | 低（共享内存）                | 高（独立内存）       |
| 通信效率 | 高（直接访问共享变量）        | 低（需 IPC）         |
| 容错性   | 差（一个线程崩溃 = 进程崩溃） | 强（进程隔离）       |
| 切换开销 | 低（无需切换地址空间）        | 高（切换地址空间）   |
| 适用场景 | I/O 密集型、高并发            | CPU 密集型、高稳定性 |

## 六、线程池：复用线程提升性能

### 1. 核心概念

线程池预先创建一组线程，复用线程处理任务，避免频繁创建 / 销毁线程的开销。

- 组成：任务队列 + 工作线程 + 线程管理器
- 优势：降低资源消耗、提高响应速度、可控并发数

### 2. 完整代码实现（C++11）

```cpp
#include <vector>
#include <queue>
#include <thread>
#include <mutex>
#include <condition_variable>
#include <functional>
#include <future>
#include <iostream>

class ThreadPool {
public:
    // 构造：创建n个工作线程
    explicit ThreadPool(size_t num_threads) : stop(false) {
        for (size_t i = 0; i < num_threads; i++) {
            workers.emplace_back([this] {
                // 工作线程循环：取任务执行
                while (true) {
                    std::function<void()> task;
                    // 加锁取任务
                    {
                        std::unique_lock<std::mutex> lock(this->queue_mutex);
                        // 等待条件：有任务 或 线程池停止
                        this->condition.wait(lock, [this] {
                            return this->stop || !this->tasks.empty();
                        });
                        // 线程池停止且无任务，退出
                        if (this->stop && this->tasks.empty()) return;
                        // 取出任务
                        task = std::move(this->tasks.front());
                        this->tasks.pop();
                    }
                    // 执行任务（解锁后）
                    task();
                }
            });
        }
    }

    // 提交任务：返回future，可获取任务结果
    template<class F, class... Args>
    auto enqueue(F&& f, Args&&... args) 
        -> std::future<typename std::result_of<F(Args...)>::type> {
        using return_type = typename std::result_of<F(Args...)>::type;

        // 包装任务为std::function
        auto task = std::make_shared<std::packaged_task<return_type()>>(
            std::bind(std::forward<F>(f), std::forward<Args>(args)...)
        );

        std::future<return_type> res = task->get_future();
        // 加锁将任务加入队列
        {
            std::unique_lock<std::mutex> lock(queue_mutex);
            // 线程池停止时，禁止提交任务
            if (stop) throw std::runtime_error("enqueue on stopped ThreadPool");
            tasks.emplace([task]() { (*task)(); });
        }
        // 唤醒一个等待的工作线程
        condition.notify_one();
        return res;
    }

    // 析构：停止线程池，等待所有线程结束
    ~ThreadPool() {
        {
            std::unique_lock<std::mutex> lock(queue_mutex);
            stop = true;
        }
        // 唤醒所有工作线程
        condition.notify_all();
        // 等待所有线程结束
        for (std::thread& worker : workers) {
            worker.join();
        }
    }

    // 禁止拷贝
    ThreadPool(const ThreadPool&) = delete;
    ThreadPool& operator=(const ThreadPool&) = delete;

private:
    std::vector<std::thread> workers;       // 工作线程集合
    std::queue<std::function<void()>> tasks;// 任务队列
    std::mutex queue_mutex;                 // 队列互斥锁
    std::condition_variable condition;      // 条件变量（唤醒线程）
    bool stop;                               // 线程池停止标志
};

// 测试：计算平方和
int sum_square(int n) {
    int sum = 0;
    for (int i = 1; i <= n; i++) {
        sum += i*i;
    }
    return sum;
}

int main() {
    // 创建4个线程的线程池
    ThreadPool pool(4);

    // 提交3个任务
    auto f1 = pool.enqueue(sum_square, 10);
    auto f2 = pool.enqueue(sum_square, 20);
    auto f3 = pool.enqueue(sum_square, 30);

    // 获取结果
    std::cout << "1-10平方和：" << f1.get() << std::endl; // 385
    std::cout << "1-20平方和：" << f2.get() << std::endl; // 2870
    std::cout << "1-30平方和：" << f3.get() << std::endl; // 9455

    return 0;
}
```

### 3. 知识点拆解

- `std::packaged_task`：包装可调用对象，生成`std::future`获取结果
- `std::condition_variable`：用于线程间同步，`wait()`阻塞线程，`notify_one()`/`notify_all()`唤醒线程
- `std::unique_lock`：灵活的锁管理（可手动解锁），配合`condition_variable`使用
- 核心逻辑：
  1. 工作线程阻塞等待任务
  2. 主线程提交任务到队列，唤醒线程
  3. 线程执行任务后，继续等待下一个任务
  4. 析构时停止线程池，唤醒所有线程并等待结束

## 七、异步 I/O：非阻塞处理 I/O 事件

### 1. 核心概念

异步 I/O 是指：发起 I/O 操作后，线程立即返回，I/O 操作由内核在后台完成，完成后通过回调 / 通知告知线程。

- 非阻塞：线程不等待 I/O 完成
- 高效：CPU 可处理其他任务，避免浪费

### 2. 实现方式 1：std::async + std::future（C++11）

```cpp
#include <iostream>
#include <future>
#include <chrono>

// 模拟耗时I/O操作（如文件读取）
std::string read_file(const std::string& filename) {
    std::this_thread::sleep_for(std::chrono::seconds(2)); // 模拟I/O耗时
    return "文件内容：" + filename;
}

int main() {
    // 异步执行I/O操作（std::launch::async：创建新线程）
    std::future<std::string> fut = std::async(std::launch::async, read_file, "test.txt");

    // 主线程执行其他任务
    std::cout << "主线程处理其他工作...\n";
    std::this_thread::sleep_for(std::chrono::seconds(1));

    // 获取I/O结果（阻塞，直到异步操作完成）
    std::string result = fut.get();
    std::cout << "I/O完成：" << result << std::endl;

    return 0;
}
```

### 3. 实现方式 2：Boost.Asio（异步网络 I/O）

```cpp
#include <iostream>
#include <boost/asio.hpp>

using namespace boost::asio;
using ip::tcp;

int main() {
    // 1. 创建IO上下文（事件循环）
    io_context io;

    // 2. 解析服务器地址
    tcp::resolver resolver(io);
    auto endpoints = resolver.resolve("www.baidu.com", "80");

    // 3. 创建套接字，异步连接
    tcp::socket socket(io);
    async_connect(socket, endpoints, [&](const boost::system::error_code& ec, const tcp::endpoint&) {
        if (!ec) {
            std::cout << "连接成功\n";
            // 异步发送数据
            std::string request = "GET / HTTP/1.1\r\nHost: www.baidu.com\r\nConnection: close\r\n\r\n";
            async_write(socket, buffer(request), [&](const boost::system::error_code& ec, std::size_t bytes) {
                if (!ec) {
                    std::cout << "发送" << bytes << "字节\n";
                    // 异步读取响应
                    char buf[1024] = {0};
                    async_read(socket, buffer(buf), [&](const boost::system::error_code& ec, std::size_t bytes) {
                        if (!ec) {
                            std::cout << "收到响应：\n" << std::string(buf, bytes) << std::endl;
                        } else {
                            std::cerr << "读取失败：" << ec.message() << std::endl;
                        }
                    });
                } else {
                    std::cerr << "发送失败：" << ec.message() << std::endl;
                }
            });
        } else {
            std::cerr << "连接失败：" << ec.message() << std::endl;
        }
    });

    // 4. 运行事件循环（阻塞，直到所有异步操作完成）
    io.run();

    return 0;
}
```

### 4. 知识点拆解

- 异步 I/O 的核心：**回调函数**（I/O 完成后执行）
- `io_context`：Boost.Asio 的事件循环，管理所有异步操作
- 优势：单线程可处理大量异步 I/O 事件，配合线程池可充分利用多核
- 适用场景：高并发网络编程（如 Web 服务器、聊天系统）

## 八、总结

### 1. I/O 模型选择

- 简单场景：阻塞 I/O
- 高并发场景：非阻塞 I/O + epoll（Linux）/poll（跨平台）
- 极致性能：epoll + 边缘触发 + 非阻塞 I/O

### 2. 并发模型选择

- I/O 密集型：线程池 + 异步 I/O
- CPU 密集型：多进程（避免 GIL 限制，如 Python）/ 多线程（C++ 无 GIL）
- 高稳定性：多进程 + 进程监控

### 3. 核心思想

- 减少阻塞：用非阻塞 I/O + 多路复用，避免线程空等
- 复用资源：用线程池 / 进程池，避免频繁创建销毁
- 异步处理：用回调 / 未来，让 CPU 不等待 I/O