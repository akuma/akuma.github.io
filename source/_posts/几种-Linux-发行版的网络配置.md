---
title: 几种 Linux 发行版的网络配置
date: 2013-07-07 15:30:50
tags: linux network
---

Linux 有很多的发行版本，而这些发行版本对于同一个功能的配置，往往会有一些不同。就拿网络配置来说，不同的版本就很不相同。本文对 Redhat、Ubuntu、Arch 等版本的 Hostname、IP、DNS、IPv6 配置 分别做一下说明。

<!--more-->

为了方便描述，假设目前需要修改的各个值如下表：

| Hostname    | IP           | Gateway     | DNS     |
|:-----------:|:------------:|:-----------:|:-------:|
| demo-server | 192.168.0.10 | 192.168.0.1 | 8.8.8.8 |

## 配置 hostname

### 通用配置

修改 `/etc/hosts`，添加如下内容：

```
127.0.0.1      localhost.localdomain   localhost demo-server
192.168.0.10   localhost               demo-server
```

### Redhat 额外配置

修改 `/etc/sysconfig/network`，将 `HOSTNAME` 设置为：

```
HOSTNAME=demo-server
```

### Arch 额外配置

修改 `/etc/rc.conf`，将 `HOSTNAME` 设置为：

```
HOSTNAME="demo-server"
```

## 配置 IP

### Redhat 配置

修改 `/etc/sysconfig/network-script`，添加如下内容（二选一）：

- 静态 IP

```
DEVICE=eth0
BOOTPROTO=static
BROADCAST=192.168.0.255
IPADDR=192.168.0.10
NETMASK=255.255.255.0
GATEWAY=192.168.0.254
ONBOOT=on
```

- 动态 IP

```
DEVICE=eth0
BOOTPROTO=dbcp
ONBOOT=on
```

### Ubunt 配置

修改 `/etc/network/interface`，添加如下内容（二选一）：

- 静态 IP

```
auto eth0
iface eth0 inet static
address 192.168.0.10
netmask 255.255.255.0
gateway 192.168.0.1
```

- 动态 IP

```
auto eth0
iface eth0 inet dbcp
```

### Arch 配置

修改 `/etc/rc.conf`，添加如下内容（二选一）：

- 静态 IP

```
interface=eth0
address=192.168.0.10
netmask=255.255.255.0
gateway=192.168.0.1
```

- 动态 IP

```
interface=eth0
address=
netmask=
gateway=
```

## 配置 DNS

DNS 配置都通用。

修改 `/etc/resolve.conf`，添加如下内容：

```
nameserver 8.8.8.8
```

## 关闭 IPv6

一般现在很多 Linux 默认将 IPv6 开启。但实际目前 IPv4 仍然是主流，而且对于我们来说，在开发测试中并不需要 IPv6，因此可以关闭。

### Redhat 配置

修改 `/etc/sysconfig/network`，将 `NETWORKING_IPV6` 设置为：

```
NETWORKING_IPV6=no
```

### Ubuntu 配置

修改 `/etc/sysctl.conf`，添加如下内容：

```
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
```

最后执行命令：

```
$ sysctl -p
```

### Arch 配置

#### 方法一：关闭模块（模块不加载）

修改 `/etc/modprobe.d/modprobe.conf`，添加如下内容：

```
# disable autoload of ipv6
alias net-pf-10 off
```

#### 方法二：关闭功能（模块仍然加载）

修改 `/etc/modprobe.d/modprobe.conf`，添加如下内容：

```
options ipv6 disable=1
```
