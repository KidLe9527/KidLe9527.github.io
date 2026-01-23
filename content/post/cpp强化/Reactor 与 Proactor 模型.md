## 一、Reactor 与 Proactor 模型核心概念

Reactor 和 Proactor 是高性能网络编程的两大**事件驱动 IO 模型**，核心差异在于 **IO 操作的发起 / 完成阶段由谁处理**：

| 维度         | Reactor（反应器）                              | Proactor（前摄器）                             |
| ------------ | ---------------------------------------------- | ---------------------------------------------- |
| 核心思想     | 监听事件（可读 / 可写），**应用层主动发起 IO** | 发起异步 IO，**内核完成 IO 后通知应用层**      |
| IO 阶段      | 事件就绪后，应用层执行 read/write（同步 IO）   | 应用层发起异步请求，内核完成 read/write 后回调 |
| 系统调用     | select/poll/epoll（Linux）                     | aio_*（Linux）/IOCP（Windows）                 |
| 编程复杂度   | 低（同步 IO 逻辑直观）                         | 高（异步回调、内存管理、错误处理复杂）         |
| 性能         | 高（减少线程上下文切换）                       | 更高（内核接管 IO，减少用户态 / 内核态切换）   |
| C++ 典型实现 | muduo/Netty（C++ 版）/Nginx 核心               | Asio（异步 IO）/libuv（部分支持）              |

> 可以用一个通俗的比喻理解：
>
> - **Reactor**：你（应用层）盯着快递柜（内核），看到快递到了（事件就绪），自己去取（主动发起 read/write）；
> - **Proactor**：你（应用层）告诉快递员（内核）“把快递放我家门口”（发起异步 IO），快递员放好后给你打电话（回调），你只需要确认签收（处理已完成的 IO 数据）。

------

## 二、Reactor 模型（C++ 实现）

### 1. 核心原理（以 Linux epoll 为例）

> Reactor 是**同步 IO + 事件多路复用**的组合模型：
>
> - 「同步 IO」：应用层必须**主动调用** `read/write` 完成数据拷贝（内核仅通知 “数据就绪”，不负责拷贝）；
> - 「事件多路复用」：通过 `select/poll/epoll` 监听多个 fd 的就绪事件（可读 / 可写 / 异常），避免单线程阻塞在单个 fd 上。

Reactor 模型分为 3 个核心组件：

- **Reactor 核心**：通过 epoll 监听 socket 事件（EPOLLIN/EPOLLOUT）；
- **事件处理器**：处理具体的 IO 事件（如读取数据、发送响应）；
- **事件分发器**：将就绪事件分发给对应的处理器。

### 2. 三种 Reactor 变体（C++ 场景）

| 变体                | 适用场景                | 缺点                     |
| ------------------- | ----------------------- | ------------------------ |
| 单 Reactor 单线程   | 连接数少、IO 操作快     | 单线程瓶颈，无法利用多核 |
| 单 Reactor 多线程   | 连接数中等、IO 计算密集 | Reactor 线程是瓶颈       |
| 主从 Reactor 多线程 | 高并发（如 Netty）      | 实现复杂，需处理线程同步 |

### 3. 单 Reactor 多线程（C++ 完整代码）

基于 epoll + 线程池实现，核心逻辑：

- Reactor 主线程：监听 epoll 事件，接受新连接，分发读 / 写事件；
- 工作线程池：处理具体的 IO 读写（如解析数据、业务逻辑）。

```cpp
#include <iostream>
#include <sys/epoll.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <unistd.h>
#include <fcntl.h>
#include <string.h>
#include <thread>
#include <mutex>
#include <queue>
#include <vector>
#include <functional>

// 线程池（处理IO任务）
class ThreadPool {
public:
    ThreadPool(int num) : stop(false) {
        // 创建num个工作线程
        for (int i = 0; i < num; ++i) {
            workers.emplace_back([this]() {
                while (true) {
                    std::function<void()> task;
                    // 加锁取任务
                    {
                        std::unique_lock<std::mutex> lock(mtx);
                        cond.wait(lock, [this]() {
                            return stop || !tasks.empty();
                        });
                    }
                    if (stop && tasks.empty()) return;
                    // 执行任务
                    task = std::move(tasks.front());
                    tasks.pop();
                    task();
                }
            });
        }
    }

    // 提交任务
    void submit(std::function<void()> task) {
        std::unique_lock<std::mutex> lock(mtx);
        tasks.emplace(std::move(task));
        cond.notify_one();
    }

    ~ThreadPool() {
        std::unique_lock<std::mutex> lock(mtx);
        stop = true;
        cond.notify_all();
        lock.unlock();
        // 等待所有线程退出
        for (auto& worker : workers) {
            worker.join();
        }
    }

private:
    std::vector<std::thread> workers;
    std::queue<std::function<void()>> tasks;
    std::mutex mtx;
    std::condition_variable cond;
    bool stop;
};

// Reactor 核心类
class Reactor {
public:
    Reactor(int threadNum) : epollFd(-1), listenFd(-1), pool(threadNum) {
        // 1. 创建epoll句柄
        epollFd = epoll_create1(0);
        if (epollFd < 0) {
            perror("epoll_create1 failed");
            exit(EXIT_FAILURE);
        }
    }

    // 初始化监听socket
    void initListenSocket(const char* ip, int port) {
        // 创建TCP socket
        listenFd = socket(AF_INET, SOCK_STREAM | SOCK_NONBLOCK, 0);
        if (listenFd < 0) {
            perror("socket failed");
            exit(EXIT_FAILURE);
        }

        // 绑定地址
        sockaddr_in addr{};
        addr.sin_family = AF_INET;
        addr.sin_port = htons(port);
        inet_pton(AF_INET, ip, &addr.sin_addr);
        if (bind(listenFd, (sockaddr*)&addr, sizeof(addr)) < 0) {
            perror("bind failed");
            exit(EXIT_FAILURE);
        }

        // 监听
        if (listen(listenFd, 1024) < 0) {
            perror("listen failed");
            exit(EXIT_FAILURE);
        }

        // 将listenFd加入epoll（监听EPOLLIN事件）
        epoll_event ev{};
        ev.events = EPOLLIN | EPOLLET; // 边缘触发（ET）
        ev.data.fd = listenFd;
        if (epoll_ctl(epollFd, EPOLL_CTL_ADD, listenFd, &ev) < 0) {
            perror("epoll_ctl add listenFd failed");
            exit(EXIT_FAILURE);
        }
    }

    // 启动Reactor循环
    void run() {
        const int MAX_EVENTS = 1024;
        epoll_event events[MAX_EVENTS];
        while (true) {
            // 等待事件就绪（阻塞）
            int nfds = epoll_wait(epollFd, events, MAX_EVENTS, -1);
            if (nfds < 0) {
                perror("epoll_wait failed");
                continue;
            }

            // 处理就绪事件
            for (int i = 0; i < nfds; ++i) {
                int fd = events[i].data.fd;
                // 1. 新连接事件
                if (fd == listenFd) {
                    handleAccept();
                }
                // 2. 可读事件
                else if (events[i].events & EPOLLIN) {
                    handleRead(fd);
                }
                // 3. 可写事件（可选）
                else if (events[i].events & EPOLLOUT) {
                    handleWrite(fd);
                }
            }
        }
    }

    ~Reactor() {
        close(listenFd);
        close(epollFd);
    }

private:
    // 设置非阻塞
    void setNonBlock(int fd) {
        int flags = fcntl(fd, F_GETFL, 0);
        fcntl(fd, F_SETFL, flags | O_NONBLOCK);
    }

    // 处理新连接
    void handleAccept() {
        sockaddr_in clientAddr{};
        socklen_t clientLen = sizeof(clientAddr);
        // 接受连接（非阻塞）
        int clientFd = accept4(listenFd, (sockaddr*)&clientAddr, &clientLen, SOCK_NONBLOCK);
        if (clientFd < 0) {
            perror("accept4 failed");
            return;
        }

        // 将clientFd加入epoll（监听可读事件）
        epoll_event ev{};
        ev.events = EPOLLIN | EPOLLET | EPOLLONESHOT; // EPOLLONESHOT防止多线程同时处理
        ev.data.fd = clientFd;
        if (epoll_ctl(epollFd, EPOLL_CTL_ADD, clientFd, &ev) < 0) {
            perror("epoll_ctl add clientFd failed");
            close(clientFd);
            return;
        }

        std::cout << "New connection: " << inet_ntoa(clientAddr.sin_addr) 
                  << ":" << ntohs(clientAddr.sin_port) << std::endl;
    }

    // 处理可读事件（交给线程池）
    void handleRead(int fd) {
        pool.submit([this, fd]() {
            char buf[1024] = {0};
            ssize_t n = read(fd, buf, sizeof(buf) - 1);
            if (n < 0) {
                perror("read failed");
                closeFd(fd);
                return;
            }
            if (n == 0) { // 客户端关闭连接
                std::cout << "Client closed: fd=" << fd << std::endl;
                closeFd(fd);
                return;
            }

            // 业务逻辑：回显数据
            std::cout << "Read from fd=" << fd << ": " << buf << std::endl;
            std::string resp = "Server echo: " + std::string(buf);
            
            // 重新监听可写事件（准备发送响应）
            epoll_event ev{};
            ev.events = EPOLLOUT | EPOLLET | EPOLLONESHOT;
            ev.data.fd = fd;
            epoll_ctl(epollFd, EPOLL_CTL_MOD, fd, &ev);
        });
    }

    // 处理可写事件
    void handleWrite(int fd) {
        pool.submit([this, fd]() {
            std::string resp = "Server response\n";
            ssize_t n = write(fd, resp.c_str(), resp.size());
            if (n < 0) {
                perror("write failed");
                closeFd(fd);
                return;
            }

            // 重新监听可读事件（等待下一次数据）
            epoll_event ev{};
            ev.events = EPOLLIN | EPOLLET | EPOLLONESHOT;
            ev.data.fd = fd;
            epoll_ctl(epollFd, EPOLL_CTL_MOD, fd, &ev);
        });
    }

    // 关闭fd并从epoll移除
    void closeFd(int fd) {
        epoll_ctl(epollFd, EPOLL_CTL_DEL, fd, nullptr);
        close(fd);
    }

private:
    int epollFd;          // epoll句柄
    int listenFd;         // 监听socket
    ThreadPool pool;      // 工作线程池
};

int main() {
    // 创建Reactor（4个工作线程）
    Reactor reactor(4);
    // 初始化监听端口（本地9090）
    reactor.initListenSocket("127.0.0.1", 9090);
    // 启动Reactor循环
    reactor.run();
    return 0;
}
```

### 4. 代码关键说明

- **非阻塞 IO**：`socket` + `SOCK_NONBLOCK` + `accept4` 确保无阻塞等待；
- **边缘触发（ET）**：`EPOLLET` 减少 epoll 事件触发次数，提升性能；
- **EPOLLONESHOT**：防止多个线程同时处理同一个 fd 的事件；
- **线程池**：将 IO 读写 / 业务逻辑交给工作线程，Reactor 主线程仅负责事件分发；
- **编译运行**：需链接 pthread 库 `g++ reactor.cpp -o reactor -lpthread && ./reactor`。

------

## 三、Proactor 模型（C++ 实现）

### 1. 核心原理（Linux aio 为例）

> Proactor 是**异步 IO + 回调通知**的组合模型：
>
> - 「异步 IO」：应用层发起 IO 请求后立即返回，内核负责完成整个 IO 流程（包括数据从网卡 / 磁盘拷贝到用户缓冲区）；
> - 「回调通知」：内核完成 IO 后，通过信号 / 回调函数通知应用层 “IO 已完成”，应用层仅处理结果。

Proactor 模型的核心是 **异步 IO**：

1. 应用层调用 `aio_read/aio_write` 发起异步 IO 请求；
2. 内核完成数据拷贝（用户态 ↔ 内核态）后，通过信号 / 回调通知应用层；
3. 应用层仅处理 IO 完成后的逻辑（无需主动发起 read/write）。

### 2. Linux aio 限制

- Linux 原生 aio 仅支持 O_DIRECT（绕过页缓存），对普通文件 / 套接字支持有限；
- 生产环境常用 `libaio`（异步 IO 库）或 Boost.Asio（跨平台异步 IO）。

### 3. Proactor 实现（Boost.Asio 示例）

Boost.Asio 是 C++ 异步 IO 标准库（C++17 纳入 std::asio），封装了 epoll/IOCP，实现 Proactor 模型：

```cpp
#include <iostream>
#include <boost/asio.hpp>
#include <memory>
#include <string>

using namespace boost::asio;
using ip::tcp;

// 会话类：处理单个客户端连接
class Session : public std::enable_shared_from_this<Session> {
public:
    Session(tcp::socket socket) : socket_(std::move(socket)) {}

    // 启动异步读写
    void start() {
        do_read(); // 发起异步读
    }

private:
    // 异步读（Proactor核心：内核完成读后回调）
    void do_read() {
        auto self = shared_from_this();
        // 发起异步读请求（内核接管IO）
        async_read_until(socket_, buffer_, "\n",
            [this, self](boost::system::error_code ec, std::size_t length) {
                if (!ec) {
                    // IO完成：处理数据
                    std::string data(buffer_.data(), length);
                    std::cout << "Read from client: " << data;
                    do_write(data); // 发起异步写
                } else if (ec != boost::asio::error::eof) {
                    std::cerr << "Read error: " << ec.message() << std::endl;
                }
                // eof：客户端关闭连接，自动释放Session
            });
    }

    // 异步写（Proactor核心：内核完成写后回调）
    void do_write(const std::string& data) {
        auto self = shared_from_this();
        std::string resp = "Server echo: " + data;
        // 发起异步写请求
        async_write(socket_, buffer(resp),
            [this, self](boost::system::error_code ec, std::size_t /*length*/) {
                if (!ec) {
                    do_read(); // 继续监听读事件
                } else {
                    std::cerr << "Write error: " << ec.message() << std::endl;
                }
            });
    }

private:
    tcp::socket socket_;       // 客户端socket
    boost::asio::streambuf buffer_; // 读缓冲区
};

// 服务器类：监听连接并创建会话
class Server {
public:
    Server(io_context& io_context, short port)
        : acceptor_(io_context, tcp::endpoint(tcp::v4(), port)) {
        do_accept(); // 发起异步接受连接
    }

private:
    // 异步接受连接
    void do_accept() {
        // 发起异步accept（内核接管）
        acceptor_.async_accept(
            [this](boost::system::error_code ec, tcp::socket socket) {
                if (!ec) {
                    // 新连接：创建会话并启动
                    std::make_shared<Session>(std::move(socket))->start();
                } else {
                    std::cerr << "Accept error: " << ec.message() << std::endl;
                }
                // 继续监听新连接
                do_accept();
            });
    }

private:
    tcp::acceptor acceptor_; // 监听acceptor
};

int main() {
    try {
        // io_context是Proactor核心（事件循环）
        boost::asio::io_context io_context;
        // 启动服务器（监听9090端口）
        Server server(io_context, 9090);
        // 运行事件循环（处理异步IO回调）
        io_context.run();
    } catch (std::exception& e) {
        std::cerr << "Exception: " << e.what() << std::endl;
    }
    return 0;
}
```

### 4. 代码关键说明

- **Proactor 核心**：`async_read_until/async_write` 发起异步 IO，内核完成后回调 lambda 函数；
- **io_context**：Boost.Asio 的事件循环核心，等价于 Reactor 的 epoll 循环；
- **智能指针**：`enable_shared_from_this` 确保回调时 Session 对象不被释放；
- **编译运行**：需安装 Boost 库 `g++ proactor.cpp -o proactor -lboost_system -lpthread && ./proactor`。

------

## 四、Reactor vs Proactor（C++ 实践建议）

| 场景                | 推荐模型 | 技术选型                        |
| ------------------- | -------- | ------------------------------- |
| 高并发 TCP 服务     | Reactor  | epoll + 线程池（muduo 框架）    |
| 海量小数据包 IO     | Proactor | Boost.Asio/std::asio            |
| Windows 平台        | Proactor | IOCP（Asio 封装）               |
| 快速开发 / 低复杂度 | Reactor  | epoll + 简单线程池              |
| 极致性能 / 低延迟   | Proactor | libaio（Linux）/IOCP（Windows） |

### 五、核心注意事项

1. **Reactor 注意点**：
   - 边缘触发（ET）需循环读取数据，直到 `EAGAIN`；
   - `EPOLLONESHOT` 需重新设置事件掩码；
   - 线程池需控制大小，避免上下文切换过载。
2. **Proactor 注意点**：
   - 异步回调需注意内存管理（智能指针避免悬空引用）；
   - Linux 原生 aio 不支持套接字，优先用 Asio；
   - 异步 IO 错误处理需覆盖网络断开、超时等场景。
3. **C++ 特性利用**：
   - C++11 以上的 `std::thread/std::function` 简化线程池 / 回调；
   - C++20 的 `coroutine` 可简化异步回调嵌套（Asio 已支持协程）；
   - RAII 管理文件描述符 /epoll 句柄，避免资源泄漏