---
title: Ubuntu 下搭建 DNS 服务器笔记
date: 2010-05-31 22:02:10
tags: linux ubuntu dns
---

## 安装 bind9

```sh
sudo apt-get install bind9
```

## 修改 bind 的配置文件

```sh
sudo vi /etc/bind/named.conf.local
```

<!--more-->

在此文件中可以添加我们想要的 zone。 zone 就是被 DNS 服务器引用的域名。将以下内容添加到此文件中：

```
# 请将 winupon.tst 替换为需要解析的主域名
zone "winupon.tst" {
    type master;
    file "/etc/bind/zones/winupon.tst.db";
};

# 反向 DNS 定义，用于根据 DNS 服务器的 IP 地址找到对应的机器名
# 此例子中，DNS 服务器的 IP 地址是 192.168.0.1
zone "0.168.192.in-addr.arpa" {
     type master;
     file "/etc/bind/zones/rev.0.168.192.in-addr.arpa";
};
```

接下来修改 named.conf.options 文件：

```sh
sudo vi /etc/bind/named.conf.options
```

修改 forwarders 部分，对于 DNS 服务器不能解析的域名，会交由该服务器处理：

```
forwarders {
      # 使用 Google 的 DNS 服务器
      8.8.8.8;
};
```

添加 zone 定义文件，请将 winupon.tst 替换为实际的主域名：

```sh
sudo mkdir /etc/bind/zones
sudo vi /etc/bind/zones/winupon.tst.db
```

加入以下内容：

```
// 请对以下内容进行必要的替换
// ns1 = DNS 服务器名
// mta = 邮件服务器名
// winupon.tst = 需要解析的域名

winupon.tst.      IN      SOA     ns1.winupon.tst. admin.winupon.tst. (
        2006081401
        28800
        3600
        604800
        38400
)

winupon.tst.      IN      NS              ns1.winupon.tst.
winupon.tst.      IN      MX     10       mta.winupon.tst.

# 对域名和 IP 进行配置，CNAME 可以设置域名别名，根据情况选用
www              IN      A       192.168.0.2
mta              IN      A       192.168.0.3
ns1              IN      A       192.168.0.1
abc              IN      A       192.168.0.4
server           IN      CNAME   abc
```

常用的资源记录类型说明：

A 地址：此记录列出特定主机名的 IP 地址。这是名称解析的重要记录。
CNAME 标准名称：此记录指定标准主机名的别名。
MX 邮件交换器：此记录列出了负责接收发到域中的电子邮件的主机。
NS 名称服务器：此记录指定负责给定区域的名称服务器。

创建反向 DNS zone 定义文件，请将 winupon.tst 替换为实际的主域名：

```sh
sudo vi /etc/bind/zones/rev.0.168.192.in-addr.arpa
```

添加内容如下：

```
// 将 winupon.tst 替换为实际的域名，将 ns1 替换为 DNS 服务器的名称
// PTR 记录前面的数字 1 是 DNS 服务器的地址，这个例子中，完整的 IP 地址是：192.168.0.1
@ IN SOA ns1.winupon.tst. admin.winupon.tst. (
        2006081401;
        28800;
        604800;
        604800;
        86400
)

        IN    NS     ns1.winupon.tst.
1       IN    PTR    winupon.tst
```

## 重启 bind 服务并进行测试

重启 bind 服务：

```sh
sudo /etc/init.d/bind9 restart
```

将本机的主 DNS 设置为 192.168.0.1。如果是 windows 系统，采用以下命令刷新 DNS 缓存：

```sh
ipconfig /flushdns
```

最后，使用 ping 测试域名解析是否正常：

```sh
ping www.winupon.tst
```

如无意外，大功告成！
