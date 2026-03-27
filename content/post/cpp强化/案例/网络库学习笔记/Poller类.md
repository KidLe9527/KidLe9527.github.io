### Poller 类 成员变量 & 成员函数 作用总结

#### 一、成员变量（按访问权限分类）

| 变量名     | 访问权限  | 类型                                       | 核心作用                                                     | 调用 / 生效时机                                              |
| ---------- | --------- | ------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| channels_  | protected | ChannelMap（unordered_map<int, Channel*>） | 存储当前 Poller 管理的所有 Channel，key 为 Socket 文件描述符（fd），value 为对应 Channel 指针 | 1. Poller 初始化时为空；2. 调用 updateChannel 添加 / 更新 Channel 时修改；3. 调用 removeChannel 删除 Channel 时修改；4. poll () 遍历活跃 fd 时查询对应 Channel |
| ownerLoop_ | private   | EventLoop*                                 | 指向当前 Poller 所属的 EventLoop 对象，标识 Poller 归属的事件循环 | 1. Poller 构造时初始化（传入 loop 参数）；2. 所有成员函数执行时，均依赖该变量确认所属事件循环（如判断 Channel 是否属于当前 EventLoop） |

#### 二、成员函数（按功能 & 调用时机分类）

##### 1. 构造 / 析构函数

| 函数签名                    | 类型       | 核心作用                                                     | 调用 / 生效时机                                              |
| --------------------------- | ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Poller(EventLoop *loop)     | 构造函数   | 初始化 Poller 对象，设置 ownerLoop_为传入的 EventLoop 指针，初始化 channels_为空 | 1. 派生类（EPollPoller/PollPoller）构造时调用；2. 通常由 newDefaultPoller () 间接触发（创建 Poller 实例时） |
| virtual ~Poller() = default | 虚析构函数 | 确保派生类析构时能正确调用基类析构函数，避免内存泄漏         | 1. Poller 对象销毁时调用；2. 因是虚函数，派生类对象析构时会先调用自身析构，再调用基类（Poller）析构 |

##### 2. 纯虚函数（IO 复用核心接口，由派生类实现）

| 函数签名                                                     | 类型     | 核心作用                                                     | 调用 / 生效时机                                              |
| ------------------------------------------------------------ | -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| virtual Timestamp poll(int timeoutMs, ChannelList *activeChannels) = 0 | 纯虚函数 | 等待 IO 事件发生，超时时间为 timeoutMs（毫秒）；将发生事件的 Channel 添加到 activeChannels 中，返回事件发生的时间戳 | 1. 由所属 EventLoop 的 loop () 循环调用（EventLoop 的核心事件循环逻辑中）；2. 每次事件循环迭代时执行，阻塞等待 fd 就绪，是 IO 复用的核心入口；3. 触发时机：EventLoop 需要检测 IO 事件时（如处理完当前任务后，等待新事件） |
| virtual void updateChannel(Channel *channel) = 0             | 纯虚函数 | 添加 / 更新 Channel 到 Poller 的管理列表（channels_），并同步到底层 IO 复用机制（epoll_ctl/pollfd） | 1. Channel 的 fd、关注事件（read/write）发生变化时调用（如 Channel::enableReading () 后）；2. EventLoop::updateChannel () 会转发调用该函数；3. 时机：Channel 需要注册 / 更新 IO 事件时 |
| virtual void removeChannel(Channel *channel) = 0             | 纯虚函数 | 从 Poller 的管理列表（channels_）中删除 Channel，并从底层 IO 复用机制中注销该 fd | 1. Channel 不再需要关注 IO 事件时调用（如连接关闭、Channel 析构前）；2. EventLoop::removeChannel () 会转发调用该函数；3. 时机：Channel 需要注销 IO 事件时 |

##### 3. 普通成员函数（工具类 / 工厂类）

| 函数签名                                         | 类型         | 核心作用                                                     | 调用 / 生效时机                                              |
| ------------------------------------------------ | ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| bool hasChannel(Channel *channel) const          | 普通成员函数 | 判断传入的 Channel 是否在当前 Poller 的管理列表（channels_）中 | 1. updateChannel/removeChannel 前校验（避免重复添加 / 删除）；2. EventLoop 或 Channel 校验归属时调用；3. 时机：操作 Channel 前的合法性检查阶段 |
| static Poller *newDefaultPoller(EventLoop *loop) | 静态成员函数 | 工厂方法，根据系统环境选择默认的 Poller 实现（Linux 下优先 EPollPoller，否则 PollPoller） | 1. EventLoop 构造时调用，创建所属的 Poller 实例；2. 唯一触发时机：EventLoop 初始化时，需要获取 IO 复用器实例时 |

### 补充说明

1. 调用链路：EventLoop 通过 newDefaultPoller () 创建 Poller 实例 → 事件循环中调用 Poller::poll () 等待 IO 事件 → 事件就绪后通过 updateChannel/removeChannel 维护 Channel；
2. 设计逻辑：Poller 作为基类定义统一接口，派生类（EPollPoller/PollPoller）实现具体 IO 复用逻辑，通过多态保证上层调用统一；
3. 核心依赖：所有成员函数的执行均依赖 ownerLoop_确认所属事件循环，channels_作为 Channel 的核心存储容器，支撑所有 IO 事件的注册、查询、注销操作。

```cpp
// 默认构造的简单实现，一般不会放在抽象类实现。避免基类依赖派生类，保持抽象模块的纯净性；
Poller* Poller::newDefaultPoller(EventLoop* loop) {
    // 1. 检测系统是否支持 epoll（Linux 特有）
    // _SC_EPOLL 是系统配置项，通过 sysconf 获取
    if (::sysconf(_SC_EPOLL)) { 
        // 优先返回高性能的 EPollPoller 实例
        return new EPollPoller(loop);
    } else {
        // 降级为 PollPoller（兼容所有 POSIX 系统）
        return new PollPoller(loop);
    }
}
```

