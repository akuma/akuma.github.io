---
title: 几个常用的 Linux 系统分析工具
date: 2017-01-13 10:04:07
tags: linux tool top
---

## top

`top` 命令用于快速了解当前系统的负载情况。

比较常用的交互式命令：

- `1`：查看每个 CPU 的时间分配情况
- `H`：查看每个线程的资源占用情况
- `P`：执行任务按 CPU 占用降序排序
- `M`：执行任务按内存占用降序排序

<!--more-->

| 行号 | 字段 | 意义 |
| - | - | - |
| 1 | top | 程序名 |
| | 14:59:20 | 当前时间 |
| | up 6:30 | 指计算机从上次启动到现在所运行的时间 |
| | 2 users | 指有多少用户登录系统 |
| | load average | 指等待运行的进程数，三个数字分别是最近的 1 分钟、5 分钟、15 分钟的平均值 |
| 2 | Tasks | 描述了进程数目和各种进程状态 |
| 3 | Cpu(s) | 描述了 CPU 时间分配统计 |
| | us | user 的缩写，CPU 消耗在 User space 的时间百分比  |
| | sy | system 的缩写，CPU 消耗在 Kernel space 的时间百分比 |
| | ni | niceness 的缩写，CPU 消耗在 nice 进程（低优先级）的时间百分比 |
| | id | idle 的缩写，CPU 消耗在闲置进程的时间百分比，这个值越低，表示 CPU 越忙 |
| | wa | wait 的缩写，CPU 等待外部 I/O 的时间百分比，这段时间 CPU 不能干其他事，但是也没有执行运算，这个值太高就说明外部设备有问题
| | hi | hardware interrupt 的缩写，CPU 响应硬件中断请求的时间百分比 |
| | si | software interrupt 的缩写，CPU 响应软件中断请求的时间百分比 |
| | st | stole time 的缩写，该项指标只对虚拟机有效，表示分配给当前虚拟机的 CPU 时间之中，被同一台物理机上的其他虚拟机偷走的时间百分比 |
| 4 | Mem | 描述物理内存的使用情况 |
| 5 | Swap | 描述交换分区（虚拟内存）的使用情况 |

## sar

TODO

## 参考资料

- [User space 与 Kernel space](http://www.ruanyifeng.com/blog/2016/12/user_space_vs_kernel_space.html)
