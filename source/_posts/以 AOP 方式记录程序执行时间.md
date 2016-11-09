---
title: 以 AOP 方式记录程序执行时间
date: 2009-12-11 11:22:04
tags: aop performance
---

在日志里记录程序的执行时间，应该是便于问题跟踪和性能优化的一个好办法。
通过 AOP 的方式，可以优雅的（尽可能对业务逻辑代码无侵入）实现这一点。
而结合一些常用的工具类，可以比较友好的显示任务的执行时间。

<!--more-->

下面的代码演示了如何使用 Spring 框架中的 StopWatch 类来记录任务执行时间。

```java
import org.springframework.util.StopWatch;

// 创建一个 stopWatch，id 命名为 "demo test"
StopWatch stopWatch = new StopWatch("demo test");

// 开始记录任务 1 的执行时间
stopWatch.start("task1");

// 任务 1 的业务逻辑
...

// 停止对任务 1 的计时
stopWatch.stop();

// 开始记录任务 2 的执行时间
stopWatch.start("task2");

// 任务 2 的业务逻辑
...

// 停止对任务 2 的计时
stopWatch.stop();

...

// 以美观的形式记录任务的执行情况
if (logger.isDebugEnabled()) {
    logger.debug(stopWatch.prettyPrint());
}
```

显示结果类似如下：

```
StopWatch 'demo test': running time (millis) = 812
-----------------------------------------
ms     %     Task name
-----------------------------------------
00765  094%  task1
00047  006%  task2
```
