### 二、C++ TCP 客户端 & 服务端通信示例

#### 1. 服务端代码（TCP_Server.cpp）

```cpp
#include <iostream>
#include <cstring>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <unistd.h>

#define PORT 8888
#define BUFFER_SIZE 1024

int main() {
    // 1. 创建TCP套接字（AF_INET=IPv4，SOCK_STREAM=TCP，0=默认协议）
    int server_fd = socket(AF_INET, SOCK_STREAM, 0);
    if (server_fd < 0) {
        perror("socket creation failed");
        exit(EXIT_FAILURE);
    }

    // 2. 设置套接字选项（复用端口/地址，避免重启服务报错）
    int opt = 1;
    if (setsockopt(server_fd, SOL_SOCKET, SO_REUSEADDR | SO_REUSEPORT, &opt, sizeof(opt))) {
        perror("setsockopt failed");
        close(server_fd);
        exit(EXIT_FAILURE);
    }

    // 3. 绑定端口和IP
    struct sockaddr_in server_addr;
    memset(&server_addr, 0, sizeof(server_addr));
    server_addr.sin_family = AF_INET;         // IPv4
    server_addr.sin_addr.s_addr = INADDR_ANY; // 监听所有网卡
    server_addr.sin_port = htons(PORT);       // 端口转换为网络字节序

    if (bind(server_fd, (struct sockaddr*)&server_addr, sizeof(server_addr)) < 0) {
        perror("bind failed");
        close(server_fd);
        exit(EXIT_FAILURE);
    }

    // 4. 监听连接（第二个参数=最大等待队列长度）
    if (listen(server_fd, 5) < 0) {
        perror("listen failed");
        close(server_fd);
        exit(EXIT_FAILURE);
    }
    std::cout << "Server listening on port " << PORT << "..." << std::endl;

    // 5. 接受客户端连接（阻塞等待）
    struct sockaddr_in client_addr;
    socklen_t client_addr_len = sizeof(client_addr);
    int new_socket = accept(server_fd, (struct sockaddr*)&client_addr, &client_addr_len);
    if (new_socket < 0) {
        perror("accept failed");
        close(server_fd);
        exit(EXIT_FAILURE);
    }
    std::cout << "Client connected: " << inet_ntoa(client_addr.sin_addr) << ":" 
              << ntohs(client_addr.sin_port) << std::endl;

    // 6. 数据收发
    char buffer[BUFFER_SIZE] = {0};
    ssize_t valread;
    while (true) {
        // 读取客户端数据
        valread = read(new_socket, buffer, BUFFER_SIZE);
        if (valread <= 0) {
            if (valread == 0) {
                std::cout << "Client disconnected" << std::endl;
            } else {
                perror("read failed");
            }
            break;
        }
        std::cout << "Received from client: " << buffer << std::endl;

        // 回复客户端
        std::string response = "Server received: " + std::string(buffer);
        send(new_socket, response.c_str(), response.length(), 0);
        memset(buffer, 0, BUFFER_SIZE); // 清空缓冲区
    }

    // 7. 关闭套接字
    close(new_socket);
    close(server_fd);
    return 0;
}
```

#### 2. 客户端代码（TCP_Client.cpp）

```cpp
#include <iostream>
#include <cstring>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <unistd.h>

#define SERVER_IP "127.0.0.1" // 服务端IP（本地测试用回环地址）
#define PORT 8888
#define BUFFER_SIZE 1024

int main() {
    // 1. 创建TCP套接字
    int client_fd = socket(AF_INET, SOCK_STREAM, 0);
    if (client_fd < 0) {
        perror("socket creation failed");
        exit(EXIT_FAILURE);
    }

    // 2. 配置服务端地址
    struct sockaddr_in server_addr;
    memset(&server_addr, 0, sizeof(server_addr));
    server_addr.sin_family = AF_INET;
    server_addr.sin_port = htons(PORT);

    // 转换IP为网络字节序
    if (inet_pton(AF_INET, SERVER_IP, &server_addr.sin_addr) <= 0) {
        perror("invalid server IP address");
        close(client_fd);
        exit(EXIT_FAILURE);
    }

    // 3. 连接服务端（触发三次握手）
    if (connect(client_fd, (struct sockaddr*)&server_addr, sizeof(server_addr)) < 0) {
        perror("connection failed");
        close(client_fd);
        exit(EXIT_FAILURE);
    }
    std::cout << "Connected to server successfully!" << std::endl;

    // 4. 发送数据到服务端
    std::string message;
    char buffer[BUFFER_SIZE] = {0};
    while (true) {
        std::cout << "Enter message (q to quit): ";
        std::getline(std::cin, message);
        if (message == "q") break;

        // 发送数据
        send(client_fd, message.c_str(), message.length(), 0);

        // 接收服务端回复
        ssize_t valread = read(client_fd, buffer, BUFFER_SIZE);
        if (valread <= 0) {
            perror("read failed or server disconnected");
            break;
        }
        std::cout << "Server response: " << buffer << std::endl;
        memset(buffer, 0, BUFFER_SIZE);
    }

    // 5. 关闭套接字（触发四次挥手）
    close(client_fd);
    std::cout << "Connection closed" << std::endl;
    return 0;
}
```

#### 3. 编译 & 运行说明

- **编译命令**（Linux/macOS 环境）：

  ```bash
  # 编译服务端
  g++ TCP_Server.cpp -o tcp_server
  # 编译客户端
  g++ TCP_Client.cpp -o tcp_client
  ```

  

- **运行步骤**：

  1. 先启动服务端：`./tcp_server`
  2. 再启动客户端：`./tcp_client`
  3. 客户端输入任意字符串，服务端接收后回复，输入`q`退出客户端。

#### 4. 核心说明

- 服务端的`accept()`是阻塞函数，直到有客户端连接；
- 客户端的`connect()`会触发 TCP 三次握手，成功后进入`ESTABLISHED`状态；
- `read()`返回 0 表示对方关闭连接（触发四次挥手）；
- 代码基于 POSIX 标准，Windows 环境需替换为 Winsock API（如`socket()`→`WSASocket()`、`close()`→`closesocket()`等）。

----

# 好像有点问题，问题不大，到时候看看👀