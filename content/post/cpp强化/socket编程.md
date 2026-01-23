# Socket编程

Socket（套接字）是网络编程的核心接口，用于实现不同主机（或同一主机）间的进程通信。C++ 中 Socket 编程基于操作系统提供的网络 API（如 POSIX 标准的 `sys/socket.h`、`netinet/in.h` 等），核心分为**TCP**（面向连接、可靠）和 **UDP**（无连接、不可靠）两种协议。

### 一、核心概念

1. **Socket 类型**

   - `SOCK_STREAM`：TCP 套接字，面向连接，字节流传输，可靠有序。
   - `SOCK_DGRAM`：UDP 套接字，无连接，数据报传输，速度快但可能丢包 / 乱序。

2. **核心函数**（Linux/macOS 下，Windows 需适配 `WSAStartup`）：

   | 函数                  | 作用                       |
   | --------------------- | -------------------------- |
   | `socket()`            | 创建套接字文件描述符       |
   | `bind()`              | 绑定 IP + 端口到套接字     |
   | `listen()`            | TCP 监听连接（服务端）     |
   | `accept()`            | TCP 接受客户端连接（阻塞） |
   | `connect()`           | TCP 客户端发起连接         |
   | `send()/recv()`       | TCP 发送 / 接收数据        |
   | `sendto()/recvfrom()` | UDP 发送 / 接收数据        |
   | `close()`             | 关闭套接字                 |

### 二、TCP 套接字编程（C++ 示例）

TCP 是**客户端 - 服务端**模型，流程：

**服务端**：创建套接字 → 绑定 → 监听 → 接受连接 → 读写数据 → 关闭

**客户端**：创建套接字 → 连接服务端 → 读写数据 → 关闭

#### 1. TCP 服务端代码

```cpp
#include <iostream>
#include <cstring>
#include <sys/socket.h>
#include <netinet/in.h>
#include <unistd.h>

#define PORT 8080
#define BUFFER_SIZE 1024

int main() {
    int server_fd, new_socket;
    struct sockaddr_in address;
    int opt = 1;
    int addrlen = sizeof(address);
    char buffer[BUFFER_SIZE] = {0};
    const char* hello = "Hello from TCP Server";

    // 1. 创建套接字（IPv4、TCP、默认协议）
    if ((server_fd = socket(AF_INET, SOCK_STREAM, 0)) == 0) {
        perror("socket failed");
        exit(EXIT_FAILURE);
    }

    // 2. 设置套接字选项（复用端口/IP）
    if (setsockopt(server_fd, SOL_SOCKET, SO_REUSEADDR | SO_REUSEPORT, &opt, sizeof(opt))) {
        perror("setsockopt");
        exit(EXIT_FAILURE);
    }

    // 配置地址结构（IPv4、端口、绑定所有网卡）
    address.sin_family = AF_INET;
    address.sin_addr.s_addr = INADDR_ANY; // 监听所有本地IP
    address.sin_port = htons(PORT);       // 端口转换为网络字节序

    // 3. 绑定套接字到IP和端口
    if (bind(server_fd, (struct sockaddr *)&address, sizeof(address)) < 0) {
        perror("bind failed");
        exit(EXIT_FAILURE);
    }

    // 4. 监听连接（最大等待队列数5）
    if (listen(server_fd, 5) < 0) {
        perror("listen");
        exit(EXIT_FAILURE);
    }
    std::cout << "TCP Server listening on port " << PORT << std::endl;

    // 5. 接受客户端连接（阻塞，返回新的套接字用于通信）
    if ((new_socket = accept(server_fd, (struct sockaddr *)&address, (socklen_t*)&addrlen)) < 0) {
        perror("accept");
        exit(EXIT_FAILURE);
    }

    // 6. 读取客户端数据
    ssize_t valread = read(new_socket, buffer, BUFFER_SIZE);
    std::cout << "Received from client: " << buffer << std::endl;

    // 7. 向客户端发送响应
    send(new_socket, hello, strlen(hello), 0);
    std::cout << "Hello message sent to client" << std::endl;

    // 8. 关闭套接字
    close(new_socket);
    close(server_fd);

    return 0;
}
```

#### 2. TCP 客户端代码

```cpp
#include <iostream>
#include <cstring>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <unistd.h>

#define PORT 8080
#define BUFFER_SIZE 1024

int main() {
    int sock = 0;
    struct sockaddr_in serv_addr;
    char buffer[BUFFER_SIZE] = {0};
    const char* hello = "Hello from TCP Client";

    // 1. 创建套接字
    if ((sock = socket(AF_INET, SOCK_STREAM, 0)) < 0) {
        perror("socket creation error");
        exit(EXIT_FAILURE);
    }

    // 配置服务端地址
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_port = htons(PORT);

    // 转换IP地址（字符串→网络字节序）
    if (inet_pton(AF_INET, "127.0.0.1", &serv_addr.sin_addr) <= 0) {
        perror("invalid address/ Address not supported");
        exit(EXIT_FAILURE);
    }

    // 2. 连接服务端
    if (connect(sock, (struct sockaddr *)&serv_addr, sizeof(serv_addr)) < 0) {
        perror("connection failed");
        exit(EXIT_FAILURE);
    }

    // 3. 向服务端发送数据
    send(sock, hello, strlen(hello), 0);
    std::cout << "Hello message sent to server" << std::endl;

    // 4. 读取服务端响应
    ssize_t valread = read(sock, buffer, BUFFER_SIZE);
    std::cout << "Received from server: " << buffer << std::endl;

    // 5. 关闭套接字
    close(sock);

    return 0;
}
```

### 三、UDP 套接字编程（C++ 示例）

UDP 无连接，无需 `listen()`/`accept()`/`connect()`（可选 `connect()` 固定目标），直接通过 `sendto()`/`recvfrom()` 收发数据。

#### 1. UDP 服务端代码

```cpp
#include <iostream>
#include <cstring>
#include <sys/socket.h>
#include <netinet/in.h>
#include <unistd.h>

#define PORT 8080
#define BUFFER_SIZE 1024

int main() {
    int sock_fd;
    struct sockaddr_in serv_addr, cli_addr;
    char buffer[BUFFER_SIZE] = {0};
    const char* hello = "Hello from UDP Server";
    socklen_t len = sizeof(cli_addr);

    // 1. 创建UDP套接字
    if ((sock_fd = socket(AF_INET, SOCK_DGRAM, 0)) < 0) {
        perror("socket creation failed");
        exit(EXIT_FAILURE);
    }

    // 配置服务端地址
    memset(&serv_addr, 0, sizeof(serv_addr));
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_addr.s_addr = INADDR_ANY;
    serv_addr.sin_port = htons(PORT);

    // 2. 绑定端口
    if (bind(sock_fd, (struct sockaddr *)&serv_addr, sizeof(serv_addr)) < 0) {
        perror("bind failed");
        exit(EXIT_FAILURE);
    }

    std::cout << "UDP Server listening on port " << PORT << std::endl;

    // 3. 接收客户端数据（阻塞）
    ssize_t n = recvfrom(sock_fd, buffer, BUFFER_SIZE, 0, (struct sockaddr *)&cli_addr, &len);
    buffer[n] = '\0'; // 手动补字符串结束符
    std::cout << "Received from client: " << buffer << std::endl;

    // 4. 向客户端发送响应
    sendto(sock_fd, hello, strlen(hello), 0, (struct sockaddr *)&cli_addr, len);
    std::cout << "Hello message sent to client" << std::endl;

    // 5. 关闭套接字
    close(sock_fd);

    return 0;
}
```

#### 2. UDP 客户端代码

```cpp
#include <iostream>
#include <cstring>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <unistd.h>

#define PORT 8080
#define BUFFER_SIZE 1024

int main() {
    int sock_fd;
    struct sockaddr_in serv_addr;
    char buffer[BUFFER_SIZE] = {0};
    const char* hello = "Hello from UDP Client";
    socklen_t len = sizeof(serv_addr);

    // 1. 创建UDP套接字
    if ((sock_fd = socket(AF_INET, SOCK_DGRAM, 0)) < 0) {
        perror("socket creation failed");
        exit(EXIT_FAILURE);
    }

    // 配置服务端地址
    memset(&serv_addr, 0, sizeof(serv_addr));
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_port = htons(PORT);
    serv_addr.sin_addr.s_addr = inet_addr("127.0.0.1");

    // 2. 向服务端发送数据
    sendto(sock_fd, hello, strlen(hello), 0, (struct sockaddr *)&serv_addr, len);
    std::cout << "Hello message sent to server" << std::endl;

    // 3. 接收服务端响应
    ssize_t n = recvfrom(sock_fd, buffer, BUFFER_SIZE, 0, (struct sockaddr *)&serv_addr, &len);
    buffer[n] = '\0';
    std::cout << "Received from server: " << buffer << std::endl;

    // 4. 关闭套接字
    close(sock_fd);

    return 0;
}
```

### 四、关键注意事项

1. **字节序转换**：
   - 主机字节序（小端 / 大端）→ 网络字节序（大端）：`htons()`（端口）、`htonl()`（IP）。
   - 网络字节序 → 主机字节序：`ntohs()`、`ntohl()`。
2. **错误处理**：所有系统调用（`socket()`/`bind()`/`connect()` 等）需检查返回值，用 `perror()` 打印错误信息。
3. **跨平台适配**：
   - Windows 需先调用 `WSAStartup()` 初始化 Winsock 库，关闭用 `closesocket()` 而非 `close()`。
   - Linux/macOS 直接使用 POSIX API。
4. **阻塞 / 非阻塞**：默认套接字是阻塞的（`accept()`/`recv()` 会等待），可通过 `fcntl()` 设置为非阻塞。
5. **资源释放**：必须关闭套接字，避免文件描述符泄漏。

### 五、编译与运行

```bash
# 编译TCP服务端
g++ tcp_server.cpp -o tcp_server
# 编译TCP客户端
g++ tcp_client.cpp -o tcp_client

# 先启动服务端
./tcp_server
# 另一个终端启动客户端
./tcp_client

# UDP同理
g++ udp_server.cpp -o udp_server
g++ udp_client.cpp -o udp_client
./udp_server
./udp_client
```

### 六、C++ 封装建议

实际开发中，可将 Socket 封装为类，简化重复逻辑：

```cpp
class TCPSocket {
private:
    int fd_;
    struct sockaddr_in addr_;
public:
    TCPSocket() : fd_(-1) {}
    ~TCPSocket() { if (fd_ >= 0) close(fd_); }

    bool create();
    bool bind(int port);
    bool listen(int backlog = 5);
    int accept();
    bool connect(const std::string& ip, int port);
    ssize_t send(const std::string& data);
    ssize_t recv(std::string& data, size_t buf_size);
};
```

通过封装可提高代码复用性，同时隐藏底层系统调用细节。



## -----------------------------------

## socket结构

### 一、核心概念总览

Socket 地址结构是网络编程中**描述通信端点（IP + 端口）** 的标准化数据结构，不同 IP 协议（IPv4/IPv6）有专属结构，同时需处理**字节序转换**和**地址格式转换**（字符串↔网络字节序），核心目的是让操作系统识别并定位网络通信的目标 / 本地端点。

### 二、逐模块详细解释

#### 1. 通用 Socket 地址结构 `sockaddr`

```c
struct sockaddr {
    unsigned short sa_family;  // 地址家族（协议族），如AF_INET(IPv4)、AF_INET6(IPv6)
    char sa_data[14];          // 存放IP+端口（长度固定14字节，通用性强但使用不便）
};
```

- **作用**：作为「通用接口」，Socket API（如`bind()`/`connect()`）的参数要求传入`struct sockaddr*`类型，因此 IPv4/IPv6 的专属结构需强制转换为该类型。
- **缺点**：`sa_data`是混合存储的字符数组，无法直接拆分 IP / 端口，因此实际开发中几乎不用，仅作接口兼容。

#### 2. IPv4 专属结构 `sockaddr_in`

```c
struct sockaddr_in {
    short int sin_family;      // 必须为AF_INET（标识IPv4）
    unsigned short sin_port;   // 端口号（16位，需用网络字节序）
    struct in_addr sin_addr;   // IPv4地址（32位）
    unsigned char sin_zero[8]; // 填充字段（凑齐16字节，与sockaddr大小一致，通常置0）
};

struct in_addr {
    unsigned long s_addr;      // IPv4地址（32位，网络字节序）
};
```

- **核心优势**：将 IP、端口、协议族拆分，便于编程操作（而非像`sockaddr`那样混合存储）。
- **关键注意**：
  - `sin_port`：必须用`htons()`转换为网络字节序（不能直接赋值主机字节序的端口号）；
  - `sin_zero`：仅用于填充，需用`memset(&addr.sin_zero, 0, 8)`置 0，保证结构大小与`sockaddr`一致；
  - `sin_addr.s_addr`：IPv4 地址的 32 位整数（网络字节序），如`127.0.0.1`对应`0x0100007F`（大端序）。

#### 3. IPv6 专属结构 `sockaddr_in6`

```c
struct sockaddr_in6 {
    u_int16_t sin6_family;     // 必须为AF_INET6（标识IPv6）
    u_int16_t sin6_port;       // 端口号（16位，网络字节序）
    u_int32_t sin6_flowinfo;   // IPv6流标签（用于QoS，通常置0）
    struct in6_addr sin6_addr; // IPv6地址（128位）
    u_int32_t sin6_scope_id;   // 作用域ID（如本地链路地址的网卡索引）
};

struct in6_addr {
    unsigned char s6_addr[16]; // IPv6地址（128位，16个字节，网络字节序）
};
```

- **特点**：
  - IPv6 地址是 128 位（而非 IPv4 的 32 位），因此用 16 字节数组存储；
  - `sin6_flowinfo`/`sin6_scope_id`是 IPv6 特有字段，普通场景可置 0；
  - 端口号仍为 16 位，字节序转换规则与 IPv4 一致（`htons()`/`ntohs()`）。

#### 4. 字节序转换（核心痛点）

网络协议规定：**所有传输的多字节数据（IP、端口）必须使用大端序（网络字节序）**，但主机字节序可能是大端 / 小端（x86 架构为小端），因此必须转换：

| 函数      | 功能                             | 适用场景              |
| --------- | -------------------------------- | --------------------- |
| `htons()` | 主机字节序 → 网络字节序（16 位） | 端口号转换（如 8080） |
| `htonl()` | 主机字节序 → 网络字节序（32 位） | IPv4 地址整数转换     |
| `ntohs()` | 网络字节序 → 主机字节序（16 位） | 解析收到的端口号      |
| `ntohl()` | 网络字节序 → 主机字节序（32 位） | 解析收到的 IPv4 地址  |

- **示例理解**：
  - 主机字节序的`8080`（十进制）= `0x1F90`（十六进制，小端序存储为`0x90 0x1F`），经`htons()`转换后变为`0x1F90`（大端序，网络传输格式）。
  - htons ： host to network short

#### 5. 地址格式转换（字符串↔网络字节序）

实际开发中，IP 地址通常是字符串（如`127.0.0.1`/`2001:0db8::1`），需转换为网络字节序的二进制格式，或反向转换：

| 函数          | 功能                               | 支持协议  | 特点                         |
| ------------- | ---------------------------------- | --------- | ---------------------------- |
| `inet_addr()` | 点分十进制字符串 → IPv4 网络字节序 | IPv4      | 简单但已过时，出错返回 - 1   |
| `inet_ntoa()` | IPv4 网络字节序 → 点分十进制字符串 | IPv4      | 返回静态缓冲区（线程不安全） |
| `inet_pton()` | 字符串 → 网络字节序（二进制）      | IPv4/IPv6 | 现代推荐，线程安全           |
| `inet_ntop()` | 网络字节序 → 字符串                | IPv4/IPv6 | 现代推荐，需手动分配缓冲区   |

- **关键示例**：

  ```c
  // IPv4字符串转网络字节序（现代写法）
  const char* ip_str = "192.168.1.1";
  struct in_addr ipv4_addr;
  inet_pton(AF_INET, ip_str, &ipv4_addr); // 结果存入ipv4_addr.s_addr（网络字节序）
  
  // 网络字节序转IPv4字符串
  char buf[INET_ADDRSTRLEN]; // INET_ADDRSTRLEN=16（IPv4字符串最大长度）
  inet_ntop(AF_INET, &ipv4_addr, buf, sizeof(buf)); // buf="192.168.1.1"
  
  // IPv6示例
  const char* ipv6_str = "2001:0db8::1";
  struct in6_addr ipv6_addr;
  inet_pton(AF_INET6, ipv6_str, &ipv6_addr); // 转换为16字节二进制
  
  char ipv6_buf[INET6_ADDRSTRLEN]; // INET6_ADDRSTRLEN=46（IPv6字符串最大长度）
  inet_ntop(AF_INET6, &ipv6_addr, ipv6_buf, sizeof(ipv6_buf));
  ```

  

### 三、实战场景示例（IPv4 服务器绑定地址）

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>

int main() {
    // 1. 创建IPv4 TCP Socket
    int sockfd = socket(AF_INET, SOCK_STREAM, 0);
    if (sockfd < 0) { perror("socket error"); return -1; }

    // 2. 初始化sockaddr_in结构
    struct sockaddr_in server_addr;
    memset(&server_addr, 0, sizeof(server_addr)); // 整体置0（包括sin_zero）
    server_addr.sin_family = AF_INET;             // 标识IPv4
    server_addr.sin_port = htons(8080);           // 端口转网络字节序
    // 绑定所有网卡（INADDR_ANY=0.0.0.0，等价于htonl(INADDR_ANY)）
    server_addr.sin_addr.s_addr = htonl(INADDR_ANY);

    // 3. 绑定地址（强制转换为sockaddr*）
    if (bind(sockfd, (struct sockaddr*)&server_addr, sizeof(server_addr)) < 0) {
        perror("bind error");
        close(sockfd);
        return -1;
    }

    printf("绑定成功：0.0.0.0:%d\n", ntohs(server_addr.sin_port));
    close(sockfd);
    return 0;
}
```

### 四、关键总结

1. **结构选择**：IPv4 用`sockaddr_in`，IPv6 用`sockaddr_in6`，仅在 API 传参时转为`sockaddr*`；
2. **字节序**：端口 / IP 必须用`htons()`/`htonl()`转网络字节序，解析时用`ntohs()`/`ntohl()`转回；
3. **地址转换**：优先使用`inet_pton()`/`inet_ntop()`（支持 IPv6、线程安全），避免过时的`inet_addr()`/`inet_ntoa()`；
4. **填充字段**：`sockaddr_in`的`sin_zero`必须置 0，保证结构大小与`sockaddr`一致。

这些结构和函数是 TCP/UDP 网络编程的基础，所有涉及「绑定地址」「连接目标」「解析地址」的场景都离不开它们。



## --------------------------------



# Socket 编程核心函数详解（附代码示例）

Socket（套接字）是网络编程的基础，用于实现不同主机间的进程通信。本文以**TCP 协议**为例（UDP 逻辑类似但无连接过程），详解核心函数，并通过完整的 C 语言客户端 / 服务端代码演示调用流程。

## 一、核心函数分类与作用

Socket 编程核心函数分为**基础创建**、**地址绑定 / 监听**、**连接 / 接收连接**、**数据收发**、**关闭**五大类，先梳理核心函数的功能、参数和返回值：

### 1. socket ()：创建套接字（文件描述符）

- **功能**：创建一个套接字，返回用于后续操作的文件描述符（fd），是所有网络通信的起点。

- **函数原型**：

  ```c
  int socket(int domain, int type, int protocol);
  ```

  

- **参数说明**：

  - `domain`：地址族（协议族），常用：
    - `AF_INET`：IPv4 协议（最常用）；
    - `AF_INET6`：IPv6 协议；
    - `AF_UNIX`：本地进程通信（Unix 域套接字）。
  - `type`：套接字类型，常用：
    - `SOCK_STREAM`：流式套接字（TCP，可靠、面向连接）；
    - `SOCK_DGRAM`：数据报套接字（UDP，不可靠、无连接）。
  - `protocol`：协议类型，通常传`0`（自动匹配 type 对应的默认协议，如 TCP 对应`IPPROTO_TCP`，UDP 对应`IPPROTO_UDP`）。

- **返回值**：成功返回非负文件描述符（fd）；失败返回`-1`（需检查`errno`）。

### 2. bind ()：绑定 IP 和端口到套接字

- **功能**：将套接字与指定的 IP 地址和端口号绑定（仅服务端需要，客户端通常由系统自动分配端口）。

- **函数原型**：

  ```c
  int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
  ```

  

- **参数说明**：

  - `sockfd`：`socket()`返回的套接字描述符；
  - `addr`：指向`struct sockaddr`的指针（实际常用`struct sockaddr_in`（IPv4）强转，因为`struct sockaddr`是通用结构）；
  - `addrlen`：`addr`结构体的长度（如`sizeof(struct sockaddr_in)`）。

- **返回值**：成功返回`0`；失败返回`-1`。

### 3. listen ()：监听套接字（仅 TCP 服务端）

- **功能**：将套接字转为 “监听状态”，等待客户端连接（仅 TCP 需要，UDP 无连接无需监听）。

- **函数原型**：

  ```c
  int listen(int sockfd, int backlog);
  ```

  

- **参数说明**：

  - `sockfd`：绑定后的套接字描述符；
  - `backlog`：未处理的连接队列最大长度（如`5`，表示最多同时有 5 个待处理连接，超出则拒绝）。

- **返回值**：成功返回`0`；失败返回`-1`。

### 4. accept ()：接受客户端连接（仅 TCP 服务端）

- **功能**：从监听队列中取出一个客户端连接，创建新的套接字（用于与该客户端通信，原监听套接字仍可继续监听）。

  > `ccept` 的核心作用是从「已完成三次握手的连接队列」中取出一个就绪连接，并为该连接创建新的套接字（而非 “重新建立连接”）监听套接字 --> 创建通信套接字

- **函数原型**：

  ```c
  int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
  ```

  

- **参数说明**：

  - `sockfd`：监听状态的套接字描述符；
  - `addr`：输出参数，**存储**客户端的 IP 和端口信息（可传`NULL`表示不关心）；
  - `addrlen`：输入输出参数，传入`addr`结构体长度，返回实际长度（可传`NULL`）。

- **返回值**：成功返回新的通信套接字描述符；失败返回`-1`。

> * 扩展接口，支持通过 `flags` 参数设置套接字属性（这里是 `SOCK_NONBLOCK`）
> * `SOCK_NONBLOCK`：核心标志，让创建的 `clientFd` 成为**非阻塞套接字**（适配 epoll 非阻塞 IO 模型）。
>
> ```cpp
> // 接受连接（非阻塞）
> int clientFd = accept4(listenFd, (sockaddr*)&clientAddr, &clientLen, SOCK_NONBLOCK);
> ```

### 5. connect ()：客户端发起连接（仅 TCP 客户端）

- **功能**：客户端向服务端的 IP 和端口发起 TCP 连接（UDP 无需调用，直接收发）。

- **函数原型**：

  ```c
  int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
  ```

  

- **参数说明**：

  - `sockfd`：客户端的套接字描述符；
  - `addr`：服务端的 IP 和端口结构体（`struct sockaddr_in`）；
  - `addrlen`：结构体长度。

- **返回值**：成功返回`0`；失败返回`-1`。

### 6. read ()/write () /recv ()/send ()：数据收发

- **功能**：通过套接字收发数据（`read/write`是通用文件操作函数，`recv/send`是套接字专用，支持额外标志）。

- **函数原型（以 recv/send 为例）**：

  ```c
  ssize_t recv(int sockfd, void *buf, size_t len, int flags);
  ssize_t send(int sockfd, const void *buf, size_t len, int flags);
  ```

  

- **参数说明**：

  - ssize_t ： 是一种有符号整数类型，通常用于表示字节数或错误码，定义在 <sys/types.h> 中。

  

  - `sockfd`：通信套接字描述符（服务端是`accept()`返回的 fd，客户端是`socket()`返回的 fd）；
  - `buf`：数据缓冲区（接收 / 发送的数据存储位置）；
  - `len`：缓冲区长度；
  - `flags`：收发标志（如`0`表示阻塞模式，`MSG_DONTWAIT`表示非阻塞）。

- **返回值**：

  - 成功：返回实际收发的字节数；
  - 失败：返回`-1`；
  - `recv`返回`0`：表示对方关闭连接。

### 7. close ()：关闭套接字

- **功能**：释放套接字资源，终止通信。

- **函数原型**：

  ```c
  int close(int sockfd);
  ```

  

- **参数**：`sockfd`：要关闭的套接字描述符；

- **返回值**：成功返回`0`；失败返回`-1`。



## 二、关键结构体：struct sockaddr_in（IPv4）

`bind()`/`connect()`/`accept()`都依赖该结构体存储 IP 和端口信息，定义如下：

```c
#include <netinet/in.h>

struct sockaddr_in {
    sa_family_t    sin_family;  // 地址族，固定为AF_INET
    in_port_t      sin_port;    // 端口号（需用htons()转网络字节序）
    struct in_addr sin_addr;    // IP地址（in_addr结构体仅含一个成员s_addr）
    char           sin_zero[8]; // 填充字段，置0即可
};

struct in_addr {
    uint32_t       s_addr;      // IP地址（需用inet_addr()或inet_pton()转网络字节序）
};
```

- **网络字节序**：网络传输统一使用大端序，本地字节序（小端 / 大端）需通过`htons()`（端口）、`htonl()`（IP）转为网络序；`ntohs()`/`ntohl()`则反向转换。
- **IP 转换**：`inet_addr("127.0.0.1")`将字符串 IP 转为网络序整数；`inet_pton(AF_INET, "127.0.0.1", &addr.sin_addr)`更推荐（支持 IPv6）。

## 三、完整代码示例（TCP 客户端 + 服务端）

### 1. TCP 服务端代码（server.c）

功能：创建套接字→绑定 IP + 端口→监听→接受连接→收发数据→关闭连接

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>

#define PORT 8888       // 服务端端口
#define BUF_SIZE 1024   // 缓冲区大小

int main() {
    // 1. 创建套接字（IPv4、TCP）
    int listen_fd = socket(AF_INET, SOCK_STREAM, 0);
    if (listen_fd == -1) {
        perror("socket failed"); // perror打印errno对应的错误信息
        exit(EXIT_FAILURE);
    }
    printf("创建套接字成功，listen_fd = %d\n", listen_fd);

    // 2. 初始化地址结构体（绑定本机任意IP + 8888端口）
    struct sockaddr_in server_addr;
    memset(&server_addr, 0, sizeof(server_addr)); // 清空结构体
    server_addr.sin_family = AF_INET;              // IPv4
    server_addr.sin_port = htons(PORT);            // 端口转网络序
    server_addr.sin_addr.s_addr = INADDR_ANY;      // 绑定本机所有网卡IP（0.0.0.0）

    // 3. 绑定套接字与地址
    if (bind(listen_fd, (struct sockaddr*)&server_addr, sizeof(server_addr)) == -1) {
        perror("bind failed");
        close(listen_fd);
        exit(EXIT_FAILURE);
    }
    printf("绑定端口 %d 成功\n", PORT);

    // 4. 监听套接字（队列长度5）
    if (listen(listen_fd, 5) == -1) {
        perror("listen failed");
        close(listen_fd);
        exit(EXIT_FAILURE);
    }
    printf("监听中...等待客户端连接\n");

    // 5. 接受客户端连接（阻塞等待）
    struct sockaddr_in client_addr;
    socklen_t client_addr_len = sizeof(client_addr);
    int conn_fd = accept(listen_fd, (struct sockaddr*)&client_addr, &client_addr_len);
    if (conn_fd == -1) {
        perror("accept failed");
        close(listen_fd);
        exit(EXIT_FAILURE);
    }
    // 打印客户端信息（ntohs转本地序，inet_ntoa转字符串IP）
    printf("客户端已连接：IP = %s, 端口 = %d, conn_fd = %d\n",
           inet_ntoa(client_addr.sin_addr), ntohs(client_addr.sin_port), conn_fd);

    // 6. 与客户端收发数据
    char buf[BUF_SIZE];
    while (1) {
        // 接收客户端数据（阻塞）
        ssize_t recv_len = recv(conn_fd, buf, BUF_SIZE - 1, 0);
        if (recv_len == -1) {
            perror("recv failed");
            break;
        } else if (recv_len == 0) {
            printf("客户端关闭连接\n");
            break;
        }
        buf[recv_len] = '\0'; // 手动加字符串结束符
        printf("收到客户端消息：%s\n", buf);

        // 回复客户端
        const char* reply = "服务端已收到：";
        send(conn_fd, reply, strlen(reply), 0);
        send(conn_fd, buf, recv_len, 0);
        send(conn_fd, "\n", 1, 0);

        // 若客户端发"exit"，关闭连接
        if (strcmp(buf, "exit") == 0) {
            printf("客户端请求退出，关闭连接\n");
            break;
        }
    }

    // 7. 关闭套接字
    close(conn_fd);    // 关闭与客户端的通信套接字
    close(listen_fd);  // 关闭监听套接字
    printf("连接已关闭，程序退出\n");
    return 0;
}
```

### 2. TCP 客户端代码（client.c）

功能：创建套接字→连接服务端→收发数据→关闭套接字

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>

#define SERVER_IP "127.0.0.1" // 服务端IP（本机测试）
#define SERVER_PORT 8888      // 服务端端口
#define BUF_SIZE 1024         // 缓冲区大小

int main() {
    // 1. 创建套接字（IPv4、TCP）
    int client_fd = socket(AF_INET, SOCK_STREAM, 0);
    if (client_fd == -1) {
        perror("socket failed");
        exit(EXIT_FAILURE);
    }
    printf("创建客户端套接字成功，client_fd = %d\n", client_fd);

    // 2. 初始化服务端地址结构体
    struct sockaddr_in server_addr;
    memset(&server_addr, 0, sizeof(server_addr));
    server_addr.sin_family = AF_INET;
    server_addr.sin_port = htons(SERVER_PORT);
    // 将字符串IP转为网络序整数（也可用inet_pton(AF_INET, SERVER_IP, &server_addr.sin_addr)）
    server_addr.sin_addr.s_addr = inet_addr(SERVER_IP);

    // 3. 连接服务端
    if (connect(client_fd, (struct sockaddr*)&server_addr, sizeof(server_addr)) == -1) {
        perror("connect failed");
        close(client_fd);
        exit(EXIT_FAILURE);
    }
    printf("成功连接服务端 %s:%d\n", SERVER_IP, SERVER_PORT);

    // 4. 与服务端收发数据
    char send_buf[BUF_SIZE];
    char recv_buf[BUF_SIZE];
    while (1) {
        // 输入要发送的消息
        printf("请输入发送给服务端的消息（输入exit退出）：");
        fgets(send_buf, BUF_SIZE, stdin);
        // 去掉fgets读取的换行符
        send_buf[strcspn(send_buf, "\n")] = '\0';

        // 发送数据到服务端
        ssize_t send_len = send(client_fd, send_buf, strlen(send_buf), 0);
        if (send_len == -1) {
            perror("send failed");
            break;
        }

        // 若输入exit，退出循环
        if (strcmp(send_buf, "exit") == 0) {
            break;
        }

        // 接收服务端回复
        ssize_t recv_len = recv(client_fd, recv_buf, BUF_SIZE - 1, 0);
        if (recv_len == -1) {
            perror("recv failed");
            break;
        } else if (recv_len == 0) {
            printf("服务端关闭连接\n");
            break;
        }
        recv_buf[recv_len] = '\0';
        printf("收到服务端回复：%s\n", recv_buf);
    }

    // 5. 关闭套接字
    close(client_fd);
    printf("客户端退出\n");
    return 0;
}
```

## 四、代码编译与运行

### 编译命令

```bash
gcc server.c -o server
gcc client.c -o client
```

### 运行步骤

1. 先启动服务端：

   ```bash
   ./server
   ```

   输出：

   ```plaintext
   创建套接字成功，listen_fd = 3
   绑定端口 8888 成功
   监听中...等待客户端连接
   ```

2. 再启动客户端（新终端）：

   ```bash
   ./client
   ```

   输出：

   ```plaintext
   创建客户端套接字成功，client_fd = 3
   成功连接服务端 127.0.0.1:8888
   请输入发送给服务端的消息（输入exit退出）：
   ```

3. 客户端输入消息（如`hello socket`），服务端会收到并回复，输入`exit`则双方退出。

## 五、UDP 核心函数差异说明

UDP 是无连接协议，无需`listen()`/`accept()`/`connect()`（也可调用`connect()`绑定目标地址，简化收发），核心差异：

1. 创建套接字时`type`为`SOCK_DGRAM`；

2. 收发数据用`recvfrom()`/`sendto()`（需指定对方地址）：

   ```c
   ssize_t recvfrom(int sockfd, void *buf, size_t len, int flags,
                    struct sockaddr *src_addr, socklen_t *addrlen);
   ssize_t sendto(int sockfd, const void *buf, size_t len, int flags,
                  const struct sockaddr *dest_addr, socklen_t addrlen);
   ```

   

## 六、核心注意事项

1. **字节序转换**：端口和 IP 必须转为网络序（大端），否则跨主机通信会出错；

2. **错误处理**：每个套接字函数都需检查返回值，避免忽略错误；

3. **资源释放**：`close()`必须调用，否则会导致文件描述符泄漏；

4. **阻塞 / 非阻塞**：默认套接字是阻塞模式（`recv()`/`accept()`/`connect()`会等待），可通过`fcntl()`设置为非阻塞；

5. **端口复用**：若服务端重启提示 “地址已被使用”，可设置套接字选项`SO_REUSEADDR`：

   ```c
   int opt = 1;
   setsockopt(listen_fd, SOL_SOCKET, SO_REUSEADDR, &opt, sizeof(opt));
   ```

   

以上是 Socket 编程的核心函数和实战示例，掌握这些后可扩展到多线程服务端、非阻塞 IO、TCP 粘包处理等进阶场景。



## --------------------------------

### 一、先解答 `uint16_t` 是什么

`uint16_t` 是 C/C++ 中**固定宽度的无符号整数类型**，定义在 `<stdint.h>` 头文件中，核心属性如下：

| 特性     | 具体说明                                                     |
| -------- | ------------------------------------------------------------ |
| 全称     | `unsigned int 16 bits`（16 位无符号整数）                    |
| 取值范围 | 0 ~ 65535（2¹⁶ - 1）                                         |
| 设计目的 | 解决传统 `unsigned short` 跨平台长度不一致的问题（比如某些平台 `short` 是 16 位，少数平台是 32 位），`uint16_t` 强制保证 16 位 |
| 典型用途 | 存储 Socket 端口号（16 位）、字节序转换函数（`htons()`/`ntohs()`）的参数 / 返回值 |
| 等价类型 | 在 x86/x64 平台上，`uint16_t` ≈ `unsigned short`，但跨平台场景必须用 `uint16_t` |

**示例**：

```c
#include <stdint.h>
uint16_t port = 8080; // 端口号（16位）
uint16_t net_port = htons(port); // htons() 的参数/返回值都是 uint16_t
```

### 二、核心问题：127.0.0.1 的网络序 `0x0100007F` 是大端还是小端？

你产生困惑的核心是**混淆了「字节级的端序」和「数值本身的进制表示」**，我们一步步拆解：

#### 1. 先明确：大端 / 小端的核心定义

端序（Endianness）是**多字节数据在内存 / 网络传输中的字节排列顺序**，核心规则：

- **大端序（网络字节序）**：**高字节（数值的高位部分）存放在低地址**，低字节存放在高地址（符合人类阅读习惯）；
- **小端序（主机字节序，x86 架构）**：**低字节存放在低地址**，高字节存放在高地址（与人类阅读习惯相反）。

#### 2. 拆解 127.0.0.1 的二进制与字节拆分

IPv4 地址是 32 位整数，`127.0.0.1` 先拆分为**4 个字节**（每个十进制数对应 1 个字节，8 位）：

| 点分十进制     | 127      | 0        | 0        | 1        |
| -------------- | -------- | -------- | -------- | -------- |
| 单字节二进制   | 01111111 | 00000000 | 00000000 | 00000001 |
| 单字节十六进制 | 0x7F     | 0x00     | 0x00     | 0x01     |

#### 3. 大端序（网络序）的存储逻辑

网络字节序要求「高字节在前（低地址），低字节在后（高地址）」：

- `127.0.0.1` 的**数值高位**是第一个字节 `127（0x7F）`，**数值低位**是最后一个字节 `1（0x01）`；
- 大端序下，内存 / 网络传输的字节排列为：`0x7F → 0x00 → 0x00 → 0x01`（低地址 → 高地址）；
- 但 32 位整数的**十六进制整体表示**是「按字节从低地址到高地址拼接」，因此：
  * 低地址字节 `0x01` → 次低 `0x00` → 次高 `0x00` → 高地址字节 `0x7F` → 拼接为 `0x0100007F`。

#### 4. 关键：为什么 `0x0100007F` 是大端，不是小端？

你误以为 `0x0100007F` 是小端，是因为混淆了「十六进制数的书写顺序」和「字节的物理存储顺序」：

| 视角                      | 大端序（网络序）                             | 小端序（x86 主机序）                         |
| ------------------------- | -------------------------------------------- | -------------------------------------------- |
| 字节拆分                  | 0x7F（高字节）、0x00、0x00、0x01（低字节）   | 0x7F（高字节）、0x00、0x00、0x01（低字节）   |
| 内存存储（低地址→高地址） | 0x7F → 0x00 → 0x00 → 0x01                    | 0x01 → 0x00 → 0x00 → 0x7F                    |
| 32 位整数十六进制表示     | 0x0100007F（按内存地址拼接：低地址字节在前） | 0x7F000001（按内存地址拼接：低地址字节在前） |

**通俗举例**：

把 32 位整数想象成「4 个抽屉（地址），从左到右是低地址→高地址」：

- 大端序：第一个抽屉（低地址）放最高位字节 `0x7F`，最后一个抽屉（高地址）放最低位字节 `0x01`；
- 现在把抽屉里的内容「从左到右（低地址→高地址）」读出来拼接成十六进制：`0x01（最后一个抽屉）00 00 7F（第一个抽屉）` → 即 `0x0100007F`；
- 小端序则是第一个抽屉放 `0x01`，最后一个放 `0x7F`，拼接为 `0x7F000001`。

#### 5. 验证：`127.0.0.1` 的网络序与主机序转换

```c
#include <stdio.h>
#include <stdint.h>
#include <arpa/inet.h>

int main() {
    // 127.0.0.1 的主机序（x86 小端）十六进制：0x7F000001
    uint32_t host_ip = 0x7F000001;
    // 转换为网络序（大端）
    uint32_t net_ip = htonl(host_ip);
    
    printf("主机序：0x%08X\n", host_ip);  // 输出：0x7F000001（小端）
    printf("网络序：0x%08X\n", net_ip);  // 输出：0x0100007F（大端）
    
    // 网络序转回点分十进制
    struct in_addr addr;
    addr.s_addr = net_ip;
    printf("点分十进制：%s\n", inet_ntoa(addr)); // 输出：127.0.0.1
    return 0;
}
```

### 三、总结

1. `uint16_t` 是「固定 16 位无符号整数」，解决跨平台类型长度不一致问题，专用于 16 位数据（如端口号）；
2. 端序的核心是「多字节的存储顺序」，而非十六进制数的书写顺序；
3. `127.0.0.1` 的网络序（大端）存储为 `0x0100007F`，是因为：
   - 大端序下内存字节排列为 `0x7F → 0x00 → 0x00 → 0x01`（低地址→高地址）；
   - 32 位整数的十六进制表示是「按内存地址从低到高拼接字节」，因此得到 `0x0100007F`；
   - 若用小端序存储，内存字节排列为 `0x01 → 0x00 → 0x00 → 0x7F`，拼接为 `0x7F000001`（主机序）。

简单记：**网络序（大端）= 人类阅读顺序（高位在前），主机序（小端）= 反人类顺序（低位在前）**，而十六进制数的整体表示是「内存地址从低到高的字节拼接」，因此大端序的 127.0.0.1 会显示为 `0x0100007F`。