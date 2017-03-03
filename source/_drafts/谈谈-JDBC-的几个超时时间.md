---
title: 谈谈 JDBC 的几个超时时间
date: 2017-02-18 12:12:58
tags: jdbc timeout
---

常见的 JDBC 相关的 `timeout` 类型从高到低级别有：

`Transaction Timeout > Statement Timeout > JDBC Driver Socket Timeout`

高级别的 timeout 依赖于低级别的 timeout，只有当低级别的 timeout 无误时，高级别的 timeout 才能确保正常。例如，当 socket timeout 出现问题时，高级别的 statement timeout 和 transaction timeout 都将失效。

statement timeout 无法处理网络连接失败时的超时，它能做的仅仅是限制statement的操作时间。网络连接失败时的 timeout 必须交由 JDBC 来处理。
JDBC 的 socket timeout 会受到操作系统 socket timeout 设置的影响。

Transaction Timeout 指一组 SQL 操作执行时应在设定的时间内完成（提交或回滚），否则将引发超时。它的值应大于 N（语句数）* Statement Timeout。

Statement Timeout 指完成单条 SQL 语句执行的最大允许时间。它的值应小于 JDBC Driver Socket Timeout的值，因为后者会被先检查。

最底层的 JDBC Driver Socket Timeout 指的是通过 socket 进行连接时的超时或者进行读写操作时的阻塞超时，如果不设置将使用 OS 层的 Socket Timeout。

<!--more-->

## Query Timeout

!(http://www.cubrid.org/files/attach/images/220547/584/303/query-timeout-execution-process-for-mysql-jdbc-statement.png)

## Socket Timeout

### JDBC 的 socket timeout

- 场景：在数据库主机突然宕机或是发生网络故障。
- Socket timeout 会将超时的连接直接关闭，并抛出异常。
- Socket timeout 的值必须要高于 statement timeout。
  - 否则 socket timeout 将会先生效， statement timeout 没法生效，也就变得没意义了

### 实现方式

通过是否有数据返回判定超时。

- select * from test for update;  -- 行锁等待超时
- select count(1) from test;      -- 查询超时
- select * from test;             -- 不发生超时
