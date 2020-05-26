title: Golang 并发模式 - Context
date: 2020-05-25 22:19:02
categories:
- 语言
tags: 
- Go
---

## 前言

本文翻译+删选+理解自 [Go Concurrency Patterns: Context](https://blog.golang.org/context)

在使用 Go 编写的服务器程序中，每个请求都由一个 goroutine 来处理，通常这些请求又会启动额外的 goroutines 来访问后台数据库或者调用 RPC 服务。这些与同一个请求相关的 goroutines，常常需要访问同一个特定的资源，比如用户标识，认证 token 等等。当请求取消或者超时时，所有相关的 goroutines 都应该快速退出，这样系统才能回收不用的资源。

为此，Google 公司开发了[context](https://golang.org/pkg/context)包。该库可以跨越 API 边界，给所有 goroutines 传递请求相关的值、取消信号和超时时间点。这篇文章会介绍如何使用[context](https://golang.org/pkg/context)库，并给出一个完整的例子。

## Context

`context`包的核心是`Context`结构体：

```go
// A Context carries a deadline, cancelation signal, and request-scoped values
// across API boundaries. Its methods are safe for simultaneous use by multiple
// goroutines.
type Context interface {
    // Done returns a channel that is closed when this Context is canceled
    // or times out.
    Done() <-chan struct{}

    // Err indicates why this context was canceled, after the Done channel
    // is closed.
    Err() error

    // Deadline returns the time when this Context will be canceled, if any.
    Deadline() (deadline time.Time, ok bool)

    // Value returns the value associated with key or nil if none.
    Value(key interface{}) interface{}
}
```

`Done`方法返回一个 channel，它会给`Context`上所有的函数发送取消信号，当 channel 关闭时，这些函数应该终止剩余流程立即返回。`Err`方法返回的错误指出了`Context`为什么取消。[Pipelines and Cancelation](https://blog.golang.org/pipelines)中讨论了`Done`的具体用法。

由于接收和发送信号的通常不是同一个函数，`Context`并没有提供`Cancel`方法，基于相同的理由，`Done`channel 只负责接收。尤其当父操作开启 goroutines 执行子操作时，子操作肯定不能取消父操作。作为替代，`WithCancel`函数可以用来取消`Context`。

`Context`对 goroutines 来说是并发安全的，你可以将单个`Context`传递给任意数量的 goroutines，然后撤销该`Context`给它们同时发送信号。

`Deadline`方法允许函数决定是否继续运行，如果时间快到了，运行也就没必要了。代码可依此为 I/O 操作设置超时时间。

`Value`方法则为`Context`携带了请求所有的数据，这些数据也必须是并发安全的。

## Context 继承