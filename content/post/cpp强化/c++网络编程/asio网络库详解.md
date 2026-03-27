# ASIO 详细使用指南：语法、核心概念与代码示例

ASIO（Boost.Asio 或独立的 Asio 库）是一个跨平台的 C++ 库，专注于**异步 I/O** 和**网络编程**，核心设计目标是通过异步模型高效处理网络通信、串口、定时器等 I/O 操作。本文将从核心概念、语法规则、常用场景代码示例三个维度详细讲解 ASIO 的使用。

> **ASIO 封装了异步 I/O 所需的所有底层细节（多路复用、线程调度、系统调用、平台差异），并提供一套一致、易用的异步编程接口**，让开发者可以专注于业务逻辑，而非底层 I/O 机制的实现。

## 一、核心前置知识

### 1.1 版本选择

- **Boost.Asio**：依赖 Boost 库，功能最全，兼容旧版 C++（C++11+）。
- **Standalone Asio**：独立版本，无需 Boost，仅依赖 C++11+ 标准库，API 与 Boost.Asio 几乎一致。
- 本文示例基于 **Standalone Asio**（C++17），语法与 Boost.Asio 通用（仅头文件路径不同）。

### 1.2 核心概念

| 概念           | 作用                                                         |
| -------------- | ------------------------------------------------------------ |
| `io_context`   | ASIO 的核心调度器，负责管理异步操作的事件循环（必须运行才能处理异步任务）。 |
| `strand`       | 线程安全的执行器，保证异步操作串行执行（避免多线程竞争）。   |
| `handler`      | 异步操作完成后的回调函数（可通过 lambda、函数对象实现）。    |
| `error_code`   | 错误处理机制，替代异常（ASIO 支持异常 /error_code 两种错误处理方式）。 |
| `endpoint`     | 网络端点（IP + 端口），如 `tcp::endpoint`。                  |
| `socket`       | 网络套接字（如 `tcp::socket`、`udp::socket`），封装底层网络操作。 |
| `async_*` 函数 | 异步操作入口（如 `async_read`、`async_write`、`async_accept`），无阻塞。 |
| `buffer`       | 内存缓冲区封装（`asio::buffer`），用于读写数据。             |

> * 如果需要测试成功场景，可以先启动一个简单的 TCP 服务（如用 `nc`）：
>
> ```cpp
> nc -l 127.0.0.1 9999
> ```

## 二、基础语法规则

### 2.1 头文件引入

- Standalone Asio：

  ```cpp
  #include <asio.hpp>
  ```

- Boost.Asio：

  ```cpp
  #include <boost/asio.hpp>
  // 命名空间替换为 boost::asio
  ```

### 2.2 命名空间

ASIO 核心组件位于 `asio` 命名空间，网络组件在 `asio::ip` 子命名空间：

```cpp
using namespace asio;          // 核心命名空间
using namespace asio::ip;      // 网络相关（tcp/udp/address 等）
```

### 2.3 错误处理方式

ASIO 提供两种错误处理模式：

#### 模式 1：异常抛出（默认）

异步操作失败时，会抛出 `system_error` 异常：

```cpp
try {
  tcp::socket sock(io_context);
  sock.connect(tcp::endpoint(address::from_string("127.0.0.1"), 8080));
} catch (const system_error& e) {
  std::cerr << "错误：" << e.what() << " (" << e.code() << ")" << std::endl;
}
```

#### 模式 2：error_code 传参（推荐）

显式传入 `error_code` 对象接收错误，避免异常：

```cpp
error_code ec;
tcp::socket sock(io_context);
sock.connect(tcp::endpoint(address::from_string("127.0.0.1"), 8080), ec);
if (ec) {
  std::cerr << "错误：" << ec.message() << " (" << ec.value() << ")" << std::endl;
}
```

### 2.4 缓冲区（buffer）

ASIO 操作数据时必须通过 `asio::buffer` 封装内存，支持：

- 动态缓冲区：`std::vector<char>`、`std::string`
- 静态缓冲区：数组、指针 + 长度

```cpp
// 可读可写缓冲区
std::vector<char> buf(1024);
asio::buffer(buf);          // 整个缓冲区
asio::buffer(buf, 512);     // 前512字节

// 只读缓冲区
std::string data = "hello";
asio::buffer(data);         // 只读

// 原生指针
char raw_buf[1024];
asio::buffer(raw_buf, sizeof(raw_buf));
```

### 2.5 io_context 运行

`io_context` 是异步操作的 “事件循环”，必须调用 `run()`/`run_one()`/`poll()` 才能处理异步任务：

```cpp
asio::io_context io_context;

// 方式1：运行直到所有异步操作完成
io_context.run();

// 方式2：运行1个异步任务后返回
io_context.run_one();

// 方式3：处理所有就绪的异步任务，不阻塞
io_context.poll();

// 重置（run() 退出后需重置才能再次运行）
io_context.reset();
```

## 三、常用场景代码示例

### 场景 1：定时器（异步 / 同步）

定时器是 ASIO 最基础的异步操作，用于延迟执行任务。

#### 同步定时器（阻塞）

```cpp
#include <asio.hpp>
#include <iostream>

int main() {
  asio::io_context io;

  // 同步定时器：阻塞3秒
  asio::steady_timer timer(io, asio::chrono::seconds(3));
  std::cout << "等待3秒..." << std::endl;
  
  // 阻塞等待定时器到期
  timer.wait();

  std::cout << "定时器到期！" << std::endl;
  return 0;
}
```

#### 异步定时器（非阻塞）

```cpp
#include <asio.hpp>
#include <iostream>

void timer_handler(const asio::error_code& ec) {
  if (ec) {
    std::cerr << "定时器错误：" << ec.message() << std::endl;
    return;
  }
  std::cout << "异步定时器到期！" << std::endl;
}

int main() {
  asio::io_context io;

  // 异步定时器：非阻塞
  asio::steady_timer timer(io, asio::chrono::seconds(3));
  std::cout << "异步等待3秒..." << std::endl;
  
  // 发起异步等待，完成后调用 timer_handler
  timer.async_wait(&timer_handler);

  // 运行事件循环（阻塞，直到异步操作完成）
  io.run();

  return 0;
}
```

#### 异步定时器（Lambda 回调）

更简洁的写法（推荐）：

```cpp
#include <asio.hpp>
#include <iostream>

int main() {
  asio::io_context io;

  asio::steady_timer timer(io, asio::chrono::seconds(3));
  std::cout << "Lambda 异步等待3秒..." << std::endl;

  // Lambda 作为回调（捕获外部变量需注意生命周期）
  timer.async_wait([](const asio::error_code& ec) {
    if (!ec) {
      std::cout << "Lambda 定时器到期！" << std::endl;
    }
  });

  io.run();
  return 0;
}
```

### 场景 2：TCP 服务器（异步）

实现一个简单的异步 TCP 服务器，支持接收客户端连接并回显数据。

```cpp
#include <asio.hpp>
#include <iostream>
#include <memory>
#include <string>

using namespace asio;
using namespace asio::ip;

// 每个客户端连接的会话（用智能指针管理生命周期）
class TcpSession : public std::enable_shared_from_this<TcpSession> {
public:
  explicit TcpSession(tcp::socket socket) : socket_(std::move(socket)) {}

  // 启动会话：异步读取客户端数据
  void start() {
    do_read();
  }

private:
  // 异步读取数据
  void do_read() {
    auto self = shared_from_this(); // 延长生命周期，避免回调时析构
    socket_.async_read_some(
      buffer(data_, max_length),
      [this, self](const error_code& ec, std::size_t length) {
        if (!ec) {
          // 读取成功，异步回显数据给客户端
          std::cout << "收到客户端数据：" << std::string(data_, length) << std::endl;
          do_write(length);
        } else if (ec != error::eof) {
          std::cerr << "读取错误：" << ec.message() << std::endl;
        }
      }
    );
  }

  // 异步回显数据
  void do_write(std::size_t length) {
    auto self = shared_from_this();
    async_write(
      socket_,
      buffer(data_, length),
      [this, self](const error_code& ec, std::size_t /*length*/) {
        if (!ec) {
          // 回显完成，继续读取下一批数据
          do_read();
        } else {
          std::cerr << "写入错误：" << ec.message() << std::endl;
        }
      }
    );
  }

  tcp::socket socket_;          // 客户端套接字
  enum { max_length = 1024 };   // 缓冲区大小
  char data_[max_length];       // 数据缓冲区
};

// TCP 服务器类
class TcpServer {
public:
  TcpServer(io_context& io, const tcp::endpoint& endpoint)
    : acceptor_(io, endpoint) {
    do_accept(); // 开始异步接受连接
  }

private:
  // 异步接受客户端连接
  void do_accept() {
    acceptor_.async_accept(
      [this](const error_code& ec, tcp::socket socket) {
        if (!ec) {
          // 连接成功，创建会话并启动
          std::make_shared<TcpSession>(std::move(socket))->start();
        } else {
          std::cerr << "接受连接错误：" << ec.message() << std::endl;
        }

        // 继续接受下一个连接
        do_accept();
      }
    );
  }

  tcp::acceptor acceptor_; // 服务器监听器
};

int main() {
  try {
    io_context io;
    // 绑定 127.0.0.1:8080
    tcp::endpoint endpoint(address::from_string("127.0.0.1"), 8080);
    TcpServer server(io, endpoint);

    std::cout << "TCP 服务器启动：127.0.0.1:8080" << std::endl;
    io.run(); // 运行事件循环
  } catch (const std::exception& e) {
    std::cerr << "服务器异常：" << e.what() << std::endl;
  }

  return 0;
}
```

### 场景 3：TCP 客户端（异步）

实现一个异步 TCP 客户端，连接服务器并发送 / 接收数据。

```cpp
#include <asio.hpp>
#include <iostream>
#include <memory>
#include <string>

using namespace asio;
using namespace asio::ip;

class TcpClient {
public:
  TcpClient(io_context& io, const std::string& host, const std::string& port)
    : io_(io), resolver_(io), socket_(io) {
    // 解析服务器地址（支持域名/IP）
    resolver_.async_resolve(
      host, port,
      [this](const error_code& ec, tcp::resolver::results_type endpoints) {
        if (!ec) {
          do_connect(endpoints); // 解析成功，异步连接
        } else {
          std::cerr << "解析地址错误：" << ec.message() << std::endl;
        }
      }
    );
  }

  // 发送数据到服务器
  void send(const std::string& message) {
    // 保证异步操作串行执行（strand 线程安全）
    asio::post(strand_, [this, message]() {
      bool write_in_progress = !write_msgs_.empty();
      write_msgs_.push_back(message);
      if (!write_in_progress) {
        do_write();
      }
    });
  }

private:
  // 异步连接服务器
  void do_connect(const tcp::resolver::results_type& endpoints) {
    async_connect(
      socket_, endpoints,
      [this](const error_code& ec, const tcp::endpoint& /*endpoint*/) {
        if (!ec) {
          std::cout << "连接服务器成功！" << std::endl;
          do_read(); // 连接成功，开始异步读取
        } else {
          std::cerr << "连接错误：" << ec.message() << std::endl;
        }
      }
    );
  }

  // 异步读取服务器响应
  void do_read() {
    socket_.async_read_some(
      buffer(data_, max_length),
      [this](const error_code& ec, std::size_t length) {
        if (!ec) {
          std::cout << "收到服务器响应：" << std::string(data_, length) << std::endl;
          do_read(); // 继续读取
        } else if (ec != error::eof) {
          std::cerr << "读取错误：" << ec.message() << std::endl;
        }
      }
    );
  }

  // 异步发送数据
  void do_write() {
    async_write(
      socket_,
      buffer(write_msgs_.front()),
      [this](const error_code& ec, std::size_t /*length*/) {
        if (!ec) {
          write_msgs_.pop_front();
          if (!write_msgs_.empty()) {
            do_write(); // 继续发送下一条
          }
        } else {
          std::cerr << "发送错误：" << ec.message() << std::endl;
        }
      }
    );
  }

  io_context& io_;
  tcp::resolver resolver_;    // 地址解析器
  tcp::socket socket_;        // 客户端套接字
  asio::strand<io_context::executor_type> strand_ {io_.get_executor()}; // 串行执行器
  enum { max_length = 1024 };
  char data_[max_length];
  std::deque<std::string> write_msgs_; // 待发送消息队列
};

int main() {
  try {
    io_context io;
    TcpClient client(io, "127.0.0.1", "8080");

    // 发送测试数据
    client.send("Hello ASIO Server!");

    // 运行事件循环（可在另一个线程运行，避免阻塞主线程）
    std::thread t([&io]() { io.run(); });

    // 主线程输入数据发送
    std::string input;
    while (std::getline(std::cin, input)) {
      if (input == "exit") break;
      client.send(input);
    }

    // 停止事件循环
    io.stop();
    t.join();
  } catch (const std::exception& e) {
    std::cerr << "客户端异常：" << e.what() << std::endl;
  }

  return 0;
}
```

### 场景 4：UDP 通信（异步）

UDP 是无连接协议，无需建立连接，直接发送 / 接收数据报。

#### UDP 服务器

```cpp
#include <asio.hpp>
#include <iostream>

using namespace asio;
using namespace asio::ip;

int main() {
  try {
    io_context io;
    udp::socket socket(io, udp::endpoint(udp::v4(), 8080));

    std::cout << "UDP 服务器启动：0.0.0.0:8080" << std::endl;

    char data[1024];
    udp::endpoint remote_endpoint;

    // 异步接收数据报
    socket.async_receive_from(
      buffer(data, 1024), remote_endpoint,
      [&](const error_code& ec, std::size_t length) {
        if (!ec) {
          std::cout << "收到客户端 [" << remote_endpoint << "] 数据：" 
                    << std::string(data, length) << std::endl;

          // 异步回显数据
          socket.async_send_to(
            buffer(data, length), remote_endpoint,
            [&](const error_code& ec, std::size_t /*length*/) {
              if (ec) {
                std::cerr << "发送错误：" << ec.message() << std::endl;
              }
            }
          );
        }
      }
    );

    io.run();
  } catch (const std::exception& e) {
    std::cerr << "服务器异常：" << e.what() << std::endl;
  }

  return 0;
}
```

#### UDP 客户端

```cpp
#include <asio.hpp>
#include <iostream>

using namespace asio;
using namespace asio::ip;

int main() {
  try {
    io_context io;
    udp::socket socket(io);
    socket.open(udp::v4());

    // 服务器地址
    udp::resolver resolver(io);
    udp::endpoint server_endpoint = *resolver.resolve(udp::v4(), "127.0.0.1", "8080").begin();

    // 发送数据
    std::string message = "Hello UDP Server!";
    socket.send_to(buffer(message), server_endpoint);
    std::cout << "发送数据：" << message << std::endl;

    // 接收响应
    char data[1024];
    udp::endpoint remote_endpoint;
    std::size_t length = socket.receive_from(buffer(data, 1024), remote_endpoint);
    std::cout << "收到服务器响应：" << std::string(data, length) << std::endl;
  } catch (const std::exception& e) {
    std::cerr << "客户端异常：" << e.what() << std::endl;
  }

  return 0;
}
```

## 四、关键注意事项

1. **生命周期管理**：异步回调中，必须保证回调所依赖的对象（如 socket、buffer）生命周期长于异步操作，推荐使用 `std::shared_ptr` + `shared_from_this()`。

2. **线程安全**：`io_context` 本身线程安全，但 socket/acceptor 等对象**非线程安全**，需通过 `strand` 保证串行访问。

3. **资源释放**：关闭 socket 时需先取消未完成的异步操作（`socket.cancel()`），避免回调触发时对象已析构。

4. **错误处理**：所有异步操作必须处理 `error_code`，尤其是 `error::eof`（连接关闭）、`error::operation_aborted`（操作取消）等常见错误。

5. **多线程运行 io_context**：可创建多个线程调用 `io_context.run()`，提升并发处理能力：

   ```cpp
   asio::io_context io;
   // 启动4个线程运行事件循环
   std::vector<std::thread> threads;
   for (int i = 0; i < 4; ++i) {
     threads.emplace_back([&io]() { io.run(); });
   }
   // 等待所有线程退出
   for (auto& t : threads) t.join();
   ```

   

## 五、编译与运行

### 编译命令（Standalone Asio）

```bash
# 依赖：需安装 asio 库（Ubuntu：apt install libasio-dev）
g++ -std=c++17 tcp_server.cpp -o tcp_server -lasio -pthread
g++ -std=c++17 tcp_client.cpp -o tcp_client -lasio -pthread
```

### 运行流程

1. 先启动服务器：`./tcp_server`
2. 再启动客户端：`./tcp_client`
3. 客户端输入数据，服务器会回显，输入 `exit` 退出客户端。

## 总结

ASIO 的核心是**异步回调模型**，通过 `io_context` 管理事件循环，`async_*` 函数发起非阻塞操作，回调函数处理结果。掌握 `io_context`、`strand`、`buffer`、错误处理四大核心，即可实现高效的异步网络程序。ASIO 还支持串口、信号、文件 I/O 等操作，API 设计一致，可举一反三。





## -------------------------------------------

## 配置asio环境变量 🔥

```
# 把路径写入 zsh 配置文件（永久生效）
echo 'export CPATH="/opt/homebrew/include:$CPATH"' >> ~/.zshrc

# 刷新配置，让修改立刻生效
source ~/.zshrc

# 验证是否生效 -- 输出里能看到 /opt/homebrew/include 就说明成功。注意，vscode终端是独立进程，要重启～
echo $CPATH
```



### 1. `io_context`：异步操作的 “事件循环核心”

#### 核心原理

`io_context` 的本质是**事件循环 + 任务队列**： 上下文

1. 异步操作（如 `async_wait`、`async_read`）发起后，不会立即执行处理逻辑，而是由操作系统内核接管；
2. 当异步操作完成（如定时器超时、数据读取完成），对应的**完成处理函数（handler）** 会被加入 `io_context` 的任务队列；
3. 调用 `io_context` 的 `run()`/`run_one()`/`poll()` 等方法时，事件循环会从队列中取出 handler 并执行；
4. 若任务队列为空且无 “未完成的工作”，`run()` 会立即返回，事件循环终止。

#### 核心作用

ASIO 所有异步操作的 “总调度中心”，负责：

- 注册异步操作（如 `async_read`、`async_wait`）；
- 监听底层 I/O 事件（如 socket 可读、定时器到期）；
- 触发异步操作完成后的回调（handler）。
- 必须调用 `run()`/`poll()` 才能让事件循环生效，否则异步操作永远不会执行。

#### 关键使用规则

| 方法             | 作用                                                         |
| ---------------- | ------------------------------------------------------------ |
| `run()`          | 阻塞运行，直到所有已注册的异步操作完成 / 取消，退出后需 `reset()` 才能复用 |
| `run_one()`      | 仅处理**一个**就绪的异步操作，处理完立即返回（适合手动控制循环） |
| `poll()`         | 非阻塞处理**所有已就绪**的异步操作，无就绪操作则立即返回     |
| `reset()`        | 重置状态（`run()` 退出后必须调用，否则再次 `run()` 会直接返回） |
| `stop()`         | 强制停止事件循环（未完成的异步操作会触发 `error::operation_aborted`） |
| `restart()`      | 重置 `io_context` 状态（清除 stop 标记），使 `run()` 可再次调用。 |
| `get_executor()` | 返回关联的执行器（用于创建 `work_guard`）。                  |

> **`run()` 启动事件循环后，会立即处理已就绪的 handler，同时持续等待新的异步操作完成，直到无待处理任务且无未完成工作（无未完成异步操作 + 无 `work_guard`）时才退出**。

#### 代码示例

```cpp
#include <asio.hpp>
#include <iostream>

int main() {
    asio::io_context io; // 1. 创建核心调度器

    // 注册异步定时器（依赖io_context）
    asio::steady_timer timer(io, asio::chrono::seconds(2)); // 3. 创建定时器，设置2秒后到期

    // 异步等待定时器到期,并指定回调函数. 发起异步操作，传入回调函数handler
  	/*调用该函数后不会阻塞当前线程，当定时器达到指定时间（或被取消）时，
  	Asio 会在 io_context 的事件循环中触发你传入的回调函数，完成定时逻辑的异步执行。*/
    timer.async_wait([](const asio::error_code& ec) {
        if (!ec) std::cout << "定时器到期（io_context驱动）" << std::endl;
        else std::cout << "定时器被取消或出错: " << ec.message() << std::endl;
    });

    std::cout << "启动io_context事件循环..." << std::endl;
    io.run(); // 2. 运行事件循环（阻塞，直到定时器回调执行完成） // 驱动异步操作，在此期间，io_context会监听和调度所有注册的异步操作，直到异步操作全部完成，才会退出run()
    std::cout << "io_context退出" << std::endl;

    // 若需再次运行，必须reset
    //io.reset();
    /*
    asio::io_context 并没有 reset() 成员函数（这是旧版 Boost.Asio 的接口，Standalone Asio 已移除），把代码里的 io.reset(); 删掉即可。*/
    // 重新注册异步操作...
    return 0;
}
```

#### 避坑点

- 不要忘记调用 `run()`：新手常犯错误是注册了异步操作但不运行 `io_context`，导致回调永远不执行；
- 多线程运行：可创建多个线程调用 `io.run()`，提升并发能力（ASIO 自动分发回调到空闲线程）；
- `run()` 退出条件：所有异步操作完成 / 取消，或调用 `stop()`。



### 2. `strand`：异步操作的 “串行执行锁”

#### 核心作用

ASIO 的 `socket`/`acceptor` 等对象**非线程安全**，若多个线程同时操作同一个 socket（如同时 `async_read` 和 `async_write`），会导致未定义行为。

`strand` 是 “串行执行器”，保证：**注册到同一个 strand 的异步操作，其回调永远串行执行**（无需手动加锁）。

#### 关键使用规则

- 绑定方式：将 `strand` 关联到异步操作的 “执行上下文”（通过 `bind_executor` 或直接传递）；
- 核心逻辑：即使 `io_context` 有多个线程运行，`strand` 也会让回调按顺序执行，避免竞争。

#### 代码示例

```cpp
#include <asio.hpp>
#include <iostream>
#include <thread>

int main() {
    asio::io_context io;    // 创建io_context,作为异步操作的核心调度器
    // 1. 创建strand（绑定到io_context的执行器）
    asio::strand<asio::io_context::executor_type> strand(io.get_executor());    // 这是一个保证串行执行的执行器，executor_type是io_context的执行器类型

    // 2. 注册多个异步操作到strand，保证串行执行
    for (int i = 0; i < 3; ++i) {
        // 用bind_executor将操作绑定到strand
        asio::post(strand, [i]() { // post用于提交一个任务到io_context。 参数strand保证任务在同一时间只有一个在执行
            std::cout << "操作" << i << "执行（线程ID：" 
                      << std::this_thread::get_id() << "）" << std::endl;
            // 模拟耗时操作
            std::this_thread::sleep_for(asio::chrono::seconds(1));
        });
    }

    // 启动2个线程运行io_context（模拟多线程环境）---> 串行执行
    std::thread t1([&io]() { io.run(); });  // 运行io_context事件循环，这是阻塞调用
    std::thread t2([&io]() { io.run(); });	// 尽管有两个线程，但是同一时间只有一个任务执行

    t1.join();
    t2.join();
    return 0;
}
```

#### 输出（串行执行，即使多线程）

```plaintext
操作0执行（线程ID：1407092800）
操作1执行（线程ID：1407093632）
操作2执行（线程ID：1407092800）
```

#### 避坑点

- 每个非线程安全对象（如 socket）对应一个 `strand`，而非全局共享；
- 异步操作的**整个回调链**都要绑定到 `strand`（如 `async_read` 回调中触发 `async_write`，需都通过 strand）。



### 3. `handler`：异步操作的 “结果处理函数”

#### 核心作用

异步操作（如 `async_read`、`async_accept`）完成后，ASIO 会调用的 “回调函数”，用于处理操作结果（成功 / 失败、数据等）。

#### 关键使用规则

- 签名要求：必须匹配异步操作的回调签名（如定时器回调是 `void(const error_code&)`，`async_read` 回调是 `void(const error_code&, size_t)`）；
- 实现方式：Lambda（推荐）、函数指针、函数对象；
- 生命周期：回调执行前，handler 依赖的对象（如 socket、buffer）必须存活（推荐用 `shared_ptr` + `shared_from_this()`）。

#### 代码示例

```cpp
#include <asio.hpp> // 本段代码辨析多种回调函数的使用方法
#include <iostream>

// 方式1：函数指针（简单场景）
void timer_handler(const asio::error_code& ec) {
    if (ec) {
        std::cerr << "定时器错误：" << ec.message() << std::endl;
        return;
    }
    std::cout << "函数指针回调：定时器到期" << std::endl;
}

// 方式2：函数对象（复杂场景，可携带状态）
struct WriteHandler {
    std::string data;
    void operator()(const asio::error_code& ec, size_t len) {
        if (!ec) {
            std::cout << "函数对象回调：发送了" << len << "字节，数据：" << data << std::endl;
        }
    }
};

int main() {
    asio::io_context io;    // 创建io_context,作为异步操作的核心调度器

    // 方式1：函数指针
    asio::steady_timer timer1(io, asio::chrono::seconds(1));    // 创建定时器，设置1秒后到期
    timer1.async_wait(&timer_handler);  // 注册异步等待，传入函数指针作为回调

    // 方式2：Lambda（推荐，简洁+可捕获变量）
    asio::steady_timer timer2(io, asio::chrono::seconds(2));    // 创建定时器，设置2秒后到期
    timer2.async_wait([](const asio::error_code& ec) {
        if (!ec) std::cout << "Lambda回调：定时器到期" << std::endl;
    }); // 注册异步等待，传入Lambda表达式作为回调

    // 方式3：函数对象（携带状态）
    asio::ip::tcp::socket sock(io);
    WriteHandler handler{"hello"};
    asio::async_write(sock, asio::buffer(handler.data), handler);
    /* 将函数对象中携带的 data（"hello"）封装为 Asio 可识别的缓冲区（内存视图），表示要写入的数据；
    buffer 是轻量级封装，不拷贝数据，仅指向 handler.data 的内存。*/

    io.run();
    return 0;
}
```

#### 避坑点

- 避免捕获局部变量的悬垂引用：Lambda 捕获 `this` 或局部变量时，需确保变量生命周期长于异步操作；
- 错误处理：**必须检查 `error_code`**（如 `error::eof` 表示连接关闭，`error::operation_aborted` 表示操作被取消）。



### 4. `error_code`：跨平台的 “错误处理载体”

> 辨析：**异常是否终止进程，只取决于否捕获它（捕获的异常不会终止进程）；error_code 全程无异常，自然不会终止进程**。
>
> 1. 函数执行时遇到错误（如连接拒绝、网络断开），不会抛出任何异常；
> 2. 错误信息会被写入传入的 `error_code` 对象（通过 `ec.value()`/`ec.message()` 获取）；
> 3. 函数执行后，程序会继续往下走，**进程不会中断、不会终止**；
> 4. 你需要主动判断 `ec` 是否为「非空」（即 `if (ec) { ... }`）来处理错误，若不判断，程序会忽略错误继续执行。

#### 核心作用

替代 C++ 异常的错误处理机制，ASIO 所有 I/O 操作都可通过 `error_code` 返回错误（而非抛出异常），优点：

- 跨平台统一：屏蔽 Linux/Windows 原生错误码（如 `errno`/`WSAGetLastError()`）；
- 可控性强：避免异常导致的程序崩溃，可精准处理每个错误场景。

#### 关键使用规则

| 操作方式         | 说明                                                         |
| ---------------- | ------------------------------------------------------------ |
| 传参方式（推荐） | 调用同步 / 异步操作时，显式传入 `error_code` 对象接收错误    |
| 异常方式（默认） | 未传入 `error_code` 时，错误会抛出 `asio::system_error` 异常 |
| 错误判断         | 通过 `ec` 是否为 `false`（`!ec`）判断是否成功，`ec.message()` 查看描述 |
| 常见错误码       | `error::eof`（连接关闭）、`error::operation_aborted`（操作取消）、`error::connection_refused`（连接拒绝） |

#### 代码示例

```cpp
#include <asio.hpp> // 本段代码辨析 connect 方法的两种错误处理方式
#include <iostream>

int main() {
    asio::io_context io;
    asio::ip::tcp::socket sock(io); // 创建 TCP socket,绑定到 io_context,用于网络通信,默认未连接,需要调用 connect 方法连接服务器,否则无法进行读写操作--> 这是一个同步阻塞调用，客户端会等待连接完成或失败
    //  创建服务器端点（IP + 端口），准备连接。 下面注释的方式存在问题：
    //asio::ip::tcp::endpoint ep(asio::ip::address::from_string("127.0.0.1"), 9999);  //from_string 仅支持 IPv4 地址 ---> 已被弃用，不推荐使用，必须捕获异常，不够优雅
    // 使用非抛出版本解析地址（推荐）
    asio::error_code addr_ec;   // 用于接收地址解析错误
    asio::ip::address addr = asio::ip::make_address("127.0.0.1", addr_ec);
    if (addr_ec) {
        std::cerr << "地址解析失败：" << addr_ec.message() << std::endl;
        return 1;
    }
    asio::ip::tcp::endpoint ep(addr, 9999); // 创建端点

    // 方式1：传参方式（推荐，无异常）
    asio::error_code ec;
    sock.connect(ep, ec); // 传入ec接收错误
    if (ec) {
        std::cerr << "连接失败：" << ec.message() << "（错误码：" << ec.value() << "）" << std::endl;
    }else{
        std::cout << "方式1 连接成功！" << std::endl;
        sock.close(); // 连接成功后关闭，避免后续重复连接报错
    }

    // 重新创建socket（避免复用已连接/出错的socket）
    asio::ip::tcp::socket sock2(io);
    // 方式2：异常方式（默认）
    try {
        sock2.connect(ep); // 未传ec，错误会抛出异常
        std::cout << "方式2 连接成功！" << std::endl;
        sock2.close(); // 连接成功后关闭，避免后续重复连接报错
    } catch (const asio::system_error& e) {
        std::cerr << "连接失败（异常）：" << e.what() << "（错误码：" << e.code().value() << "）" << std::endl;
    }

    return 0;
}
```

* 输出

  ```cpp
  // 你的客户端程序试图连接 127.0.0.1:9999，但该地址和端口上没有运行任何 TCP 服务器程序，操作系统拒绝了这次连接请求。
  连接失败：Connection refused（错误码：61）
  连接失败（异常）：connect: Connection refused（错误码：61）
  ```

#### 避坑点

- 不要混合两种错误处理方式：同一操作要么传 `error_code`，要么捕获异常，避免逻辑混乱；
- 异步操作必须处理 `error_code`：忽略错误会导致程序在网络异常时行为不可控。



### 5. `endpoint`：网络通信的 “地址标识”

> - **Endpoint 是抽象概念**：IPv4 场景下等价于「IP 地址 + 端口号」； ---> 端点
> - **`sockaddr_in` 是具体实现**：POSIX 系统中用于存储 IPv4 Endpoint 信息的结构体，是操作 Socket API 的 “载体”；--> 地址结构体
> - 可以理解为：`sockaddr_in` 是 Endpoint 概念在 IPv4 Socket 编程中的 “标准化数据结构实现”。

#### 核心作用

封装 “IP 地址 + 端口” 的网络端点，用于：

- 服务器：绑定（`bind`）到指定 IP / 端口监听连接；
- 客户端：指定要连接的服务器地址；
- UDP：指定数据发送 / 接收的目标地址。

#### 关键使用规则

| 类型                    | 用途         | 示例                                                         |
| ----------------------- | ------------ | ------------------------------------------------------------ |
| `tcp::endpoint`         | TCP 端点     | `tcp::endpoint(tcp::v4(), 8080)`（IPv4所有网卡 + 8080 端口） |
| `udp::endpoint`         | UDP 端点     | `udp::endpoint(asio::ip::make_address("192.168.1.1"), 9000)` |
| `tcp::v4()`/`tcp::v6()` | 指定 IP 版本 | 绑定到所有 IPv4 地址：`tcp::endpoint(tcp::v4(), 8080)`       |

> ```cpp
> // 获取 endpoint 的属性
> // 创建一个 endpoint，绑定指定 IP 和端口
> asio::ip::tcp::endpoint ep(asio::ip::make_address("192.168.1.100"), 8080);  
> asio::ip::address addr = ep.address();       // 获取 IP 地址对象
> unsigned short port = ep.port();   // 获取端口（主机字节序，无需手动转换）
> bool is_ipv4 = addr.is_v4();       // 判断是否 IPv4
> bool is_ipv6 = addr.is_v6();       // 判断是否 IPv6
> std::string ip_str = addr.to_string(); // 转换为字符串（如 "192.168.1.100"）
> // 获取本地绑定的 endpoint
> tcp::endpoint local_ep = sock.local_endpoint(ec);   // 本地端点是随机的，系统分配的
> // 获取远端服务端的 endpoint
> tcp::endpoint remote_ep = sock.remote_endpoint(ec); // 远端端点是服务端的地址，即127.0.0.1:8080
> ```

#### 代码示例

```cpp
#include <asio.hpp> // 演示多种创建 TCP/UDP 端点（endpoint）的方法
#include <iostream>

int main() {
    std::error_code ec;

    //构造网络地址	// 从字符串创建 IP 地址，传入错误码接收错误
    asio::ip::address addr = asio::ip::make_address("192.168.1.1", ec); 
    if (ec) {
        std::cerr << "IP地址解析失败：" << ec.message() << std::endl
                    << "错误码：" << ec.value() << std::endl;
        return 1;
    }

    // 方式1：直接创建IPv4端点（绑定所有网卡，端口8080）
    asio::ip::tcp::endpoint ep1(asio::ip::tcp::v4(), 8080);
    if (ec) {
        std::cerr << "端点1创建失败：" << ec.message() << std::endl;
        return 1;
    }
    std::cout << "端点1：" << ep1.address() << ":" << ep1.port() << std::endl;

    // 方式2：指定具体IP地址
    //asio::ip::address ip = asio::ip::make_address("192.168.1.100");
    asio::ip::address ip = addr; // 复用前面创建的地址对象
    asio::ip::tcp::endpoint ep2(ip, 9000);
    std::cout << "端点2：" << ep2 << std::endl; // 直接输出：192.168.1.100:9000

    // 方式3：解析域名（自动转换为endpoint）---》 地址解析器
    asio::io_context io;
    asio::ip::tcp::resolver resolver(io);   // resolver 用于域名解析,是asio库中处理DNS查询的类
    // 解析"www.baidu.com:80" // 百度等大型网站的域名会绑定多个 IPv4 地址（多 IP 负载均衡 / 容灾），DNS 解析会返回所有可用 IP，因此 resolver.resolve 会输出多个结果。
    auto results = resolver.resolve("www.baidu.com", "80");
    for (auto& res : results) {
        std::cout << "解析结果：" << res.endpoint() << std::endl;  // 输出每个解析到的endpoint
    }

    return 0;
}
```

#### 避坑点

- 绑定 “所有网卡”：用 `tcp::v4()`/`tcp::v6()` 而非具体 IP，避免只能本地访问；
- 端口范围：避免使用 1-1024 知名端口（需管理员权限），推荐 1024-65535；
- IPv6 兼容：若需支持 IPv6，需创建 `tcp::v6()` 端点，或同时监听 IPv4/IPv6。



### 6. `socket`：网络通信的 “数据通道”

#### 核心作用

封装底层操作系统的 socket 描述符，提供 TCP/UDP 通信的核心操作：

- TCP：连接（`connect`）、监听（`accept`）、读（`read_some`/`async_read_some`）、写（`write_some`/`async_write_some`）；
- UDP：发送（`send_to`/`async_send_to`）、接收（`receive_from`/`async_receive_from`）。

> 首先在创建套接字时绑定上下文；不需要bind绑定套接字和地址，直接创建端点然后让套接字连接到端点

#### 关键使用规则

| 类型            | 特点               | 核心操作                                                     |
| --------------- | ------------------ | ------------------------------------------------------------ |
| `tcp::socket`   | 面向连接、可靠传输 | `connect(ep, ec)`、`read_some(asio::buffer(buf), ec)`、`write_some()`同read |
| `udp::socket`   | 无连接、不可靠传输 | `send_to()`、`receive_from()`                                |
| `tcp::acceptor` | TCP 服务器监听器   | `bind()`、`listen()`、`accept()`/`async_accept()`            |

> ### 1. TCP Socket（`ip::tcp::socket`）
>
> TCP 是面向连接的协议，因此其 socket 操作围绕 “连接建立 - 数据传输 - 连接关闭” 的生命周期展开。
>
> #### 核心操作（同步 + 异步）
>
> | 操作类型 | 同步接口                | 异步接口（回调版）                       | 异步接口（协程版，C++20+）               | 作用说明                                                     |
> | -------- | ----------------------- | ---------------------------------------- | ---------------------------------------- | ------------------------------------------------------------ |
> | 连接     | `connect(endpoint)`     | `async_connect(endpoint, handler)`       | `co_await async_connect(...)`            | 主动连接到远端服务器（客户端）                               |
> | 监听     | -（由 `acceptor` 处理） | -（由 `ip::tcp::acceptor` 处理）         | -                                        | TCP 监听由 `acceptor` 负责，socket 仅负责接收连接（见下文）  |
> | 接收连接 | -                       | `acceptor.async_accept(socket, handler)` | `co_await acceptor.async_accept(socket)` | 服务器通过 `acceptor` 接收客户端连接，将连接绑定到一个新的 socket 对象 |
> | 读数据   | `read_some(buffer)`     | `async_read_some(buffer, handler)`       | `co_await async_read_some(...)`          | 从 socket 读取数据（“some” 表示可能读取部分数据，需循环读取） |
> | 写数据   | `write_some(buffer)`    | `async_write_some(buffer, handler)`      | `co_await async_write_some(...)`         | 向 socket 写入数据（“some” 表示可能写入部分数据，需循环写入） |
> | 关闭     | `close()`               | -（同步操作）                            | -                                        | 关闭 socket 连接，释放底层资源                               |
>
> #### 关键说明
>
> - TCP 连接是 “一对一” 的：客户端 socket 连接到服务器的 `acceptor`，服务器通过 `accept` 生成一个新的 socket 与客户端通信；
> - `read_some`/`write_some` 是 “部分读写”：例如调用 `read_some` 时，可能只读取到一部分数据（底层缓冲区有多少读多少），因此实际开发中通常用 `asio::read`/`asio::async_read`（封装循环，保证读取指定长度）；
> - 异步操作的核心是 “不阻塞调用线程”：异步调用后立即返回，当操作完成（如数据读取完毕、连接建立成功）时，Asio 会调用注册的回调函数（或唤醒协程）。
>
> ### 2. UDP Socket（`ip::udp::socket`）
>
> UDP 是无连接协议，因此其 socket 无需提前建立连接，直接通过 “端点（endpoint）” 定位通信对端。
>
> #### 核心操作（同步 + 异步）
>
> | 操作类型     | 同步接口                         | 异步接口（回调版）                              | 异步接口（协程版）                 | 作用说明                                                     |
> | ------------ | -------------------------------- | ----------------------------------------------- | ---------------------------------- | ------------------------------------------------------------ |
> | 绑定端口     | `bind(endpoint)`                 | -（同步操作）                                   | -                                  | 服务器绑定固定端口，客户端可选绑定（通常由系统自动分配临时端口） |
> | 发送数据     | `send_to(buffer, endpoint)`      | `async_send_to(buffer, endpoint, handler)`      | `co_await async_send_to(...)`      | 向指定远端端点发送数据报                                     |
> | 接收数据     | `receive_from(buffer, endpoint)` | `async_receive_from(buffer, endpoint, handler)` | `co_await async_receive_from(...)` | 接收任意远端端点发送的数据报，同时通过 `endpoint` 输出对端地址 |
> | （可选）连接 | `connect(endpoint)`              | `async_connect(endpoint, handler)`              | `co_await async_connect(...)`      | 可选 “逻辑连接”：限制 socket 仅与指定端点通信，后续可直接用 `send`/`receive` |
>
> #### 关键说明
>
> - UDP 是 “数据报” 模式：每次 `send_to` 发送一个完整的数据报，`receive_from` 接收一个完整的数据报（若缓冲区不足，数据会被截断）；
> - 无连接特性：无需提前建立连接，一个 UDP socket 可与多个远端端点通信；
> - “逻辑连接” 的作用：UDP 的 `connect` 并非真正建立连接，而是过滤数据 —— 仅接收指定端点的数据，发送时无需每次指定 `endpoint`（直接用 `send`/`async_send`）。

#### 代码示例（TCP 客户端）

* 有点幽默了，这是客户端代码，客户端需要主动发起连接connect，不需要被动监听连接listen～～～

```cpp
#include <asio.hpp> // 简单的 TCP 客户端示例. asio::socket 的基本使用
#include <iostream>

int main() {
    asio::io_context io;    // 创建io_context,作为异步操作的核心调度器

    // 1. 创建TCP socket
    asio::ip::tcp::socket sock(io); // 绑定到 io_context，用于网络通信，默认未连接

    // 2. 连接到服务器端点
    asio::ip::tcp::endpoint ep(asio::ip::make_address("127.0.0.1"), 8080);
    asio::error_code ec;    // 错误码对象，可以重复使用吗？答案：可以
    sock.connect(ep, ec);   // 让套接字连接服务器端点,传入错误码接收错误    // 是一个同步阻塞调用，客户端会等待连接完成或失败
    if (ec) {
        std::cerr << "连接失败：" << ec.message() << std::endl;
        return 1;
    }

    // 3. 发送数据
    std::string data = "Hello Server!";
    ssize_t sent = sock.write_some(asio::buffer(data), ec); // 返回值到底是 ssize_t 还是 size_t？无敌了孩子，官方文档前后不一致，好像都可以
    if (!ec) {
        std::cout << "发送了" << sent << "字节" << std::endl;
    }

    // 4. 接收响应
    char buf[1024] = {0};
    size_t recv = sock.read_some(asio::buffer(buf), ec);    // read_some 是阻塞调用，等待数据到达
    if (!ec) {
        std::cout << "收到响应：" << std::string(buf, recv) << std::endl;   // std::string(buf, recv) 构造函数，指定长度
    }

    // 5. 关闭socket
    sock.close();
    return 0;
}
```

#### 避坑点

- 异步操作后不要立即销毁 socket：异步操作未完成时，socket 必须存活（用 `shared_ptr` 管理）；
- TCP 粘包：`read_some()`/`async_read_some()` 只读取 “就绪的部分数据”，需手动处理粘包（如固定长度、分隔符）；
- 关闭 socket：先调用 `cancel()` 取消未完成的异步操作，再 `close()`，避免回调触发时 socket 已析构。

#### 代码示例（TCP服务端）tcp:acceptor

```cpp
#include <asio.hpp>  // 与客户端使用相同的asio头文件 // 简单的 TCP 服务器示例，演示 acceptor 的基本使用
#include <iostream>

int main() {
    asio::io_context io;  // 核心调度器，与客户端一致

    // 1. 创建TCP acceptor（服务器专属，用于监听连接）
    // acceptor 是服务器端的核心组件，替代客户端的普通socket完成监听/接受连接
    asio::ip::tcp::acceptor acceptor(io);   // 类似监听socket，绑定到 io_context

    // 2. 绑定服务器端点（IP+端口）
    // 绑定 127.0.0.1:8080，与客户端连接的端点一致
    asio::ip::tcp::endpoint ep(asio::ip::make_address("127.0.0.1"), 8080);
    asio::error_code ec;  // 复用错误码对象，与客户端逻辑一致

    // 打开acceptor并绑定端口
    acceptor.open(ep.protocol(), ec);   // 打开acceptor，指定协议（IPv4/IPv6）// 是阻塞调用
    if (ec) {
        std::cerr << "打开acceptor失败：" << ec.message() << std::endl;
        return 1;
    }

    acceptor.bind(ep, ec);  // 绑定端点（IP+端口）,是阻塞调用
    if (ec) {
        std::cerr << "绑定端口失败：" << ec.message() << std::endl;
        return 1;
    }

    // 3. 监听客户端连接（服务器核心操作，客户端不需要）
    // listen第一个参数是backlog（连接队列长度），默认值即可
    acceptor.listen(asio::socket_base::max_listen_connections, ec);
    if (ec) {
        std::cerr << "监听端口失败：" << ec.message() << std::endl;
        return 1;
    }
    std::cout << "服务器已启动，监听 127.0.0.1:8080 ..." << std::endl;

    // 4. 接受客户端连接（阻塞调用，等待客户端连接）
    asio::ip::tcp::socket sock(io);  // 用于与客户端通信的socket（类似客户端的sock）
    acceptor.accept(sock, ec);  // 接受连接，将新连接绑定到sock
    if (ec) {
        std::cerr << "接受连接失败：" << ec.message() << std::endl;
        return 1;
    }
    std::cout << "客户端已连接！" << std::endl;

    // 5. 接收客户端发送的数据（与客户端read_some逻辑一致）
    char buf[1024] = {0};  // 接收缓冲区，与客户端一致
    size_t recv_len = sock.read_some(asio::buffer(buf), ec);
    if (ec) {
        std::cerr << "接收数据失败：" << ec.message() << std::endl;
        return 1;
    }
    std::cout << "收到客户端数据：" << std::string(buf, recv_len) << std::endl;

    // 6. 向客户端发送响应（与客户端write_some逻辑一致）
    std::string resp = "Hello Client! I got your message: " + std::string(buf, recv_len);
    ssize_t sent_len = sock.write_some(asio::buffer(resp), ec);
    if (ec) {
        std::cerr << "发送响应失败：" << ec.message() << std::endl;
        return 1;
    }
    std::cout << "向客户端发送了 " << sent_len << " 字节响应" << std::endl;

    // 7. 关闭资源（先关闭通信socket，再关闭acceptor）
    sock.close(ec);
    acceptor.close(ec);

    return 0;
}
```

> * `acceptor.open(ep.protocol(), ec);`  // 打开acceptor，指定协议（IPv4/IPv6）---> 初始化监听套接字
>   * `ep.protocol()`：`tcp::endpoint` 的成员函数，返回当前端点对应的「协议对象」（`tcp` 协议），用于告诉 `acceptor` 要使用的网络协议。
>   * 调用 `open` 的核心目的是：**为 `acceptor` 申请底层 TCP 套接字资源，完成初始化，让它具备与操作系统交互的能力**—— 这是 `bind`、`listen` 等后续操作的前提。
>
> * `acceptor.bind(ep, ec);`  // 绑定端点（IP+端口） --->  类似监听套接字绑定地址
> * `acceptor.listen(asio::socket_base::max_listen_connections, ec);` ---> 监听客户端连接
>   * 第一个参数 `int backlog`（监听队列长度），`asio::socket_base::max_listen_connections`是Asio 预定义常量，表示操作系统允许的 “最大监听队列长度”（不同系统默认值不同，如 Linux 通常是 128）。
> * `acceptor.accept(sock, ec);`  // 接受连接，将新连接绑定到sock  ---> 接收连接，创建通信套接字
>   * 第一个参数：`tcp::socket&` 类型（非 const 引用）：要求：`sock` 必须是 “未打开” 的
>   * `accept` 会自动为 `sock` 分配底层套接字资源（绑定客户端的 IP + 端口），完成初始化；



### 7. `async_*` 函数：异步操作的 “入口”

#### 核心作用

ASIO 所有以 `async_` 开头的函数（如 `async_read`、`async_accept`、`async_wait`）是 “异步操作触发入口”，特点：

- 非阻塞：调用后立即返回，操作在后台执行；
- 回调驱动：操作完成后，由 `io_context` 触发 handler；
- 可组合：多个 `async_*` 函数可组成异步操作链（如 `async_accept` → `async_read` → `async_write`）。

#### 关键使用规则

| 常用 `async_*` 函数              | 用途                | 回调签名                               |
| -------------------------------- | ------------------- | -------------------------------------- |
| `async_wait`（定时器）           | 异步等待定时器到期  | `void(const error_code&)`              |
| `async_accept`（acceptor）       | 异步接受 TCP 连接   | `void(const error_code&, tcp::socket)` |
| `async_read`/`async_read_some`   | 异步读取数据        | `void(const error_code&, size_t)`      |
| `async_write`/`async_write_some` | 异步写入数据        | `void(const error_code&, size_t)`      |
| `async_send_to`（UDP）           | 异步发送 UDP 数据报 | `void(const error_code&, size_t)`      |

> ```cpp
> /*调用该函数后不会阻塞当前线程，当定时器达到指定时间（或被取消）时，
> Asio 会在 io_context 的事件循环中触发你传入的回调函数，完成定时逻辑的异步执行。*/
> timer.async_wait([](const asio::error_code& ec) {...}
> ```

#### 代码示例（异步 TCP 接受连接）

```cpp
#include <asio.hpp> // 异步TCP服务器示例，演示 async_accept 的基本使用
#include <iostream>
#include <memory>  // 用于智能指针

// 定义连接数据结构，管理socket和缓冲区
struct ConnectionData {
    asio::ip::tcp::socket sock;
    std::array<char, 1024> buf;  // 用array替代裸数组，更安全

    // 移动构造函数（必须，因为socket不可拷贝）
    explicit ConnectionData(asio::ip::tcp::socket s) : sock(std::move(s)) {}
};

void do_read(std::shared_ptr<ConnectionData> conn);

void handle_read(const asio::error_code& ec, size_t len, 
                 std::shared_ptr<ConnectionData> conn) {
    if (ec) {
        if (ec != asio::error::eof) {  // 排除正常断开连接的情况
            std::cerr << "读取失败：" << ec.message() << std::endl;
        }
        return;
    }

    // 打印收到的数据
    std::cout << "收到客户端(" << conn->sock.remote_endpoint() << ")数据：" 
              << std::string(conn->buf.data(), len) << std::endl;   // buf.data() 获取底层指针

    // 异步回显数据给客户端
    asio::async_write(conn->sock, asio::buffer(conn->buf.data(), len),
        [conn](const asio::error_code& ec, size_t) {
            if (ec) {
                std::cerr << "回显失败：" << ec.message() << std::endl;
            } else {
                std::cout << "数据已回显给客户端" << std::endl;
            }
            // 回显完成后，继续等待下一次读取（保持连接）
            do_read(conn);
        });
}

// 封装读取逻辑，复用
void do_read(std::shared_ptr<ConnectionData> conn) {
    conn->sock.async_read_some(asio::buffer(conn->buf),
        [conn](const asio::error_code& ec, size_t len) {
            handle_read(ec, len, conn); // 继续处理读取结果
        });
}

void accept_handler(const asio::error_code& ec, asio::ip::tcp::socket sock,
                    asio::ip::tcp::acceptor& acceptor) {
    if (ec) {
        std::cerr << "接受连接失败：" << ec.message() << std::endl;
        return;
    }

    // 打印客户端信息
    std::cout << "客户端连接成功：" << sock.remote_endpoint() << std::endl;

    // 用智能指针管理连接数据，确保生命周期覆盖异步操作
    auto conn = std::make_shared<ConnectionData>(std::move(sock));
    
    // 开始异步读取客户端数据
    do_read(conn);

    // 继续接受新的连接（关键：异步accept需要循环调用）
    acceptor.async_accept([&acceptor](const asio::error_code& ec, asio::ip::tcp::socket sock) {
        accept_handler(ec, std::move(sock), acceptor);
    });
}

int main() {
    try {
        asio::io_context io;

        // 创建TCP监听器，绑定8080端口
        asio::ip::tcp::acceptor acceptor(io, asio::ip::tcp::endpoint(asio::ip::tcp::v4(), 8080));
        std::cout << "服务器启动，监听端口8080..." << std::endl;

        // 异步接受第一个连接
        acceptor.async_accept([&acceptor](const asio::error_code& ec, asio::ip::tcp::socket sock) {
            accept_handler(ec, std::move(sock), acceptor);
        });

        // 运行事件循环
        io.run();
    } catch (const std::exception& e) {
        std::cerr << "服务器异常：" << e.what() << std::endl;
    }

    return 0;
}
```

#### 避坑点

- `async_read_some` vs `async_read`：`async_read_some` 读取 “就绪的部分数据”，`async_read` 读取 “指定长度的数据”（直到读满或出错）；
- 避免重复发起异步操作：同一 socket 未完成 `async_read` 时，再次发起 `async_read` 会导致竞争；
- 回调参数传递：`async_accept` 的回调需接收 `tcp::socket`（用值传递或移动，避免悬垂引用）。

### 8. `buffer`：内存数据的 “I/O 封装”

#### 核心作用

ASIO 要求所有 I/O 操作（读 / 写）必须通过 `buffer` 封装内存，作用：

- 统一内存访问接口：支持 `std::string`、`std::vector`、原生数组、指针等；
- 安全管理：避免越界访问，自动计算缓冲区长度；
- 区分读写：只读缓冲区（如 `std::string`）、可写缓冲区（如 `std::vector<char>`）。

#### 关键使用规则

| 封装方式                    | 示例                                                 | 说明                                    |
| --------------------------- | ---------------------------------------------------- | --------------------------------------- |
| 动态容器                    | `asio::buffer(vec)`（vec 是 std::vector<char>）      | 可写，长度为 vec.size ()                |
| 字符串                      | `asio::buffer(str)`（str 是 std::string）            | 只读（C++17 前），长度为 str.size ()    |
| 原生数组 / 指针             | `asio::buffer(buf, len)`（buf 是 char*，len 是长度） | 可写，指定固定长度                      |
| 只读缓冲区                  | `asio::const_buffer(buf)`                            | 显式标记为只读，避免修改                |
| 多缓冲区（分散 / 聚合 I/O） | `asio::buffer_sequence`                              | 一次读写多个缓冲区（如 `{buf1, buf2}`） |

#### 代码示例

```cpp
#include <asio.hpp> // 展示asio缓冲区的多种用法
#include <iostream>
#include <vector>
#include <string>
#include <cstring>  // 新增：用于strlen

int main() {
    // 1. 可写缓冲区（vector）- 修复指针算术运算警告
    std::vector<char> write_buf(1024);
    const char* hello_str = "Hello ASIO";
    // 方式1：用strlen获取长度，避免硬编码9，更健壮
    std::copy(hello_str, hello_str + std::strlen(hello_str), write_buf.begin());
    asio::mutable_buffer buf1 = asio::buffer(write_buf); // 可写
    std::cout << "缓冲区1长度：" << buf1.size() << std::endl; // 1024

    // 2. 只读缓冲区（string）
    std::string read_data = "Read Only Data";
    asio::const_buffer buf2 = asio::buffer(read_data); // 只读
    std::cout << "缓冲区2长度：" << buf2.size() << std::endl; // 14

    // 3. 固定长度缓冲区（原生数组）
    char raw_buf[512];
    asio::mutable_buffer buf3 = asio::buffer(raw_buf, 256); // 仅封装前256字节
    std::cout << "缓冲区3长度：" << buf3.size() << std::endl; // 256

    // 4. 多缓冲区（分散写）
    std::vector<asio::const_buffer> bufs;
    bufs.push_back(asio::buffer("Part1: "));    // 字符串字面值, 长度7
    bufs.push_back(asio::buffer(read_data));
    
    // 可选：打印多缓冲区总长度（验证）
    size_t total_size = 0;
    for (const auto& b : bufs) {
        total_size += b.size();
    }
    std::cout << "多缓冲区总长度：" << total_size << std::endl; // 7 + 14 = 21 ？ 22

    return 0;
}
```

#### 避坑点

- 缓冲区生命周期：异步操作未完成时，缓冲区必须存活（如不能用局部变量作为异步写的缓冲区）；
- 字符串缓冲区：C++17 前 `std::string` 封装的 buffer 是只读的，若需写，用 `std::vector<char>`；
- 长度控制：`async_read` 会一直读直到缓冲区满，需确保缓冲区足够大，避免内存溢出。



### 总结：核心概念的联动关系

这些概念并非孤立，而是形成完整的异步 I/O 流程：

```plaintext
io_context（调度核心）
  ↓ 管理
async_* 函数（异步入口）→ 绑定 strand（串行保护）
  ↓ 操作对象
socket/acceptor（数据通道）→ 关联 endpoint（网络地址）
  ↓ 数据载体
buffer（内存封装）
  ↓ 结果处理
handler（回调）→ 用 error_code（错误处理）反馈结果
```

理解这个联动关系，就能掌握 ASIO 异步编程的核心逻辑：以 `io_context` 为驱动，通过 `async_*` 函数触发 I/O 操作，用 `buffer` 传递数据，`handler` 处理结果，`strand` 保证线程安全，`endpoint` 定位网络地址，`error_code` 处理异常。