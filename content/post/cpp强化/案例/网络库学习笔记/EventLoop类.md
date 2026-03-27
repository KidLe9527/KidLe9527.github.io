## EventLoop 类 成员变量 & 成员函数 作用总结

### 一、成员变量

#### 1. 原子布尔型（线程安全标识）

|          变量名           |                 作用                 |                       调用 / 生效时机                        |
| :-----------------------: | :----------------------------------: | :----------------------------------------------------------: |
|        `looping_`         | 标识 EventLoop 是否正在运行事件循环  | 调用`loop()`时置为`true`，调用`quit()`并完成循环退出后置为`false`；整个`loop()`循环过程中持续校验该值，若为`false`则退出循环 |
|          `quit_`          |       标识是否需要退出事件循环       | 调用`quit()`时置为`true`；`loop()`循环内每次完成`poller_->poll()`后，会校验该值，若为`true`则终止循环 |
| `callingPendingFunctors_` | 标识当前是否正在执行待处理的回调函数 | 进入`doPendingFunctors()`时置为`true`，执行完所有待处理回调后置为`false`；用于防止回调执行过程中重复触发或竞态问题 |

#### 2. 线程标识相关

|   变量名    |                 作用                 |                       调用 / 生效时机                        |
| :---------: | :----------------------------------: | :----------------------------------------------------------: |
| `threadId_` | 记录创建当前 EventLoop 对象的线程 ID | EventLoop 构造时初始化；调用`isInLoopThread()`时，会将该值与当前线程 ID（`CurrentThread::tid()`）对比，判断是否在 EventLoop 所属线程 |

#### 3. 时间戳相关

|      变量名       |                       作用                       |                       调用 / 生效时机                        |
| :---------------: | :----------------------------------------------: | :----------------------------------------------------------: |
| `pollRetureTime_` | 记录 Poller 检测到有事件发生的 Channels 的时间点 | 每次调用`poller_->poll()`（在`loop()`循环内）返回时，用返回的时间戳更新该值；调用`pollReturnTime()`时返回该时间戳 |

#### 4. Poller 与 Channel 相关

|      变量名       |                             作用                             |                       调用 / 生效时机                        |
| :---------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
|     `poller_`     | 指向 Poller 对象（epoll 抽象）的智能指针，是 EventLoop 处理事件的核心依赖 | EventLoop 构造时初始化；`loop()`循环内持续调用其`poll()`方法检测事件，`updateChannel()`/`removeChannel()`/`hasChannel()`会转发调用`poller_`的对应方法 |
| `activeChannels_` |    存储 Poller 检测到的当前有事件发生的所有 Channel 指针     | 每次`poller_->poll()`返回时，会将有事件的 Channel 列表赋值给该变量；`loop()`循环内遍历该列表，调用 Channel 的事件处理方法 |
|    `wakeupFd_`    | 事件文件描述符（eventfd），用于唤醒阻塞在`epoll_wait`的 EventLoop 所属线程 | EventLoop 构造时创建；`wakeup()`时向该 fd 写入数据触发事件，`handleRead()`时读取该 fd 的数据 |
| `wakeupChannel_`  | 绑定`wakeupFd_`的 Channel 智能指针，用于<u>**监听**</u>`wakeupFd_`的读事件（只监听，不存储返回的任务） | EventLoop 构造时初始化，将`wakeupFd_`的读事件绑定到`handleRead()`回调；`loop()`循环内检测到`wakeupFd_`有读事件时，触发该 Channel 的回调 |

#### 5. 回调函数队列相关

|       变量名       |                             作用                             |                       调用 / 生效时机                        |
| :----------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| `pendingFunctors_` |        存储 EventLoop 需要异步执行的所有上层回调函数         | 调用`queueInLoop()`时，将回调函数添加到该容器；调用`doPendingFunctors()`时，遍历并执行容器内的回调，执行后清空 |
|      `mutex_`      | 保护`pendingFunctors_`的互斥锁，保证多线程操作该容器的线程安全 | 调用`queueInLoop()`（添加回调）和`doPendingFunctors()`（读取 / 清空回调）时，需先加锁，操作完成后解锁 |

> `pendingFunctors_`本质是 `EventLoop` 的**异步任务缓冲区**，解决了 “跨线程向 IO 线程提交任务” 的核心问题，结合 `eventfd` 的 `wakeup` 机制，既保证了任务执行的线程安全，又保证了任务执行的及时性。存储来自其他线程向本线程发起的回调函数

### 二、成员函数

#### 1. 构造 / 析构函数

|     函数名     |                             作用                             |                         调用时机                         |
| :------------: | :----------------------------------------------------------: | :------------------------------------------------------: |
| `EventLoop()`  | 初始化 EventLoop 对象：创建`wakeupFd_`、初始化`wakeupChannel_`、`poller_`，记录`threadId_`，初始化各原子变量和容器  ✅**创建了一个只属于当前线程、可以被跨线程唤醒、能监听所有 IO 事件的事件循环。** | 创建 EventLoop 实例时调用（如 mainLoop/subLoop 初始化）  |
| `~EventLoop()` | 释放 EventLoop 相关资源（如`poller_`、`wakeupChannel_`等智能指针自动析构） | EventLoop 实例生命周期结束时调用（如线程退出、对象销毁） |

> * 构造函数需要为wakefd_以及其channel设置监听听事件，设置回调读函数
> * 析构函数调用必须先将wakefd_的channel移除监听事件（防止 Poller 还在监听已销毁的 Channel），再从Poller中移除，然后关闭wakefd_，最后解绑EventLoop和关联线程：先停止事件 → 再移除监听 → 再关 fd → 最后解绑线程

#### 2. 事件循环核心控制

|  函数名  |                 作用                  |                       调用时机                        |                   内部逻辑 / 触发后续操作                    |
| :------: | :-----------------------------------: | :---------------------------------------------------: | :----------------------------------------------------------: |
| `loop()` | 开启事件循环，是 EventLoop 的核心入口 |    线程启动后主动调用（如 subLoop 线程启动后调用）    | 1. 校验是否在所属线程，若否则终止；2. 置`looping_`为`true`、`quit_`为`false`；3. 循环调用`poller_->poll()`获取活跃 Channel；4. 处理活跃 Channel 事件；5. 调用`doPendingFunctors()`执行待处理回调；6. 校验`quit_`，若为`true`则退出循环 |
| `quit()` |           触发事件循环退出            | 外部需要终止 EventLoop 时调用（如程序退出、线程停止） | 置`quit_`为`true`；若当前调用线程不是 EventLoop 所属线程，调用`wakeup()`唤醒阻塞的`poll()`，确保循环能检测到`quit_`为`true` |

#### 3. 回调函数执行控制

|          函数名           |                           作用                            |                           调用时机                           |                   内部逻辑 / 触发后续操作                    |
| :-----------------------: | :-------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
|  `runInLoop(Functor cb)`  |           在当前 EventLoop 所属线程执行回调函数           |           上层需要在 EventLoop 线程执行任务时调用            | 1. 若当前线程是 EventLoop 所属线程，直接执行`cb`；2. 若不是，调用`queueInLoop(cb)`将回调加入队列，等待 EventLoop 线程执行 |
| `queueInLoop(Functor cb)` | 将回调函数加入待执行队列，满足一定条件唤醒 EventLoop 线程 | 1. `runInLoop()`检测到跨线程时调用；2. 外部需要异步提交任务到 EventLoop 线程时直接调用 | 1. 加锁保护`pendingFunctors_`，将`cb`加入队列；2. 若当前不是 EventLoop 所属线程，或正在执行回调（`callingPendingFunctors_`为`true`），调用`wakeup()`唤醒 EventLoop，确保回调能被及时执行 |
|   `doPendingFunctors()`   |          执行`pendingFunctors_`中的所有回调函数           |        `loop()`循环内，处理完活跃 Channel 事件后调用         | 1. 加锁拷贝`pendingFunctors_`到局部变量，清空原队列，解锁；2. 置`callingPendingFunctors_`为`true`；3. 遍历执行局部变量中的所有回调；4. 置`callingPendingFunctors_`为`false`；5. 避免持有锁执行回调，减少锁竞争 |

> **所有 IO 操作、Channel 操作、定时器操作，都必须在 EventLoop 所属线程执行，不能跨线程直接调用。**

#### 4. 线程唤醒相关

|     函数名     |                  作用                   |                           调用时机                           |                   内部逻辑 / 触发后续操作                    |
| :------------: | :-------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
|   `wakeup()`   | 唤醒阻塞在`poll()`的 EventLoop 所属线程 | 1. `queueInLoop()`跨线程提交回调时；2. `quit()`跨线程调用时；3. mainLoop 给 subLoop 分配 Channel 时 | 向`wakeupFd_`写入 8 字节数据，触发`wakeupChannel_`的读事件，使`poll()`返回，EventLoop 线程从阻塞中唤醒 |
| `handleRead()` |    `wakeupChannel_`的读事件回调函数     | `wakeupFd_`有读事件（即`wakeup()`调用后），被 EventLoop 触发 | 读取`wakeupFd_`中的 8 字节数据，清空事件，使`wakeupFd_`可再次触发；核心目的是唤醒阻塞的`poll()`，无其他业务逻辑 |

> - `wakeup()`：负责「发信号」，把线程从阻塞中叫醒，让 epoll_wait 立刻返回，由其他线程 / 同线程任务执行
> - `handleRead()`：负责「收信号」，把信号清掉，恢复正常循环，复位 eventfd，避免循环空转，EventLoop 自己线程执行

#### 5. Channel 管理（转发给 Poller）

|              函数名               |                             作用                             |                           调用时机                           |                           内部逻辑                           |
| :-------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| `updateChannel(Channel *channel)` | 更新 Channel 在 Poller 中的事件监听状态（如新增 / 修改监听事件） | 1. Channel 的事件状态变化时（如`enableReading()`）；2. 新增 Channel 到 EventLoop 时 | 校验是否在 EventLoop 所属线程，然后调用`poller_->updateChannel(channel)` |
| `removeChannel(Channel *channel)` |         将 Channel 从 Poller 中移除，停止监听其事件          |           Channel 生命周期结束、不再需要监听时调用           | 校验是否在 EventLoop 所属线程，然后调用`poller_->removeChannel(channel)` |
|  `hasChannel(Channel *channel)`   |       判断指定 Channel 是否在当前 Poller 的监听列表中        |       上层需要校验 Channel 是否被 EventLoop 管理时调用       |         调用`poller_->hasChannel(channel)`并返回结果         |

> channel update remove => EventLoop updateChannel removeChannel => Poller updateChannel removeChannel
>
> ```cpp
> // 完整的调用链路：Channel → EventLoop → Poller
> 用户(TcpConnection/Acceptor)
>     ↓
> Channel::enableReading / enableWriting / disableAll	// 设置fd监听的event状态
>     ↓
> Channel::update()	/ Channel::remove()	// 转发
>     ↓
> EventLoop::updateChannel(channel)	/ EventLoop::removeChannel()	// 转发
>     ↓
> Poller::updateChannel(channel)	/ Poller::removeChannel()	// 向epoll实例中修改fd的操作
>     ↓
> epoll_ctl / poll 真正系统调用	/ epoll_ctl DEL
> ```

#### 6. 辅助查询 / 校验

|          函数名          |                   作用                    |                           调用时机                           |                       内部逻辑                        |
| :----------------------: | :---------------------------------------: | :----------------------------------------------------------: | :---------------------------------------------------: |
| `pollReturnTime() const` |  返回 Poller 最近一次检测到事件的时间戳   |        上层需要获取事件发生时间时调用（如日志、监控）        |               直接返回`pollRetureTime_`               |
| `isInLoopThread() const` | 判断当前调用线程是否是 EventLoop 所属线程 | 1. `loop()`启动时校验；2. `updateChannel()`/`removeChannel()`等方法校验；3. 上层需要确认线程归属时调用 | 对比`threadId_`和`CurrentThread::tid()`，返回布尔结果 |

#### 7. 工具函数

```cpp
int createEventfd()	// 函数签名，作用：创建wakeupfd 用来notify唤醒subReactor处理新来的channel
```

`createEventfd()` 不定义在头文件、也不作为 `EventLoop` 类成员函数的核心原因：

1. 它是**工具函数**（仅为 `EventLoop` 构造函数服务的辅助逻辑），无需暴露给外部；
2. 避免头文件冗余依赖，减少编译耦合；
3. 类内成员函数会引入不必要的 `this` 指针开销，且该函数无访问类成员的需求；
4. 保证线程局部存储（`__thread`）和单例逻辑的独立性。
5. **线程安全辅助**：`createEventfd()` 创建的 `eventfd` 带 `EFD_CLOEXEC`（fork 时自动关闭父进程描述符）和 `EFD_NONBLOCK`（非阻塞），为 `EventLoop` 的跨线程唤醒（`wakeup()`）提供了安全的基础，且该逻辑封装在工具函数中，避免 `EventLoop` 构造函数臃肿。