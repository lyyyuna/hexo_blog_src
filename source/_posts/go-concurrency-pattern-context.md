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