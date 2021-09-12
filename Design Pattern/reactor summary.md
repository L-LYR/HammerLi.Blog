---
Author: HammerLi
Date: 2021.3.16
Tag: [Reactor]
---

[Toc]

# Reactor模型学习总结

## 原始思路

```pseudocode
while (true) {
	socket = accept(/*...*/);
	handle(socket);
}
```

上述是单进程的阻塞式的网络编程思路，不支持并发，串行处理，当前请求未处理完成，后来的请求只能被阻塞，服务器吞吐量很低。

## 多线程模型

为了提高并发，自然而然选用了多线程的设计思路，即经典的 connection per thread 。

```pseudocode
while (true) {
	socket = accept(/*...*/);
	thread = new Thread(handle, socket);
	thread.run();
}q
```

多线程的模式极大提高了服务器的吞吐量，这将使得同时到来的每个连接请求能够由一个线程来处理。每一个 socket 都是阻塞的，所以一个线程中只能服务一个连接。

该模型下的缺点在于资源要求较高，创建线程是需要系统资源的，如果连接数比较高，系统无法承受，同时，反复地创建销毁线程也是需要较高的代价与开销。

## Reactor模型

Reactor模型是一种典型的事件驱动编程模型。

### 组件

#### Handles

操作系统管理的资源，其实就是所谓的文件描述符 (File Descriptor) 或者句柄 (Handle)。比如外部的客户端连接，内部的定时器事件。

#### Synchronous Event Demultiplexer

同步事件多路分离器，它会在资源 (handles) 上等待事件的发生，调用方会在调用时阻塞，直到同步事件来临，常见的就是我们所调用的多路复用机制函数 select poll epoll 等。

#### Initiation Dispatcher

初始分发器，实质上就是 Reactor ，用来进行注册、删除和分发事件处理。它是整个模型的核心。

#### Event Handler

事件处理接口，需要针对应用的具体服务来实现。

#### Concrete Event Handler

事件处理接口的具体实现，即事件处理器，它实现了事件处理接口所定义的各种回调方法，从而完善特定业务逻辑。每个具体的事件处理器总和一个描述符相关。它使用描述符来识别事件、识别应用程序提供的服务。

### 流程

1. Initiation Dispatcher 初始化，注册若干预先定义的 Concrete Event Handler ，同时标识需要监听的事件。当标识的事件发生时，Initiation Dispatcher 将会发出通知，因为事件是通过 Handle 来标识的，而 Concrete Event Handler 与 Handle 关联，所以它可以拿到通知。
2. Initiation Dispatcher 需要 Concrete Event Handler 传递所持有的 Handler ，用以标识 Concrete Event Handler。
3. Initiation Dispatcher 启动事件循环，所有 Handle 合并起来，使用 Synchronous Event Demultiplexer 同步阻塞，等待事件发生。
4. 当某个事件源对应的 Handle 变为 Ready 状态时，Synchronous Event Demultiplexer 将会通知 Initiation Dispatcher。
5. Initiation Dispatcher 触发 Ready 状态的 Handle 对应的注册的 Concrete Event Handler 的回调方法，从而响应事件
6. Initiation Dispatcher 回调事件处理器的 handle_event(type) 回调方法来执行特定业务。

