---
Author: HammerLi
Date: 2021-09-03
---

[TOC]

# 错误处理

# Panic

## 程序启动时期

- 强依赖服务故障，如拉取配置失败超出重试次数上限
- 配置初始化失败，如没有找到配置文件
- 配置参数明显不符合要求，如参数非负

## 程序运行时期

- 尽量抛出 Error，而非 Panic

- http 框架需要在中间件中 Recover 一些原生 Panic ，如解引用空指针

- 避免野goroutine

 - 异步请求采用协程池，有效节约资源，同时一般协程池都会捕获每一个工作协程的 Panic ，保证异步任务的正常运作

 ```golang
  func GO(f func()) {
    go func() {
      defer func() {
        if r := recover; r != nil {
          // handle
        }
      }()

      f()
    }()
  }
  ```

# Error

## 错误相关实践

- error 应该是函数的最后一个返回值；error 不为 nil 时，其他返回值应当是不可用值，调用方也不应该去期待这些返回值，在处理返回值时必须先处理错误。

- github.com/pkg/error 扩展包很好用。

- 调用外部库或者非基础库代码出现的错误请使用 errors.Wrap 添加堆栈信息，或者使用 errors.WithMessage 添加必要上下文信息。（fmt.Errorf 使用%w时没有堆栈信息）

- 基础库代码出现错误请使用 errors.New, errors.Errorf 或预定义全局错误变量（哨兵错误，Sentinel Error）。

- 判断错误请使用 errors.Is，判断错误类型请使用 errors.As。

- 不要每个地方都打日志，尽可能在最高层使用 `%+v` 打印。不需要返回的错误就地打日志。

- 报错信息是否够用的衡量标准是当需要排查错误时，报错信息是否能够帮助你快速行为问题。

- 注意使用 defer 来辅助错误处理，例如文件关闭。

## 错误处理
