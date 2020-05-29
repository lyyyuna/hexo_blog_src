title: Linux 性能分析 - 如何理解 CPU 平均负载
date: 2020-05-29 16:36:38
categories:
- 测试
tags:
- Linux 性能分析
series: Linux 性能分析
---

## 平均负载的定义

`uptime`是最简单的性能分析命令之一，它的输出非常简单，比如：

```
$ uptime
 16:08:29 up 42 days,  5:11,  2 users,  load average: 0.33, 0.22, 0.22
```

前面几列易于理解，分别是当前时间，系统已运行时间，登录的用户数。可是最后的平均负载是什么？查看`uptime`帮助文档对其的定义：

    System load averages is the average number of processes that are either in a runnable or uninterruptable  state.   A  process  in  a runnable  state  is  either  using  the  CPU  or waiting to use the CPU.  A process in uninterruptable state is waiting for some I/O access, eg waiting for disk.  The averages are taken over the three time intervals.  Load averages are not normalized for the number of CPUs in a system, so a load average of 1 means a single CPU system is loaded all the time while on a 4 CPU system it means it was idle 75% of the time.

大概翻译过来就是：

    系统的平均负载是指处于可运行和不可中断状态的进程平均数量。所谓可运行状态是指正在使用或等待使用 CPU，而不可中断状态是指进程正执行某种 I/O 操作，比如读写磁盘。平均负载会计算 1 分钟、5 分钟和 15 分钟内的该指标。不过需要注意的是，该数据没有针对 CPU 个数作归一化处理，所以平均负载 1 在单核系统上意味满负载运行，而在 4 核系统上意味着只使用了 25% 的负载。

### 进程的状态