# EPollPoller 类成员变量与成员函数作用总结

## 一、成员变量

|      变量名      |                       类型 / 相关常量                        |                             作用                             |                      初始化 / 赋值时机                       |
| :--------------: | :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
|    `epollfd_`    |        `int`（由`epoll_create1(EPOLL_CLOEXEC)`创建）         | 表示 epoll 实例的文件描述符，是与内核 epoll 机制交互的核心句柄 | 构造函数中初始化，若创建失败（`epollfd_ < 0`）会触发`LOG_FATAL`日志并终止程序；析构函数中调用`::close(epollfd_)`关闭该句柄 |
|    `events_`     | `std::vector<epoll_event>`（初始大小`kInitEventListSize`，默认 16） | 存储`epoll_wait`返回的就绪事件，是内核与用户态传递事件的缓冲区 | 构造函数中初始化为容量 16 的 vector；`poll`函数中若就绪事件数`numEvents == events_.size()`，会扩容为原大小的 2 倍（避免频繁扩容损耗性能） |
|   `channels_`    | （隐含，代码中为`std::map<int, Channel*>`，key 为 fd，value 为 Channel 指针） | 维护 fd 与 Channel 的映射关系，方便通过 fd 快速找到对应的 Channel 对象 | `updateChannel`函数中，当 Channel 状态为`kNew`时，将`channel->fd()`作为 key、Channel 指针作为 value 存入；`removeChannel`函数中，删除 Channel 时<u>**先**从`channels_`中`erase`对应 fd 的映射</u>（防止野指针） |
| 状态常量（全局） |              `kNew=-1`/`kAdded=1`/`kDeleted=2`               | 标记 Channel 在 Poller 中的状态：- `kNew`：Channel 未添加至 Poller（Channel 的`index_`初始值）- `kAdded`：Channel 已添加至 Poller- `kDeleted`：Channel 已从 Poller 删除 | 仅作为状态标记，在`updateChannel`/`removeChannel`中修改 Channel 的`index_`属性时使用 |
|   `ownerLoop_`   |                基类所属事件循环，`EventLoop`                 |              定义Poller所属的事件循环EventLoop               |                 列表初始化`ownerLoop_(loop)`                 |

## 二、成员函数

按功能分类，并明确调用条件、时机与核心逻辑：

### （一）构造 / 析构函数：生命周期管理

|             函数名             |                      调用时机                      |                           核心作用                           |                        注意事项                        |
| :----------------------------: | :------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------: |
| `EPollPoller(EventLoop *loop)` | 创建 EPollPoller 对象时（由 EventLoop 初始化触发） | 1. 调用父类 Poller 的构造函数，关联所属的 EventLoop；2. 创建 epoll 实例（`epoll_create1(EPOLL_CLOEXEC)`）并赋值给`epollfd_`；3. 初始化`events_`为默认大小的事件缓冲区；4. 若 epoll 实例创建失败，触发 FATAL 日志终止程序 | `EPOLL_CLOEXEC`确保进程 exec 时关闭该 fd，避免资源泄漏 |
|        `~EPollPoller()`        |   EPollPoller 对象销毁时（EventLoop 销毁时触发）   |        关闭`epollfd_`，释放 epoll 实例对应的内核资源         |       仅执行`::close(epollfd_)`，无其他额外逻辑        |

### （二）核心事件等待：poll 函数

|                       函数名                       |                  调用时机                  |                    调用条件                     |                           核心作用                           |                           分支逻辑                           |
| :------------------------------------------------: | :----------------------------------------: | :---------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| `poll(int timeoutMs, ChannelList *activeChannels)` | EventLoop 的事件循环中（每次循环都会调用） | 无前置条件，是 EventLoop 获取就绪事件的核心接口 | 1. 调用`epoll_wait`等待 epoll 实例上的就绪事件，超时时间为`timeoutMs`；2. 收集就绪事件并填充到`activeChannels`；3. 处理`epoll_wait`的返回值（正常 / 超时 / 错误）；4. 返回当前时间戳供 EventLoop 使用 | - 若`numEvents > 0`：调用`fillActiveChannels`填充活跃 Channel，若事件数等于缓冲区大小则扩容`events_`；- 若`numEvents == 0`：记录 DEBUG 日志（超时，无错误）；- 若`numEvents < 0`：仅当错误码非`EINTR`（信号中断）时记录 ERROR 日志（容错设计，不终止事件循环） |

### （三）Channel 管理：updateChannel/removeChannel

#### 1. updateChannel（更新 Channel 状态）

|                           调用时机                           |                   调用条件                    |                           核心作用                           |                           分支逻辑                           |
| :----------------------------------------------------------: | :-------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| EventLoop 调用`updateChannel`时触发（Channel 关注的事件变化 / 首次添加 / 重新添加） | 无前置条件，是 Poller 管理 Channel 的核心接口 | 1. 根据 Channel 的当前状态（`index_`）决定 epoll 操作类型（ADD/MOD/DEL）；2. 维护`channels_`映射与 Channel 的`index_`状态；3. 调用`update`函数执行实际的`epoll_ctl`操作 | ✅ 对「新注册 / 重注册」的 Channel，执行 ADD 操作并标记状态；✅  对「已注册但无事件」的 Channel，执行 DEL 操作并标记删除状态；✅ 对「已注册且事件变更」的 Channel，执行 MOD 操作保留注册状态； |

#### 2. removeChannel（删除 Channel）

|                           调用时机                           |                   调用条件                   |                           核心作用                           |                           分支逻辑                           |
| :----------------------------------------------------------: | :------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| EventLoop 调用`removeChannel`时触发（Channel 不再使用 / 连接关闭） | 无前置条件，用于彻底移除 Poller 中的 Channel | 1. 从`channels_`中删除 fd 对应的 Channel 映射（优先删除，防止野指针）；2. 若 Channel 状态为`kAdded`，调用`update(EPOLL_CTL_DEL)`从 epoll 实例中删除 fd；3. 重置 Channel 的`index_`为`kNew` | 仅当`index == kAdded`时执行`epoll_ctl DEL`操作（避免对未添加的 fd 执行删除）；无论是否执行 DEL，都会重置`index_`为初始状态 |

### （四）辅助函数：fillActiveChannels/update

#### 1. fillActiveChannels（填充活跃 Channel）

|                           调用时机                           |       调用条件       |                           核心作用                           |                             细节                             |
| :----------------------------------------------------------: | :------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| `poll`函数中，当`epoll_wait`返回的就绪事件数`numEvents > 0`时调用 | 仅在有就绪事件时触发 | 1. 遍历`events_`中的就绪事件；2. 从`epoll_event`的`data.ptr`取出 Channel 指针；3. 设置 Channel 的`revents_`（就绪事件类型）；4. 将 Channel 添加到`activeChannels`（输出参数，供 EventLoop 处理） | 利用`epoll_event`的联合体`data.ptr`存储 Channel 指针，避免 fd 与 Channel 的转换开销，是高效设计的关键 |

#### 2. update（执行 epoll_ctl 操作）

|                           调用时机                           |                       调用条件                       |                           核心作用                           |                           错误处理                           |
| :----------------------------------------------------------: | :--------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| `updateChannel`/`removeChannel`中，需要执行 ADD/MOD/DEL 操作时调用 | 由上层 Channel 管理函数触发，是底层 epoll 操作的封装 | 1. 初始化`epoll_event`结构体（清零，避免内存残留）；2. 设置事件类型（`channel->events()`）和关联的 Channel 指针（`data.ptr`）；3. 调用`epoll_ctl`操作 epoll 实例 | - 若操作是`EPOLL_CTL_DEL`失败：记录 ERROR 日志（容错）；- 若操作是 ADD/MOD 失败：记录 FATAL 日志并终止程序（核心操作不可失败） |

## 三、核心调用链路与时机总结

1. **初始化阶段**：EventLoop 创建 EPollPoller → 构造函数创建 epollfd、初始化 events_ → 完成 Poller 初始化；
2. **事件循环阶段**：EventLoop 每次循环调用 poll () → epoll_wait 等待事件 → 有就绪事件则填充 activeChannels → 无事件则处理超时 / 错误；
3. **Channel 状态变更**：
   - 首次添加 Channel：updateChannel（kNew）→ 执行 ADD → 状态改为 kAdded；
   - 修改 Channel 事件：updateChannel（kAdded）→ 执行 MOD；
   - 取消 Channel 所有事件：updateChannel（kAdded）→ 执行 DEL → 状态改为 kDeleted；
   - 重新添加已删除的 Channel：updateChannel（kDeleted）→ 执行 ADD → 状态改为 kAdded；
   - 彻底删除 Channel：removeChannel → 从 channels_删除映射 → 若为 kAdded 则执行 DEL → 状态改为 kNew；
4. **销毁阶段**：EventLoop 销毁 → EPollPoller 析构 → 关闭 epollfd → 释放资源。

## 四、设计亮点与容错逻辑

1. **状态管理**：通过 kNew/kAdded/kDeleted 标记 Channel 状态，避免重复添加 / 删除 fd；
2. **内存安全**：removeChannel 时先删除 channels_映射，防止后续访问野指针；
3. **扩容策略**：events_满时扩容为 2 倍，平衡内存占用与扩容开销；
4. **容错设计**：epoll_wait 被信号中断（EINTR）不报错，DEL 操作失败仅打日志，ADD/MOD 失败终止程序（核心操作必须保证）。