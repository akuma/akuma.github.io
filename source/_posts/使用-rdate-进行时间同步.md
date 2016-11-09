---
title: 使用 rdate 进行时间同步
date: 2010-06-05 21:35:53
tags: linux rdate ntpdate time
---

## 使用 ntpdate 的问题

最近在使用 `ntpdate` 的时候经常遇到这样的错误提示：

```
no server suitable for synchronization found
```

<!--more-->

`ntpdate` 采用了 UDP 协议 的 123 端口。网上查找资料说很可能是防火墙封锁了 UDP 的 123 端口。如果关闭防火墙后问题依旧，很可能是上层路由的设置有问题。

当遇到这种情况时，我们还有一个方案是通过 TCP 方式来更新时间，这个时候就轮到 `rdate` 登场了。

## 使用 rdate

### 介绍

名称：rdate（receive date）
功能说明：显示其他主机的日期与时间。
语法：rdate [-ps] [主机名称或IP地址...]
补充说明：执行 rdate 命令，向其他主机询问系统时间并显示出来。

参数：

```
-p 　显示远端主机的日期与时间。
-s 　把从远端主机收到的日期和时间，回存到本地主机的系统时间。
```

### 使用举例

查看时间服务器的时间:

```
$ sudo rdate rdate.darkorb.net
```

设置时间和时间服务器同步:

```
$ sudo rdate -s rdate.darkorb.net
```

### 时间服务器列表

```
time-b.nist.gov
rdate.darkorb.net
time-b.timefreq.bldrdoc.gov
```
