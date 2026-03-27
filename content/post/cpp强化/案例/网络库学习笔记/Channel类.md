## Channel 类成员变量与成员函数作用总结

### 一、成员变量（按功能分类）

|     变量名     |         类型 / 常量值          |                           核心作用                           |                      初始化 / 赋值时机                       |
| :------------: | :----------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
|    常量成员    |                                |                                                              |                                                              |
|   kNoneEvent   |         const int = 0          |        标识 “无事件” 状态，作为事件掩码的初始 / 空值         |                 类静态常量，程序启动时初始化                 |
|   kReadEvent   | const int = EPOLLIN\|EPOLLPRI  |   封装读事件（普通读 + 高优先级读）的掩码，统一读事件标识    |      类静态常量，程序启动时初始化（依赖 epoll 宏定义）       |
|  kWriteEvent   |      const int = EPOLLOUT      |               封装写事件的掩码，统一写事件标识               |                 类静态常量，程序启动时初始化                 |
|    实例成员    |                                |                                                              |                                                              |
|     loop_      |           EventLoop*           |       指向 Channel 所属的 EventLoop 对象，关联事件循环       |         构造函数初始化（由外部传入 EventLoop 指针）          |
|      fd_       |              int               | Channel 关联的文件描述符（如 socket fd），是事件管理的核心载体 |               构造函数初始化（由外部传入 fd）                |
|    events_     |              int               | 记录当前 Channel 需要监听的事件掩码（如读 / 写事件），供 Poller 使用 | 构造函数初始化为 0，后续通过 EventLoop/Poller 修改（如注册读 / 写事件时） |
|    revents_    |              int               | 记录 Poller 检测到的、fd 实际触发的事件掩码（如 EPOLLIN/EPOLLERR 等） | 构造函数初始化为 0，Poller（如 EpollPoller）调用 epoll_wait 后，为 Channel 赋值该变量 |
|     index_     |              int               | 标记 Channel 在 Poller 中的状态（如是否已注册、待删除等），初始值 - 1 | 构造函数初始化为 - 1，Poller（如 EpollPoller）调用 epoll_ctl （updateChannel等）时更新 |
|     tied_      |              bool              | 标记 Channel 是否绑定了共享对象（如 TcpConnection），控制生命周期检查逻辑 |     构造函数初始化为 false，调用 tie () 方法时设为 true      |
|      tie_      |      std::weak_ptr<void>       | 弱引用绑定的共享对象，用于检测绑定对象（如 TcpConnection）是否存活 | 构造函数初始化为空，调用 tie () 方法时赋值（接收外部传入的 shared_ptr 并转为 weak_ptr） |
|    回调成员    |                                |                                                              |                                                              |
| closeCallback_ |     std::function<void()>      |            关闭事件回调函数，触发 EPOLLHUP 时执行            |          外部（如 TcpConnection）按需设置，无默认值          |
| errorCallback_ |     std::function<void()>      |            错误事件回调函数，触发 EPOLLERR 时执行            |          外部（如 TcpConnection）按需设置，无默认值          |
| readCallback_  | std::function<void(Timestamp)> |  读事件回调函数，触发 EPOLLIN/EPOLLPRI 时执行（携带时间戳）  |          外部（如 TcpConnection）按需设置，无默认值          |
| writeCallback_ |     std::function<void()>      |             写事件回调函数，触发 EPOLLOUT 时执行             |          外部（如 TcpConnection）按需设置，无默认值          |

### 二、成员函数（按调用条件 / 时机分类）

#### 1. 构造 / 析构函数

|           函数名            |                           核心作用                           |                       调用条件 / 时机                        |
| :-------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| Channel(EventLoop*, int fd) | 初始化 Channel 的核心关联关系（所属 EventLoop、关联 fd），初始化事件 / 状态变量 | 创建 Channel 对象时调用（如 TcpConnection 创建时初始化对应 fd 的 Channel、EventLoop 创建 listenfd 的 Channel） |
|         ~Channel()          |                      析构函数（空实现）                      | Channel 对象销毁时调用（因未手动分配堆资源，无需释放）；绑定 TcpConnection 时，通过 tie 机制保证 TcpConnection 先销毁 |

#### 2. 生命周期绑定函数

|              函数名               |                           核心作用                           |                       调用条件 / 时机                        |
| :-------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| tie(const std::shared_ptr<void>&) | 绑定共享对象（如 TcpConnection），通过 weak_ptr 弱引用避免循环引用，同时标记 tied_为 true | 仅在 Channel 关联 “有生命周期依赖的对象” 时调用（如 TcpConnection 初始化 Channel 时，绑定自身 shared_ptr）；非所有 Channel 都调用（如 listenfd 的 Channel 无需绑定） |

#### 3. Poller 事件管理函数

|  函数名  |                           核心作用                           |                       调用条件 / 时机                        |
| :------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| update() | 通知所属 EventLoop，调用 Poller 的 epoll_ctl 更新当前 fd 的监听事件（events_） | 当 Channel 的监听事件（events_）发生变化时调用（如注册读事件、添加写事件、移除事件）；例如 TcpConnection 建立后注册读事件、写缓冲区非空时注册写事件 |
| remove() | 通知所属 EventLoop，调用 Poller 的 epoll_ctl 从 epoll 实例中移除当前 fd 的监听 | 当 Channel 关联的 fd 不再需要监听时调用（如 TcpConnection 断开连接、关闭 listenfd） |

#### 4. 事件处理核心函数

|             函数名              |                           核心作用                           |                       调用条件 / 时机                        |
| :-----------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
|     handleEvent(Timestamp)      |   事件处理入口，先检查绑定对象是否存活，再分发事件处理逻辑   | EventLoop 的事件循环中，Poller（如 EpollPoller）调用 epoll_wait 获取触发事件后，遍历 Channel 列表调用该函数；入参为事件触发的时间戳 |
| handleEventWithGuard(Timestamp) | 实际的事件处理逻辑，按事件类型（关闭 / 错误 / 读 / 写）调用对应回调 | 1. 未绑定对象（如 listenfd 的 Channel）：直接由 handleEvent 调用；2. 已绑定对象：handleEvent 中 weak_ptr 提升为 shared_ptr 成功（绑定对象存活）时调用 |

#### 5. 事件回调触发逻辑（handleEventWithGuard 内部分支）

| 事件类型 |             触发条件              |                         回调调用逻辑                         |
| :------: | :-------------------------------: | :----------------------------------------------------------: |
| 关闭事件 | revents_ & EPOLLHUP 且 无 EPOLLIN | 触发场景：TcpConnection 关闭写端（shutdown）导致 epoll 触发 EPOLLHUP；若设置了 closeCallback_则调用 |
| 错误事件 |        revents_ & EPOLLERR        | 触发场景：fd 发生错误（如连接重置、fd 无效）；若设置了 errorCallback_则调用 |
|  读事件  |  revents_ & (EPOLLIN\|EPOLLPRI)   | 触发场景：fd 有数据可读（普通数据 / 高优先级数据）；若设置了 readCallback_则调用（传递事件时间戳） |
|  写事件  |        revents_ & EPOLLOUT        | 触发场景：fd 可写（如写缓冲区空闲）；若设置了 writeCallback_则调用 |

### 三、核心逻辑补充

1. 成员变量的依赖关系：`revents_`由 Poller 赋值，`events_`由外部设置后通过`update()`同步到 Poller，两者共同驱动`handleEvent`的逻辑；
2. 回调函数的特点：所有回调均由外部（如 TcpConnection）设置，Channel 仅负责 “触发”，不负责 “实现”，符合回调模式的解耦设计；
3. tie 机制的核心价值：通过 weak_ptr 检测绑定对象（如 TcpConnection）是否存活，避免在对象已销毁时调用回调，防止野指针访问。