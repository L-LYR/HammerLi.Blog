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

- 尽量抛出错误，而非panic

- http框架需要在中间件中recover一些原生panic，如解引用空指针

- 避免野goroutine

  - 异步请求采用协程池，有效节约资源，同时一般协程池都会捕获每一个工作协程的panic，保证异步任务的正常运作

  - ```golang
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

