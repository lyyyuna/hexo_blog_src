title: Golang 的版本管理原则
date: 2020-02-22 16:20:02
categories:
- 语言
tags: 
- Go
---

## 前言

本文翻译+理解自 [The Principles of Versioning in Go](https://research.swtch.com/vgo-principles)

## 为什么需要版本？

让我们先看下传统基于`GOPATH`的`go get`是如何导致版本依赖失败的。

假设有一个全新安装的 Go 环境，我们需要写一个程序导入`D`，因此运行`go get D`。记住现在是基于`GOPATH`的`go get`，不是 `go mod`。

```
$ go get D
```

![](/img/posts/version-in-go/vgo-why-1@1.5x.png)

该命令会寻找并下载最新版本的`D 1.0`，并假设现在能成功编译。

几个月后我们需要一个新的库`C`，我们接着运行`go get C`，该库的版本为 1.8。

```
$ go get C
```

![](/img/posts/version-in-go/vgo-why-2@1.5x.png)

`C`导入`D`，